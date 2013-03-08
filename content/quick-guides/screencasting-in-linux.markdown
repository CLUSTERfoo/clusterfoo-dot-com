---
title: Screencasting In Linux
kind: article
created_at: 2013-02-28
---
<!-- i_i -->

A quick screencast and some notes on screencasting in Linux. 

<iframe class="centered" width="560" height="315" 
    src="http://www.youtube.com/embed/c3EFVviXS40" 
    frameborder="0" allowfullscreen></iframe>

### Tools used

* arandr (easily manage your screens and their resolution)

* recordmydesktop (record your desktop activity)

* key-mon (display keyboard activity)

* pavucontrol (GUI for managing PulseAudio volume and stuff)

* vlc (to extract OGG audio file)

* audacity (edit your audioi for proper volume levels)

* openshot (edit and export final video)

### Preparation

Using arandr, change screen resolution to 1280x720 (720p) and switch to
a large terminal font.

Use pavucontrol to make sure audio levels are decent.

For the actual recording, the following command gives me best results:

~~~
recordmydesktop --freq 48000 -o [path]
~~~

`CTRL-C` when finished. 

### Exporting for YouTube

I export the file as a 720p flv, using aac for audio codec.
