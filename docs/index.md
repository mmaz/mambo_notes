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

There are a few options available for camera access. (This is a non-exhaustive list!)

1. The downward camera is currently available in Matlab (see Piazza for details, link to be added)
2. Options for directly accessing the front camera stream in Matlab are being explored (more information to come...)
3. The Python library [`pyparrot`](https://pyparrot.readthedocs.io/en/latest/) has [documentation on accessing the downward camera](https://pyparrot.readthedocs.io/en/latest/quickstartminidrone.html#demo-of-the-ground-facing-camera). Note the docs state that this is only available in wifi-mode (not bluetooth-mode).
4. [GStreamer](https://gstreamer.freedesktop.org/documentation/), a Linux command-line-tool, supports front-camera access (including from within python)

I (Mark) have been using GSteamer from within Python, which the documentation below will explain in some more detail.

For projects using the cameras, I highly recommend starting with a few representative data-collects. For instance, if you wanted to track a person, begin by recording some videos from the drone's camera. For instance, record some videos of one person in frame, multiple people in frame, and videos without people. Record videos with the drone flying or with the drone hand-carried. You can use GStreamer for all these collects. Then, these files can be used for exploration and testing without needing to rely on the short battery life of the drone.

## Installing GStreamer

To install GStreamer on Linux, you can follow [the instructions in their documentation](https://gstreamer.freedesktop.org/documentation/installing/on-linux.html). On Ubuntu, I used this command (taken from the above documentation):

```shell
$ sudo apt-get install libgstreamer1.0-0 gstreamer1.0-plugins-base \ 
   gstreamer1.0-plugins-good gstreamer1.0-plugins-bad \ 
   gstreamer1.0-plugins-ugly gstreamer1.0-libav gstreamer1.0-doc gstreamer1.0-tools
```

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

However (as far as I know), most prepackaged versions of OpenCV (e.g., the one you'll find via `apt-get`, `pip`, or `conda`) do not come with internal support for GStreamer. You'll have to download and compile OpenCV with support for GStreamer yourself.

First, you'll need development packages for gstreamer to compile opencv against (the interested reader can see [this stackoverflow question]( https://stackoverflow.com/questions/37678324/compiling-opencv-with-gstreamer-cmake-not-finding-gstreamer) for details), but essentially: 

```bash
$ apt-get install libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev
```

Now you'll need a bunch of dependencies for building OpenCV from source:

```bash
$ sudo apt-get install build-essential cmake pkg-config \
      libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev \
      libavcodec-dev libavformat-dev libswscale-dev  \
      libv4l-dev libxvidcore-dev libx264-dev \
      libgtk2.0-dev libgtk-3-dev \
      libatlas-base-dev gfortran \
      python2.7-dev python3-dev
```

I will place OpenCV in the home directory (but you can move it elsewhere)

```bash
cd ~ #move to home directory
```

The following command should download, build, and install OpenCV 3.3.0:

```bash
wget -O opencv.zip https://github.com/Itseez/opencv/archive/3.3.0.zip
unzip opencv.zip
wget -O opencv_contrib.zip https://github.com/Itseez/opencv_contrib/archive/3.3.0.zip
unzip opencv_contrib.zip
cd ~/opencv-3.3.0/
mkdir build
cd build
cmake -D CMAKE_BUILD_TYPE=RELEASE \
      -D CMAKE_INSTALL_PREFIX=/usr/local \
      -D INSTALL_PYTHON_EXAMPLES=OFF \
      -D WITH_V4L=ON \
      -D WITH_GSTREAMER=ON \
      -D BUILD_opencv_cnn_3dobj=OFF \
      -D BUILD_opencv_dnn_modern=OFF \
      -D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-3.3.0/modules \
      -D BUILD_EXAMPLES=OFF ..
make -j4
sudo make install
sudo ldconfig
```

The relevant option from the above build and install is `-D WITH_GSTREAMER=ON`, which is not a default flag for OpenCV.

## Python camera stream access

Now you should be able to access the drone's stream from within Python. (Re)start the drone and connect to the access point over WiFi. Then run this script:

```python
import cv2
cv2.namedWindow("frame", cv2.WINDOW_NORMAL)
gst = "rtspsrc location=rtsp://192.168.99.1/media/stream2 latency=10 ! rtph264depay ! h264parse ! avdec_h264 ! videoconvert ! appsink"
cap = cv2.VideoCapture(gst)
while(cap.isOpened()):
    ret, frame = cap.read()
    # you can add your processing here on the frame
    cv2.imshow('frame',frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
```

Pressing `q` will close the window.

**Troubleshooting:** If the above code doesn't work, you can stream in a 'test' video from gstreamer to verify that python is importing the *correct* version of OpenCV (i.e., the one you compiled to support GStreamer). First, inspect OpenCV's build information:

```python
from __future__ import print_function
import cv2
print(cv2.getBuildInformation())
```

Make sure you see the line `GStreamer: YES` in the output.

Next, use the `videotestsrc` to make sure Python can display a gstreamer video source using OpenCV:

```
import cv2
cv2.namedWindow("frame", cv2.WINDOW_NORMAL)
gst = "videotestsrc ! appsink"
cap = cv2.VideoCapture(gst)
while(cap.isOpened()):
    ret, frame = cap.read()
    cv2.imshow('frame',frame)
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
cap.release()
cv2.destroyAllWindows()
```

