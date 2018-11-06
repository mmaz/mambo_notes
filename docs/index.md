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

## WiFi access to the Mambo

To connect via GStreamer you will need to connect to your drone over WiFi. Power on your drone with the camera attachment connected, and then look for the WiFi network which is created by the drone. 

!!! danger "Caveat"
    WiFi connections to the drone are flaky in my experience. This connection process takes multiple tries on my drone (within 4-5 tries usually, but up to 10). I usually pull the battery out and re-insert for each try. 

The network SSID will be called, e.g., `Mambo_12345`, with an ID# corresponding to your drone. Once you are connected, you can use gsteamer to test streaming an image to your computer (on the command line, without python) or connect to the stream within python.

!!! warning "re-connecting not supported?"
    I usually have to restart the drone in between camera stream connections, i.e., I can't test the connection with gstreamer on the command line, and then start a processing script in python.

## GStreamer

Viewing the front camera 