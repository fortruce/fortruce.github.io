---
title: "Clojure and OpenCV"
---

I have recently been spending a lot of time learning Clojure. I am just now starting a small robotics project involving motion tracking, and I wanted to see how the REPL might be able to help me. I had a bit of trouble getting everything up and running, but after some fuss I have the beginnings of a nice exploratory platform.

![Robotron Control Panel][robotron-control-panel]

###Setup

For the most part, I attempted to follow the already existing Clojure + OpenCV [tutorial](http://docs.opencv.org/2.4/doc/tutorials/introduction/clojure_dev_intro/clojure_dev_intro.html). It got me pretty close to a working solution, but unfortunately it failed when trying to load the native OpenCV library.

{% highlight clj %}
user=> (clojure.lang.RT/loadLibrary org.opencv.core.Core/NATIVE_LIBRARY_NAME)
CompilerException java.lang.ClassNotFoundException:
org.opencv.core.Core, compiling:(NO_SOURCE_PATH:1:1)
{% endhighlight %}

It turns out this is an easy fix, which isn't quite noticeable for someone who is just learning Clojure and Leiningen. I updated my `project.clj` to point to a local copy of the native dll (so/dylib on other platforms).

{% highlight clj %}
(defproject test "0.1.0-SNAPSHOT"
  ...
  :jvm-opts ["-Djava.library.path=opencv_lib"]
  :injections [(clojure.lang.RT/loadLibrary org.opencv.core.Core/NATIVE_LIBRARY_NAME)]
  :dependencies [[org.clojure/clojure "1.5.1"]
				 [opencv/opencv "2.4.9"]
				 [opencv/opencv-native "2.4.9"]])
{% endhighlight %}

`opencv_lib` is just a relative path the the dll file.

{% highlight clj %}
user=> (clojure.lang.RT/loadLibrary org.opencv.core.Core/NATIVE_LIBRARY_NAME)
nil
{% endhighlight %}

**Viola!** (This might have taken longer than I would care to admit...)

###Play Time
Now let's capture some frames and throw them up on screen.

{% highlight clj %}
(import '[org.opencv.core Mat])
(import '[org.opencv.highgui VideoCapture])
(def vc (VideoCapture. 0))
(def m (Mat.))
(.read vc m)
; true
m
; #<Mat Mat [ 480*640*CV_8UC3, isCont=true,
;             isSubmat=false, nativeObj=0x1687bf18, 
;             dataAddr=0x1921e030 ]>
{% endhighlight %}

Well, it looks like that worked, but there is no `imshow` in Java's OpenCV api. Writing it out to a file is a good way to check that everything is workign so far.

{% highlight clj %}
user=> (Highgui/imwrite "resources/test.png" m)
; true
{% endhighlight %}

![Chewy Bar][chewy-bar]

###Conclusion
The tutorial on OpenCV's website was a little lacking, but overall getting OpenCV and Clojure playing together is pretty easy. Being able to quickly try things on the REPL has noticably sped up my programming, and I can't wait to dive into OpenCV further in Clojure.


[chewy-bar]: {{ site.url }}/images/chewy-bar.png
[robotron-control-panel]: {{ site.url }}/images/robotron-control-panel.jpg
