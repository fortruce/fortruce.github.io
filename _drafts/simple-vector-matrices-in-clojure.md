---
title: "Simple Vector Matrices in Clojure"
---

There are a few matrix solutions widely available in Clojure already ([core.matrix](https://github.com/mikera/core.matrix)), but I wanted to roll my own simple implementation for programming Tetris in Clojure. My ultimate goal being to use [Om](https://github.com/swannodette/om) to program a simple browser Tetris clone. I ran into a few quirks along the way.

###Pieces
I decided to represent the pieces as simple nested vectors:
```prettyprint lang-clj
    (def pieces [[[0 0 0 0]
                  [0 0 0 0]
                  [1 1 1 1]
                  [0 0 0 0]]
                 [[1 1 0]
                  [1 0 0]
                  [1 0 0]]
                 [[1 1]
                  [1 1]]
                  ; ... you get the point
                  ])
```

###Rotation
So for some reason, rotation comes before the game board. The rotation functions took a bit of thought and a few moments on the whiteboard to work it out in my head. Transposing a matrix represented in this way is really easy:

```prettyprint lang-clj
(defn transpose [m]
  (apply mapv vector m))
```
(Note the use of `mapv` here instead of `map`, this will become important later...)

However, transposition is not rotation.

{<2>}![Matrix Transposition](/content/images/2014/May/matrix-transpose-3-by-3.gif)

It does get us closer though, observe that reversing the rows of the transposed matrix results in a right rotation, and reversing the columns results in a left rotation. This is undoubtedly something I should have learned in linear algebra had I paid more attention but nothing a few seconds and a whiteboard can't remedy. With those observations, rotation in both directions is simply:

```prettyprint lang-clj
;; spoiler alert, there is a problem here...
(defn rotate-right [m]
  (->> m
       transpose
       (mapv reverse)))

(defn rotate-left [m]
  (->> m
       transpose
       reverse))
```

###Collision and the Game Board
So almost all of the main Tetris logic lies in figuring out if a particular falling piece has collided with the growing pile of not quite finished lines. First we need a board representation:

```prettyprint lang-clj
(def board (vec (for [y (range 21)] (vec (repeat 10 false)))))

(def game {:board board :piece {:piece (get-block) :pos [5 0]}})
```
Here I define a standard `10x21` board initialized to all `false`. At this time we can assume that the pieces are also represented as `true` & `false`, not `0` & `1`. In the game map, `[:piece :pos]` represents the upper left corner of the falling piece and `[:piece :piece]` is the actual matrix of the piece. Thus the functions must check if the current piece can occupy it's position on the board:

```prettyprint lang-clj
(defn is-valid?
  "Determine if a game state is valid (is current piece place-able)."
  [{board :board {:keys [piece pos]} :piece}]
  (let [[x y] pos
        coords (for [rows (range (count piece))
                     cols (range (count (first piece)))]
                 [[rows cols] [(+ y rows) (+ x cols)]])]
    (not-any? true? (map (fn [[c cc]]
                           (and (get-in board cc) (get-in piece c)))
                         coords))))
```

This is a relatively less than readable way to iterate over each coordinate in the piece and it's destination in the gameboard and ensure that two `true`s aren't trying to occupy the same space. The `for` generates a tuple of coordinate pairs, the first the index into the piece and the second the index into the game board for that part of the piece. The math behind the coordinate pairs can be a little (read: **lot**) confusing because of the difference between how I represent pos ([0 1] the first element of the second row) and how get-in operates based on our representation of a matrix ([0 1] gets the second element of the first row).

###Why It Doesn't Work...
This hasn't all been for naught, but it's true, the code doesn't quite work...sometimes. Even more infuriating is that it doesn't throw an exception, it just always returns false sometimes. Let me be more specific, it always returns false (even when there should be a collision) when I first call on of the rotation functions on the piece. The root of the problem boils down to `get`.

```prettyprint lang-clj
(get [0 1 2 3] 0)
; 0
(get '(0 1 2 3) 0)
; nil
```
This is pretty straightforward, Clojure `lists` do not have access by index, which is what `get` is all about. So if `get` doesn't work, `get-in` also doesn't work. But wait! My pieces are nested vectors as defined!...And then I had to go and rotate them.

```prettyprint lang-clj
(rotate-left [[0 1] [0 1]])
; ([1 1] [0 0])
```
Note the return value is actually a `seq` of `vector`s. I've lost my index access!

###Getting Fun Back
I actually made a post in the [Clojure Google Group](https://groups.google.com/forum/#!topic/clojure/Al-U7QQmM8s) and got some pretty quick replies. It seems the best way to handle this is to ensure that all input and output of matrix transformation functions are actually matrices. This is a great plug for Clojure's pre and post conditions.

```prettyprint lang-clj
(defn matrix? [m]
  (and (vector? m)
       (every? vector? m)))
       
(defn bad-rotate-left [m]
  {:pre [(matrix? m)] :post [(matrix? %)]}
  (->> m
       transpose
       reverse))
       
(bad-rotate-left '([0 1] [0 1]))
; java.lang.AssertionError: Assert failed: (matrix? m)
(bad-rotate-left [[0 1] [0 1]])
; java.lang.AssertionError: Assert failed: (matrix? m)
```
Awesome, so now our code squawks immediately when it has a problem and doesn't silently mess with heads later on down the line. Anyway, we should fix this by making sure we output a valid matrix (Note: this is why the use of `mapv` instead of `map` is important!).

```prettyprint lang-clj
(defn rotate-right [m]
  {:pre [(matrix? m)] :post [(matrix? %)]}
  (->> m
       transpose
       (mapv #(-> % reverse vec))))

(defn rotate-left [m]
  {:pre [(matrix? m)] :post [(matrix? %)]}
  (->> m
       transpose
       reverse
       vec))
```

###Conclusion
This has already been quite a learning experience, and I am still pretty far from actually having a working Tetris game. Going forward I am very eager to work with core.matrix instead, but getting more familiar with the core Clojure functions is definitely worth it.