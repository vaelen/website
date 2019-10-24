---
date: "2013-06-04T04:14:28"
lastmod: "2013-06-04T04:14:28"
title: "Skype Problems on Ubuntu 13.04 64bit"
categories:
- Linux

---
I've been having problems getting Skype to work properly on Ubuntu 13.04. When Skype was running properly the other audio applications on the box wouldn't work. If I started Skype while other audio applications were running then I'd have to audio in Skype. Ubuntu uses the PulseAudio audio server for its audio services and so does Skype, so things should have been working. However, when I went into the Skype options panel PulseAudio wasn't listed as a possible audio device. It turns out the problem has to do with me using a 64bit version of Ubuntu. The Skype binary is 32bit and it is linked against a 32bit version of GStreamer, but the 32bit version of the PulseAudio client isn't installed by default. This is easily fixed by running:

``` bash
sudo apt-get install libpulse0:i386
```

Hopefully this will save somebody else some time trying to debug this issue!
