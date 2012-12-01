---
layout: page
---

An Embedded Linux Distribution for Hamradio
===========================================
Hambedded Linux is an embedded linux distribution specifically built for amateur
radio usage in mind. Hambedded makes use of [OpenEmbedded][openembedded] and
[Yocto][yocto] using [bitbake][bitbake] as a build system. Currently, Hambedded
is in research phase and it does not have a release. When images are released,
the page will be updated and you will be notified.  Keep following Hambedded on
[Twitter][hambedded-twitter] and [Github][hambedded-github]!

[openembedded]: http://openembedded.org
[yocto]: https://www.yoctoproject.org/
[bitbake]: http://docs.openembedded.org/bitbake/html/
[hambedded-github]: https://github.com/hambedded-linux
[hambedded-twitter]: https://twitter.com/hambedded

Motivation
==========
Hamradio operators work with a lot of different hardwares (most of them have
limited CPU and memory) to enjoy their hobby and to make things work. However,
there is no unified effort to use Linux for hamradio applications on these kind
of devices. Most hams generally use stripped down version of a full linux
distribution such as Debian, Ubuntu, or Voyage. The problems with stripped down
version of a full distribution are:

* The distribution itself was not created to on embedded devices. It was
  intended for normal users to be used in desktop pc.
* The image you get or create will not work on different architectures such as
  ARM, and MIPS. Setting up a development environment for targeting such
  architectures is hard. 
* It's really hard to fully strip down the distribution under 5MB. What if you
  want to run your linux on a different hardware (probably cheaper) with limited storage?

Hambedded aims to provide easy-to-install, hamradio optimized, working linux
images for different hardwares. It's based on OpenEmbedded and Yocto Project
which is actively developed, and for which a great community exists.

There is plan to provide a tiny image which runs APRS daemon and
repeater controller, a GUI image that bundles all the useful hamradio
applications, and an easy-to-get development environment for curious hams to make
their own image.

To make the advantages of OpenEmbedded clear, let's think about an example.
When a minimal layer which includes aprx and repeater controller has been made,
you will automatically be able to create the same image for different
architectures and hardwares by only changing a few configuration variables. If
you want to run [aprx][aprx] daemon on BeagleBoard, you will get meta-ti layer and
create an image for beagleboard. Want to run it on Intel? Go get the associated
layer, use it and compile the image. To make your life easier, Hambedded can
provide pre built, ready-to-use images for different hardware. You will just
download the image, write on the hardware, and run. Of course, providing images
will be possible if Hambedded will have a powerful build server :)

Progress
========
I have read openembedded documentations, subscribed to development list, and
delved into openembedded code to better understand the the project I will be
using. I have learned a lot about the build system (bitbake) and openembedded
work flow. I even started contributing to documentation :) You can see the draft
on [blog](/blog/2012/11/24/from-bitbake-hello-world-to-an-image/)

Within hambedded linux, I will provide the following:

* BSP Layer for [ALIX3D3][alix3d3] board.
* meta-hamradio layer for openembedded. The layer will include hamradio related
  applications such as [aprx][aprx], [svxlink][svxlink], [fldigi][fldigi],
  [drats][drats], etc.
* Recipes for creating tiny, GUI, and other images. For GUI, E17 is planned to
  be used since it is fast, robust, and memory efficient which makes it perfect
  for embedded systems.

[alix3d3]: http://pcengines.ch/alix3d3.htm
[aprx]: http://wiki.ham.fi/Aprx.en
[svxlink]: http://sourceforge.net/apps/trac/svxlink/
[fldigi]: http://www.w1hkj.com/Fldigi.html
[drats]: http://www.d-rats.com/

Supported Hardwares
===================
Since Hambedded will be using OpenEmbedded, Hambedded will support the hardware
and the boards that openembedded has a support for. It includes support for
popular architectures such as ARM, Intel, MIPS, and PowerPC. Some of the
officially supported hardware worth mentioning are:

TI's

* beagleboard
* pandaboard
* beaglebone
* hawkboard

Intel's

* cedartrail
* chiefriver
* fri2
* crystalforest
* emenlow

Gumstix'

* Overo

... and dozens of community driven support for different hardwares such as
RaspberryPi, Xilinx, and HidaV. Full layer index can be found in [Openembedded
Layer Index][oe-layer-index] page.

[oe-layer-index]: http://www.openembedded.org/wiki/LayerIndex


Author
======
Eren Türkay, TA1AET, (eren .--.-. hambedded.org)
