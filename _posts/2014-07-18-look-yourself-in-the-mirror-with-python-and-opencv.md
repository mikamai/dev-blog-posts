---
ID: 224
post_title: >
  Look yourself in the mirror with Python
  and OpenCV
author: Nicola Racco
post_date: 2014-07-18 12:21:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2014/07/18/look-yourself-in-the-mirror-with-python-and-opencv/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/92140422064/look-yourself-in-the-mirror-with-python-and-opencv
tumblr_mikamayhem_id:
  - "92140422064"
---
Let's see how to transform your screen into a mirror.
<!--more-->

![Python Mirror in action](https://dev.mikamai.com/wp-content/uploads/2014/07/python_mirror.jpg) {.aligncenter}

The starting point is rather easy:

```python
#!/usr/bin/env python

import numpy as np
import cv2

cap = cv2.VideoCapture(0)
# Set camera resolution. The max resolution is webcam dependent
# so change it to a resolution that is both supported by your camera
# and compatible with your monitor
cap.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, 800)

# If you have problems running this code on MacOS X you probably have to reinstall opencv with
# qt backend because cocoa support seems to be broken:
#   brew reinstall opencv --HEAD --qith-qt
cv2.namedWindow(&#039;frame&#039;, cv2.WND_PROP_FULLSCREEN)
cv2.setWindowProperty(&#039;frame&#039;, cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)

while True:
    ret, frame = cap.read()
    cv2.imshow(&#039;frame&#039;, frame)
    if cv2.waitKey(1) == 27:
        break

cap.release()
cv2.destroyAllWindows()
```

With this simple code (adapted from [the official doc](http://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_gui/py_video_display/py_video_display.html)), we have a fullscreen window showing the camera.

But we want the camera. So, first of all we have to transform this code to something more similar to a real app, written by a real dev.

```python
#!/usr/bin/env python

import numpy as np
import cv2

class Mirror:
  def __init__(self):
    self.cam = cv2.VideoCapture(0)
    self.__setupCamera()
    # If you have problems running this code on MacOS X you probably have to reinstall opencv
    # with qt backend because cocoa support seems to be broken:
    #   brew reinstall opencv --HEAD --qith-qt
    self.__setupWindow()

  # Set camera resolution. The max resolution is webcam dependent
  # so change it to a resolution that is both supported by your camera
  # and compatible with your monitor
  def __setupCamera(self):
    self.cam.set(cv2.CAP_PROP_FRAME_WIDTH, 1280)
    self.cam.set(cv2.CAP_PROP_FRAME_HEIGHT, 800)

  def __setupWindow(self):
    cv2.namedWindow(&#039;frame&#039;, cv2.WND_PROP_FULLSCREEN)
    cv2.setWindowProperty(&#039;frame&#039;, cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)

  def update(self):
    ret, frame = self.cam.read()
    cv2.imshow(&#039;frame&#039;, frame)

  def release(self):
    self.cam.release()
    cv2.destroyAllWindows()

if __name__ == &quot;__main__&quot;:
  mirror = Mirror()
  while True:
    mirror.update()
    if cv2.waitKey(1) == 27:
      break
  mirror.release()
```

Okay, **NOW** we can discuss the code.

If we run the script directly, it will:

- create a new Mirror
- update the mirror infinitely until the key _ESC_ is pressed
- dispose the Mirror

The `Mirror` constructor acquires the `VideoCapture` object referencing the webcam, sets the camera resolution (this is camera dependent) and sets the window to be fullscreen. You should change the resolution used in the code to use one supported by your camera, but be warned that greater resolution means lower fps.

To obtain a real mirror we have to flip the image horizontally adding in the Mirror class a new method responsible of getting the image and applying a filter on it:

```python
def read(self):
  ret, frame = self.cam.read()
  return cv2.flip(frame, 1)

def update(self):
  cv2.imshow(&#039;frame&#039;, self.read())
```

And here it’s the mirror! We’re just stopping here, but why don’t you try add more stuff ? As you can see [here](http://opencv-python-tutroals.readthedocs.org/en/latest/py_tutorials/py_imgproc/py_table_of_contents_imgproc/py_table_of_contents_imgproc.html), there are a lot of things OpenCV can do.