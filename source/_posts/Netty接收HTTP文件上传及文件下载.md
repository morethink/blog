---
title: Netty接收HTTP文件上传及文件下载
date: 2018-10-14
tags: Netty
categories: Java
---

# 文件上传

这个处理器的原理是接收HttpObject对象，按照HttpRequest，HttpContent来做处理，文件内容是在HttpContent消息带来的。

然后在HttpContent中一个chunk一个chunk读，chunk大小可以在初始化HttpServerCodec时设置。将每个chunk交个httpDecoder复制一份，当读到LastHttpContent对象时，表明上传结束，可以将httpDecoder中缓存的文件通过HttpDataFactory写到磁盘上，然后在删除缓存的HttpContent对象。

<!-- more -->

```java
@Slf4j
public class HttUploadHandler extends SimpleChannelInboundHandler<HttpObject> {

    public HttUploadHandler() {
        super(false);
    }

    private static final HttpDataFactory factory = new DefaultHttpDataFactory(true);
    private static final String FILE_UPLOAD = "/data/";
    private static final String URI = "/upload";
    private HttpPostRequestDecoder httpDecoder;
    HttpRequest request;

    @Override
    protected void channelRead0(final ChannelHandlerContext ctx, final HttpObject httpObject)
            throws Exception {
        if (httpObject instanceof HttpRequest) {
            request = (HttpRequest) httpObject;
            if (request.uri().startsWith(URI) && request.method().equals(HttpMethod.POST)) {
                httpDecoder = new HttpPostRequestDecoder(factory, request);
                httpDecoder.setDiscardThreshold(0);
            } else {
                //传递给下一个Handler
                ctx.fireChannelRead(httpObject);
            }
        }
        if (httpObject instanceof HttpContent) {
            if (httpDecoder != null) {
                final HttpContent chunk = (HttpContent) httpObject;
                httpDecoder.offer(chunk);
                if (chunk instanceof LastHttpContent) {
                    writeChunk(ctx);
                    //关闭httpDecoder
                    httpDecoder.destroy();
                    httpDecoder = null;
                }
                ReferenceCountUtil.release(httpObject);
            } else {
                ctx.fireChannelRead(httpObject);
            }
        }

    }

    private void writeChunk(ChannelHandlerContext ctx) throws IOException {
        while (httpDecoder.hasNext()) {
            InterfaceHttpData data = httpDecoder.next();
            if (data != null && HttpDataType.FileUpload.equals(data.getHttpDataType())) {
                final FileUpload fileUpload = (FileUpload) data;
                final File file = new File(FILE_UPLOAD + fileUpload.getFilename());
                log.info("upload file: {}", file);
                try (FileChannel inputChannel = new FileInputStream(fileUpload.getFile()).getChannel();
                     FileChannel outputChannel = new FileOutputStream(file).getChannel()) {
                    outputChannel.transferFrom(inputChannel, 0, inputChannel.size());
                    ResponseUtil.response(ctx, request, new GeneralResponse(HttpResponseStatus.OK, "SUCCESS", null));
                }
            }
        }
    }


    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable cause) throws Exception {
        log.warn("{}", cause);
        ctx.channel().close();
    }

    @Override
    public void channelInactive(ChannelHandlerContext ctx) throws Exception {
        if (httpDecoder != null) {
            httpDecoder.cleanFiles();
        }
    }

}
```

# 文件下载

参考官方Demo： https://github.com/netty/netty/blob/4.1/example/src/main/java/io/netty/example/http/file/HttpStaticFileServerHandler.java

做了改动：
- 为了更高效的传输大数据，实例中用到了ChunkedWriteHandler编码器，它提供了以zero-memory-copy方式写文件。
- 通过ChannelProgressiveFutureListener对文件下载过程进行监听。

```java
 // 新增ChunkedHandler，主要作用是支持异步发送大的码流（例如大文件传输），但是不占用过多的内存，防止发生java内存溢出错误
ch.pipeline().addLast(new ChunkedWriteHandler());
// 用于下载文件
ch.pipeline().addLast(new HttpDownloadHandler());
```

```java
@Slf4j
public class HttpDownloadHandler extends SimpleChannelInboundHandler<FullHttpRequest> {

    public HttpDownloadHandler() {
        super(false);
    }

    private String filePath = "/data/body.csv";

    @Override
    protected void channelRead0(ChannelHandlerContext ctx, FullHttpRequest request) {
        String uri = request.uri();
        if (uri.startsWith("/download") && request.method().equals(HttpMethod.GET)) {
            GeneralResponse generalResponse = null;
            File file = new File(filePath);
            try {
                final RandomAccessFile raf = new RandomAccessFile(file, "r");
                long fileLength = raf.length();
                HttpResponse response = new DefaultHttpResponse(HTTP_1_1, HttpResponseStatus.OK);
                response.headers().set(HttpHeaderNames.CONTENT_LENGTH, fileLength);
                response.headers().set(HttpHeaderNames.CONTENT_TYPE, "application/octet-stream");
                response.headers().add(HttpHeaderNames.CONTENT_DISPOSITION, String.format("attachment; filename=\"%s\"", file.getName()));
                ctx.write(response);
                ChannelFuture sendFileFuture = ctx.write(new DefaultFileRegion(raf.getChannel(), 0, fileLength), ctx.newProgressivePromise());
                sendFileFuture.addListener(new ChannelProgressiveFutureListener() {
                    @Override
                    public void operationComplete(ChannelProgressiveFuture future)
                            throws Exception {
                        log.info("file {} transfer complete.", file.getName());
                        raf.close();
                    }

                    @Override
                    public void operationProgressed(ChannelProgressiveFuture future,
                                                    long progress, long total) throws Exception {
                        if (total < 0) {
                            log.warn("file {} transfer progress: {}", file.getName(), progress);
                        } else {
                            log.debug("file {} transfer progress: {}/{}", file.getName(), progress, total);
                        }
                    }
                });
                ctx.writeAndFlush(LastHttpContent.EMPTY_LAST_CONTENT);
            } catch (FileNotFoundException e) {
                log.warn("file {} not found", file.getPath());
                generalResponse = new GeneralResponse(HttpResponseStatus.NOT_FOUND, String.format("file %s not found", file.getPath()), null);
                ResponseUtil.response(ctx, request, generalResponse);
            } catch (IOException e) {
                log.warn("file {} has a IOException: {}", file.getName(), e.getMessage());
                generalResponse = new GeneralResponse(HttpResponseStatus.INTERNAL_SERVER_ERROR, String.format("读取 file %s 发生异常", filePath), null);
                ResponseUtil.response(ctx, request, generalResponse);
            }
        } else {
            ctx.fireChannelRead(request);
        }
    }

    @Override
    public void exceptionCaught(ChannelHandlerContext ctx, Throwable e) {
        log.warn("{}", e);
        ctx.close();

    }
}
```

## 下载文件遇到的坑

由于`RandomAccessFile`是一种文件资源，所以我习惯性的在最后关闭文件资源，采用的是Java7的 `try-with-resources` 语法，于是问题就出现了，由于 `ctx.write(new DefaultFileRegion(raf.getChannel(), 0, fileLength), ctx.newProgressivePromise());` 是异步的，在我关闭`RandomAccessFile`时，文件还未传输完毕，就会导致下载文件停止。


代码放在： https://github.com/morethink/code/tree/master/java/netty-example
