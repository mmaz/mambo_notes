# Parrot Mambo Notes (MIT 16.30/16.31)

The following notes may help you connect to the front and/or downward camera for initial experimentation and projects.

!!! warning "Documentation is in-progress!"
    *Do you have tips, resources, or corrections to add? Did you find something frustrating or unhelpful and want to remove it?* [Issues and pull-requests are welcome!](https://github.com/mmaz/mambo_notes/issues)

## Overview

The Mambo can support two cameras: 

* a downward-facing camera and, 
* (optionally) a front-facing camera, via the FPV accessory which snaps on top of the drone.

!!! note
    Simultaneoulsy streaming both cameras is still **untested** - please [make note](https://github.com/mmaz/mambo_notes/issues) if you test this!

There are a few options available for camera information. (This is a non-exhaustive list!)

1. The downward camera is currently available in Matlab (see Piazza for details, link to be added)
2. Options for directly accessing the front camera stream in Matlab are being explored (more information to come...)
3. The Python library [`pyparrot`](https://pyparrot.readthedocs.io/en/latest/) has [documentation on accessing the downward camera](https://pyparrot.readthedocs.io/en/latest/quickstartminidrone.html#demo-of-the-ground-facing-camera). Note the docs state that this is only available in wifi-mode (not bluetooth-mode).
4. [GStreamer](https://gstreamer.freedesktop.org/documentation/), a Linux command-line-tool, supports front-camera access (including from within python)

I (Mark) have been using GSteamer from within Python, which the documentation below will explain in some more detail.

## Installing GStreamer

To install GStreamer on Linux, you can follow [the instructions in their documentation](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html). 

## WiFi access to the Mambo

To connect via GStreamer you will need to connect to your drone over WiFi. Power on your drone with the camera attachment connected, and then look for the WiFi network which is created by the drone. 

!!! danger "Caveat"
    WiFi connections to the drone are flaky in my experience. This connection process takes multiple tries on my drone (within 4-5 tries usually, but up to 10). I usually pull the battery out and re-insert for each try. 

The network SSID will be called, e.g., `Mambo_12345`, with an ID# corresponding to your drone. Once you are connected, you can use gsteamer to test streaming an image to your computer (on the command line, without python) or connect to the stream within python.

!!! warning "re-connecting not supported?"
    I usually have to restart the drone in between camera stream connections, i.e., I can't test the connection with gstreamer on the command line, and then start a processing script in python.

## Viewing the front camera  with GStreamer

After you have installed gstreamer and connected to your drone's wifi access point, you should be able to use the following command which opens a new window on your laptop to view the drone's camera feed:

```bash
gst-launch-1.0 rtspsrc location=rtsp://192.168.99.1/media/stream2 latency=10 ! decodebin ! autovideosink
```

`latency=10` is a [tuneable parameter](https://gstreamer.freedesktop.org/documentation/design/latency.html), to control the amount of buffering.

`Ctrl-C` at the terminal should end the stream.

Once this works, you can also test recording a video from the front camera feed. You will probably need to power-cycle the drone first (pull the battery out and place it back in)

Note: you'll need to add a `-e` flag (end of stream) before `rtspsrc` so that the `.mp4` video file will be correctly closed when you `ctrl-c` the stream. See [this link](https://stackoverflow.com/a/25949095) for more details if interested.
 
```bash
gst-launch-1.0 -e rtspsrc location=rtsp://192.168.99.1/media/stream2 latency=10 ! decodebin ! x264enc ! mp4mux ! filesink location=file1.mp4
```

You can specify your preferred filename at the end of the above command (e.g., `file1`).

## Installing OpenCV

!!! Note
    This is optional - if you just want to test with GStreamer, for instance, to record a video from the front camera, you don't need to use OpenCV.

OpenCV will allow you to access the front camera stream programmatically (e.g., via python or c++).

First, you'll need development packages for gstreamer to compile opencv against (the interested reader can see [this stackoverflow question]( https://stackoverflow.com/questions/37678324/compiling-opencv-with-gstreamer-cmake-not-finding-gstreamer) for details), but essentially: 

```bash
$ apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
```

