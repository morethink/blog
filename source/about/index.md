---
title: about
date: 2016-08-21 10:12:36
type: "about"
---
---
![alasijia](https://images.morethink.cn/alasijia.jpg)

<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/aplayer/1.10.1/APlayer.min.css">

<div id="aplayer"></div>
<script src="https://cdnjs.cloudflare.com/ajax/libs/aplayer/1.10.1/APlayer.min.js"></script>
<script>
    const ap = new APlayer({
        container: document.getElementById('aplayer'),
        mini: false,
        autoplay: false,
        theme: '#242424',
        loop: 'all',
        order: 'random',
        preload: 'auto',
        volume: 0.7,
        mutex: true,
        listFolded: false,
        listMaxHeight: 90,
        lrcType: 3,
        audio: [
            {
                name: 'Something Just Like This',
                artist: 'The Chainsmokers / Coldplay',
                url: 'http://music.morethink.cn///Something%20Just%20Like%20This.mp3',
                cover: 'http://music.morethink.cn//Something%20Just%20Like%20This.jpeg',
                lrc: 'http://music.morethink.cn//Something%20Just%20Like%20This.lrc',
                theme: '#242424'
            }
        ]
    });
    ap.init();

</script>
