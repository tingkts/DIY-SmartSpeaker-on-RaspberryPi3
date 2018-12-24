# How to port Amazon AVS Device SDK on Raspberry Pi 3 Model B running [Raspbian Stretch with Desktop](https://www.raspberrypi.org/downloads/raspbian/)



1. ## Follow this official guide：

   https://github.com/alexa/avs-device-sdk/wiki/Raspberry-Pi-Quick-Start-Guide-with-Script

2. ## If sample app can't run normally, then try to build AVS Device SDK manually step by step.



   The first step is to make sure your machine has the latest package lists and then install the latest version of each package in that list:

   ​	

   ```
   sudo apt-get update && sudo apt-get upgrade -y
   ```





   We need somewhere to put everything, so let's create some folders. This guide presumes that everything is built in {HOME}, which we will presume is your home directory. If you choose to use different folder names, please update the commands throughout this guide accordingly:

   ```
   mkdir sdk-folder && cd sdk-folder && mkdir sdk-build sdk-source third-party application-necessities
   ```



   Now, let's set up our toolchain:

   ```
   sudo apt-get install -y git gcc cmake openssl clang-format
   ```





   Next, we need to download dependencies available from apt-get:

   ```
   sudo apt-get install -y openssl libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-good libgstreamer-plugins-good1.0-dev libgstreamer-plugins-bad1.0-dev gstreamer1.0-libav pulseaudio doxygen libsqlite3-dev repo libasound2-dev
   ```





   The AVS Device SDK relies on an HTTP2 connection with the Alexa service. To address this requirement, we're going to build curl with nghttp2 and openssl. If you would prefer to build with mbed TLS, [click here for instructions](https://github.com/alexa/avs-device-sdk/wiki/Build-libcurl-with-mbed-TLS-and-nghttp2):





   ------

   ## nghttp2

   Install the [latest version of nghttp2](https://github.com/nghttp2/nghttp2/releases) to your principle working directory $WD. Run these commands:

   Note: This example presumes you have downloaded v1.19.0. Make sure you adjust the commands below to match the version of nghttp2 you have downloaded.

   - wget <https://github.com/nghttp2/nghttp2/releases/download/v1.19.0/nghttp2-1.19.0.tar.gz>

   - tar -xvf nghttp2-1.19.0.tar.gz

   - cd nghttp2-1.19.0

   - autoreconf -i



     Note: You may need to run the following command to obtain the right tools: sudo apt-get update && sudo apt-get install g++ make binutils autoconf automake autotools-dev libtool pkg-config zlib1g-dev libcunit1-dev libssl-dev libxml2-dev libev-dev libevent-dev -y

   - automake

   - autoconf

   - ./configure --prefix=$WD/nghttp2/

   - make

   - sudo make install





   ------

   ## mbed TLS

   Install the latest version of [mbed TLS](https://github.com/ARMmbed/mbedtls/releases) to your principle working directory $WD. Run these commands:

   Note: This example presumes you have downloaded v2.4.0. Make sure you adjust the commands below to match the version of mbed TLS you have downloaded.

   - wget <https://tls.mbed.org/download/mbedtls-2.4.0-apache.tgz>

   - tar -xvf mbedtls-2.4.0-apache.tgz

   - cd mbedtls-2.4.0/

   - Build with CMake:

   - - cmake -DCMAKE_INSTALL_PREFIX=$WD/mbedtls/ -DUSE_SHARED_MBEDTLS_LIBRARY=On
     - make
     - sudo make install



   ------

   ## libcurl

   Install the latest version of [libcurl (7.50.2 or later)](https://curl.haxx.se/download.html). Run these commands:

   Note: This example presumes you have downloaded v7.53.0. Make sure you adjust the commands below to match the version of libcurl you have downloaded.

   1. - wget <https://curl.haxx.se/download/curl-7.53.0.tar.gz>
      - tar -xvf curl-7.53.0.tar.gz
      - cd curl-7.53.0/
      - LIBS="-lpthread" LDFLAGS="-Wl,-R$WD/mbedtls/lib" ./configure --with-nghttp2=$WD/nghttp2/ --without-ssl --with-mbedtls=$WD/mbedtls --prefix=$WD/curl/
      - The configuration summary should be printed to your terminal. Make sure that SSL support (mbed TLS) and HTTP/2 support (nghttp2) are enabled.
      - make
      - sudo make install



   ------

   ## PortAudio

   PortAudio is required to record microphone data. Run this command to install and configure PortAudio

   ```
   $ cd ~/sdk-folder/third-party
   $ wget -c http://www.portaudio.com/archives/pa_stable_v190600_20161030.tgz && tar zxf pa_stable_v190600_20161030.tgz && cd portaudio && ./configure —without-jack && make
   ```



   ------

   # avs-device-sdk

   Now it's time to clone the AVS Device SDK. Navigate to your sdk-source folder and run this command:

   ```
   $ cd ~/sdk-folder/sdk-source && git clone <git://github.com/alexa/avs-device-sdk.git>
   ```





   Assuming the download succeeded, the next step is to build the SDK. This command does a few things:

   - It declares that PortAudio is used to capture microphone data and points to its lib path and includes directory

   - It declares that gstreamer is installed and will be used when you build the SampleApp

   - It declares that the wake word detector is OFF

   - - Linux supports wake word detectors from Sensory and [Kitt.ai](http://kitt.ai/). Each requires a license from the provider. For instructions to build with a wake word detector, please see [Build Options](https://github.com/alexa/avs-device-sdk/wiki/Build-Options).



   IMPORTANT: Replace {HOME} with the full path to your home directory (do not use ~/):

   ```
   $ cd /{HOME}/sdk-folder/sdk-build
   $ cmake /{HOME}/sdk-folder/sdk-source/avs-device-sdk \
   -DSENSORY_KEY_WORD_DETECTOR=OFF \
   -DGSTREAMER_MEDIA_PLAYER=ON \
   -DPORTAUDIO=ON \
   -DPORTAUDIO_LIB_PATH=/{HOME}/sdk-folder/third-party/portaudio/lib/.libs/libportaudio.a \
   -DPORTAUDIO_INCLUDE_DIR=/{HOME}/sdk-folder/third-party/portaudio/include
   $ make
   ```








     


