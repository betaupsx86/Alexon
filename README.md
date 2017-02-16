# Alexon
Alexa is without doubts the coolest voice assistant out there. What's even cooler is that you can integrate Amazon Voice Services into your embedded project with ease as explained in https://github.com/alexa/alexa-avs-sample-app. It would be great if we could use the Java client with Intel Edison, as an AVS wearable is pretty viable with this platform. Unfortunately, getting started with the sample app is a bit complicated, since Edison is headless and uses the Yocto build system.

The idea is to allow for a remote desktop connection to get a sense of how the app works and off course  build all dependencies required by the client which includes VLC.  We're going to use x11 forwarding to render the GUI in our remote machine. This is by far the easiest way as it doesn't require building remote desktop server like tightvnc which can be really frustrating.

The first thing to do is to set up our build environment to bitbake our Edison image. For that https://software.intel.com/en-us/node/593591 is a good place to start. Before building we need to add some features to the image. Edit auto.conf and add:

DISTRO_FEATURES_append = " alsa bluetooth x11 opengl"

Here x11 is for the X server and opengl is required for the VLC package build. Additionally, we need to add extra layers to find our recipes. Edit bblayers.conf and add:

  ${TOPDIR}/../poky/openembedded-core/meta \
  ${TOPDIR}/../poky/meta-openembedded/meta-multimedia \
  ${TOPDIR}/../poky/meta-openembedded/meta-gnome \ 

Git clone these layers into your workspace and checkout the dizzy branch which is what Edison uses. We also need to modify local.conf as follows in order to resolve the jpeg provider conflict ( between openembedded-core/meta and meta-openembedded/meta-multimedia) and whitelist the codecs that VLC uses:

 #PREFERED_PROVIDER
PREFERRED_PROVIDER_jpeg = "libjpeg-turbo"
PREFERRED_PROVIDER_jpeg-native = "libjpeg-turbo-native"

 #WHITELISTED
LICENSE_FLAGS_WHITELIST = "commercial_libav commercial_mpeg2dec commercial_x264"

Lastly we need to edit the VLC recipe so it compiles with the correct configuration. We need to enable or disable libva and avcodec separately. We don't have video acceleration hardware on the Edison so libva is useless. Modify vlc.inc as follows:

PACKAGECONFIG ?= " live555 libav"
PACKAGECONFIG[libva] = "--enable-libva,--disable-libva,libva"
PACKAGECONFIG[libav] = "--enable-avcodec,--disable-avcodec,libav"

With all this set we should be able to bitbake our image and then bitbake the VLC package.
To be continued...

