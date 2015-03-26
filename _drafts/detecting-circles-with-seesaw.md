---
title: "Detecting Circles with Seesaw"
---

Last time I left off writing out an image captured through OpenCV to the filesystem. That's nice, but it would be way better if it would just pop up on screen so I could see the changes. Working with image detection code, I ultimately want side-by-side comparison of raw image and detected image:

![side-by-side][side-by-side]

##Seesaw

First thing first, let's create a frame to hold the image. It's easy to create a Seesaw frame and throw a label with the icon set to the image, but OpenCV gives us a `Mat` and the icon expects a `BufferedImage`.


{% highlight clj %}
(defn mat-to-bufimg
  "Converts a opencv Mat to a Java BufferedImage"
  [mat]
  (let [mob (MatOfByte.)
        enc (Highgui/imencode ".png" mat mob)]
    (when enc
      (-> mob
          .toArray
          (ByteArrayInputStream.)
          ImageIO/read))))
{% endhighlight %}


A little bit of necessary boiler-plate.

Then just throw that into a Seesaw frame:


{% highlight clj %}
(->> img
     mat-to-bufimg
     (label :icon)
     (frame :title "Robotron" :content)
     pack!
     show!)
{% endhighlight %}

![Seesaw frame][seesaw-frame]

##Hotswapping

Popping up frames is great, but every time it takes focus and must be closed. It would be better if we could hotswap the image inside the frame at will from the REPL, and we can with Seesaw binding and atoms!



{% highlight clj %}
; create an atom
(def lx (atom (buffered-image 640 480)))
; create label to hold the image
(def llabel (label))
; create the frame
(def f (frame :title "Robotron"
              :content llabel
              :minimum-size [640 :by 480]))
; bind the atom to the label
(bind lx (property llabel :icon))
; show the frame
(-> f pack! show!)
{% endhighlight %}



Now the frame is visible, but we haven't shown the image yet. This is easy enough, `reset!`


{% highlight clj %}
(reset! lx (mat-to-bufimg img))
{% endhighlight %}


It's possible to replay the webcam in this way just like a video.


{% highlight clj %}
(dotimes [x 60]
  (let [mm (Mat.)]
    (when (.read vc mm)
      (reset! lx (mat-to-bufimg mm)))))
{% endhighlight %}


##Conclusion

Using this its easy to pop up two side by side images, one the raw image, the second transformed by OpenCV functions such as in my example above: Gaussian Blur, Grayscale, Hough Circles. Doing all this from the repl is amazing fun!


[side-by-side]: {{ site.url }}/images/side-by-side.jpg
[seesaw-frame]: {{ site.url }}/images/seesaw-frame.jpg
