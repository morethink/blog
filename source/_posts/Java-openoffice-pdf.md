---
title: Java使用Openoffice将word、ppt转换为PDF
date: 2017-12-25
tags: PDF
categories: Java
---


最近项目中要实现WORD的文件预览功能，我们可以通过将WORD转换成PDF或者HTML，然后通过浏览器预览。

# OpenOffice

OpenOffice.org 是一套跨平台的办公室软件套件，能在 Windows、Linux、MacOS X (X11)、和 Solaris 等操作系统上执行。它与各个主要的办公室软件套件兼容。OpenOffice.org 是自由软件，任何人都可以免费下载、使用、及推广它。

<!-- more -->


**下载地址**

http://www.openoffice.org/

# JodConverter

jodconverter-2.2.2.zip 下载地址：
http://sourceforge.net/projects/jodconverter/files/JODConverter/

# Word转换

**启动OpenOffice的服务**

进入openoffice安装目录，通过cmd启动一个soffice服务，启动的命令是`soffice -headless -accept="socket,host=127.0.0.1,port=8100;urp;"`。

# 运行代码


```java
public class PDFDemo {

    public static boolean officeToPDF(String sourceFile, String destFile) {
        try {

            File inputFile = new File(sourceFile);
            if (!inputFile.exists()) {
                // 找不到源文件, 则返回false
                return false;
            }
            // 如果目标路径不存在, 则新建该路径
            File outputFile = new File(destFile);
            if (!outputFile.getParentFile().exists()) {
                outputFile.getParentFile().mkdirs();
            }
            //如果目标文件存在，则删除
            if (outputFile.exists()) {
                outputFile.delete();
            }
            DateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm");
            OpenOfficeConnection connection = new SocketOpenOfficeConnection("127.0.0.1", 8100);
            connection.connect();
            //用于测试openOffice连接时间
            System.out.println("连接时间:" + df.format(new Date()));
            DocumentConverter converter = new StreamOpenOfficeDocumentConverter(
                    connection);
            converter.convert(inputFile, outputFile);
            //测试word转PDF的转换时间
            System.out.println("转换时间:" + df.format(new Date()));
            connection.disconnect();
            return true;
        } catch (ConnectException e) {
            e.printStackTrace();
            System.err.println("openOffice连接失败！请检查IP,端口");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return false;
    }

    public static void main(String[] args) {
        officeToPDF("E:\\test.docx", "E:\\test.pdf");
    }
}
```
# Word、ppt转Html

只需要将后缀名从`.pdf`改为`.html`即可。

```java
public static void main(String[] args) {
    officeToPDF("E:\\test.docx", "E:\\test.html");
}
```

# Maven配置

**Maven依赖**

```xml
<dependency>
	<groupId>com.artofsolving</groupId>
	<artifactId>jodconverter</artifactId>
	<version>2.2.1</version>
</dependency>
<dependency>
	<groupId>org.openoffice</groupId>
	<artifactId>jurt</artifactId>
	<version>3.0.1</version>
</dependency>
<dependency>
	<groupId>org.openoffice</groupId>
	<artifactId>ridl</artifactId>
	<version>3.0.1</version>
</dependency>
<dependency>
	<groupId>org.openoffice</groupId>
	<artifactId>juh</artifactId>
	<version>3.0.1</version>
</dependency>
<dependency>
	<groupId>org.openoffice</groupId>
	<artifactId>unoil</artifactId>
	<version>3.0.1</version>
</dependency>
<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-jdk14</artifactId>
	<version>1.4.3</version>
</dependency>
```

Maven只有 2.2.1版本，2.2.1版本有一个问题，那就是不兼容docx和pptx，如果你们不使用jodconverter-2.2.2 中lib，而想要使用2.2.1版本，需要修改一下 `BasicDocumentFormatRegistry` 类中的 `getFormatByFileExtension`方法：
1. 新建包 `com.artofsolving.jodconverter`
2. 新建类`BasicDocumentFormatRegistry`，复制下面代码
```java
package com.artofsolving.jodconverter;

/**
 * @author 李文浩
 * @date 2017/12/25
 */

import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class BasicDocumentFormatRegistry implements DocumentFormatRegistry {
    private List documentFormats = new ArrayList();

    public BasicDocumentFormatRegistry() {
    }

    public void addDocumentFormat(DocumentFormat documentFormat) {
        this.documentFormats.add(documentFormat);
    }

    protected List getDocumentFormats() {
        return this.documentFormats;
    }

    public DocumentFormat getFormatByFileExtension(String extension) {
        if (extension == null) {
            return null;
        } else {
            if (extension.indexOf("doc") >= 0) {
                extension = "doc";
            }
            if (extension.indexOf("ppt") >= 0) {
                extension = "ppt";
            }
            if (extension.indexOf("xls") >= 0) {
                extension = "xls";
            }
            String lowerExtension = extension.toLowerCase();
            Iterator it = this.documentFormats.iterator();

            DocumentFormat format;
            do {
                if (!it.hasNext()) {
                    return null;
                }

                format = (DocumentFormat)it.next();
            } while(!format.getFileExtension().equals(lowerExtension));

            return format;
        }
    }

    public DocumentFormat getFormatByMimeType(String mimeType) {
        Iterator it = this.documentFormats.iterator();

        DocumentFormat format;
        do {
            if (!it.hasNext()) {
                return null;
            }

            format = (DocumentFormat)it.next();
        } while(!format.getMimeType().equals(mimeType));

        return format;
    }
}
```

下面是增加的部分，仅仅增加了将docx按照doc的处理方式处理。而2.2.2版本已经默认增加了。

```java
if (extension.indexOf("doc") >= 0) {
    extension = "doc";
}
if (extension.indexOf("ppt") >= 0) {
    extension = "ppt";
}
if (extension.indexOf("xls") >= 0) {
    extension = "xls";
}
```

**参考文档**：
1. [Java实现在线预览–openOffice实现](http://blog.csdn.net/yjclsx/article/details/51445546)
2. [Java项目中使用OpenOffice转PDF](http://blog.csdn.net/qq_33571718/article/details/51154472)
3. [java使用openoffice将office系列文档转换为PDF](http://blog.csdn.net/make_a_difference/article/details/53771136###;)
4. [java 如何将 word,excel,ppt如何转pdf--jacob](http://www.cnblogs.com/xxyfhjl/p/6773786.html)
