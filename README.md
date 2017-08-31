# A baby sleep monitor using a Raspberry Pi

This setup shows how to create a baby sleep monitor which is able to stream a low latency image stream from a Raspberry Pi to a computer with some additional features:

1. Motion detection to detect when the baby wakes up.
2. It also works with a Massimo oximeter to monitor O2 and heart rate sats.

## Setup
## Upgrade Raspberry Pi

Depending on how old your Raspberry Pi is, you might need to do an apt-get
update/upgrade in order to be able to compile Janus (which is not available
on apt as of this writing). On a terminal:

    sudo apt-get update
    sudo apt-get upgrade
    
This takes a while, so be patient.

## Setup Raspberry Pi Camera

First enable Raspberry Pi in the firmware:

    sudo raspi-config
    # Once the UI comes up, select the following options
    # Choose (5) Interfacing options
    # Choose (P1) Camera
    # Choose <Yes>
    # Finish

Now enable the Raspberry Pi Camera to work like a standard Linux video
webcam:

    sudo nano /etc/rc.local
    # Put the following line just before exit(0)
    sudo modprobe bcm2835-v4l2

(Optional) Disable the bright red LED

    sudo nano /boot/config.txt
    # Add the following line to the end:
    disable_camera_led=1

## Download the sleep monitor code

Now download this repo. Please note that the location of the repo is
hard-coded in the init.d script below. So if you download the repo to a
different location, modify that.

     cd ~
     mkdir code && code code
     git clone https://github.com/srinathava/raspberry-pi-stream-sleep-monitor.git
     cd raspberry-pi-sleep-monitor

## Setup gstreamer
This should be pretty simple since its available on apt. You can do:

    sudo apt-get install gstreamer-1.0
    
to install it.

## Test camera

Reboot your machine. Once it comes up:

    cd ~/code/raspberry-pi-sleep-monitor
    ./gstream_test_video.sh

This should display a small window with the current image from the
Raspberry Pi camera.

## Setup Janus

NOTE: Setting up and installing Janus is optional if you do not care about
audio from this setup

Janus provides a way to convert an audio-stream obtained from the webcam
into a WebRTC stream which is understood by many modern browsers.
Unfortunately, Janus is not available as a debian package as of now.
Following the instructions from
[here](https://www.rs-online.com/designspark/building-a-raspberry-pi-2-webrtc-camera),
you need to do:

     sudo aptitude install libmicrohttpd-dev libjansson-dev \
        libnice-dev libssl-dev libsrtp-dev libsofia-sip-ua-dev \
        libglib2.0-dev libopus-dev libogg-dev libini-config-dev \
        libcollection-dev pkg-config gengetopt libtool automake dh-autoreconf
     cd ~
     mkdir janus && cd janus
     git clone https://github.com/meetecho/janus-gateway.git
     cd janus-gateway
     sh autogen.sh
     ./configure --disable-websockets --disable-data-channels \
        --disable-rabbitmq --disable-docs --disable-mqtt --prefix=/opt/janus
     make
     sudo make install
     sudo make configs

## Install Janus

Install Janus plugin

     # NOTE: If you are already using Janus for something else, do not just
     # over-write this. Append the contents of janus.plugin.streaming.cfg
     # /opt/janus/etc/janus/janus.plugin.streaming.cfg
     sudo cp janus.plugin.streaming.cfg /opt/janus/etc/janus/janus.plugin.streaming.cfg

Install init.d from this repo

    sudo cp init.d/sleep-monitor /etc/init.d/ 

Now start the server

    sudo /etc/init.d/sleep-monitor start

    (or just reboot your pi)

## Use a browser

Now from any other computer in the local network, navigate to:

     http://ip.of.your.rpi/
