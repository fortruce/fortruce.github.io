---
title: "React.js Tetris Clone"
---

I have been really interested in trying out [React.js](http://facebook.github.io/react/index.html) recently. My previous post about [matrices in Clojure](http://josephmr.com/rolling-your-own-matrices-in-clojure/) originated from starting to build a Tetris clone in Clojure as a reason to use [@swannodette](https://twitter.com/swannodette)'s [Om](https://github.com/swannodette/om) interface to React.js. I wrapped up most of the core Tetris code in Clojure and decided it would probably be best to have an understanding of React before I tried to use Om. So I decided to first write Tetris in Javascript on React. For having no previous Javascript or frontend experience, this has gone suprisingly well.

This post will assume very basic knowledge of React ([tutorial](http://facebook.github.io/react/docs/tutorial.html)) and *very* basic Javascript.

Disclaimer: I am very new to React and Javascript. The code that follows was hacked together quickly and most likely doesn't reflect best practices.

###Displaying a Board

The first step is getting a board up on screen to start working with. I used the basic template for React from the tutorial for my html with a change to the CDN as it appeared to not work:

```language-markup
<!-- index.html -->
<html>
  <head>
    <title>React Tetris</title>
    <script src="//cdnjs.cloudflare.com/ajax/libs/react/0.10.0/react.js"></script>
    <link rel="stylesheet" type="text/css" href="css/main.css">
  </head>
  <body>
    <div id="target"></div>
    <script src="build/tetris.js"></script>
  </body>
</html>
```

The `#target` div is where I will be instructing React to render my game component. Also worth noting is that I am running `jsx --watch src/ build/` in the background instead of using the `JSXTransformer` simply because I found it easier to use, YMMV.

The board will be `10x21` and respresented by nested vectors:

```language-javascript
// tetris.js
function repeat (x, n) {
	// repeat x n times
	var z = [];
	for (i = 0; i < n; i++) {
		z.push(x);
	}
	return z;
}

function emptyBoard() {
	return repeat(repeat(0, 10), 21);
}
```

Now we need to build a React component to hold and display the game board.

```language-javascript
var Game = React.createClass({
  render: function() {
    var i = 0;
    var gridRows = this.props.board.map(function (row) {
      return <GridRow row={row} key={i++}/>;
    });
    return (
      <div className="container">
        <div className="grid-container">
        {gridRows}
        </div>
      </div>
    );
  }
});
  
var GridRow = React.createClass({
  render: function() {
  	var i = 0;
    var gridCells = this.props.row.map(function (cell) {
      return <GridCell key={i++} value={cell}></GridCell>;
    });
    return (
      <div className="grid-row">
       {gridCells}
      </div>
    );
  }
});

var GridCell = React.createClass({
  render: function() {
    return (
      <div className="grid-cell">{this.props.value}</div>
    );
  }
});
```

This is a decent chunk of code, but it is fairly simple. `Game` gets passed the board representation through its `props` in `board`. It builds an array of `GridRow` elements, one for each row of the board, passing along the row in the `row` prop. It also assigns a unique `key` for each element (this isn't strictly necessary, but React complains if you don't and is less performant if not done). `GridRow` operates similarly, creating a `div` for each individual cell and setting its content to be the `value` prop passed down from `GridRow`. With a little bit of css help from [2048](http://gabrielecirulli.github.io/2048/) (I said I don't have frontend experience...) you should see an actual grid!

{<1>}![Basic Tetris Board](/content/images/2014/May/basic-tetris-board.png)

### Building State
Tetris is an extremely easy game to build on top of React due to it's state being extremely easy to model. The only information necessary to show the game at any moment in time is the **board**, the currently descending **piece**, the piece's **position**, and the **state** of the game. We already have the **board** representation, and the pieces are very similar.

```language-javascript
const pieces = [
  [[0, 0, 2, 0],
   [0, 0, 2, 0],
   [0, 0, 2, 0],
   [0, 0, 2, 0]],
  [[0, 3, 0],
   [0, 3, 3],
   [0, 0, 3]],
  [[0, 0, 4],
   [0, 4, 4],
   [0, 4, 0]],
  [[5, 5],
   [5, 5]],
  [[6, 0, 0],
   [6 ,6, 6],
   [0, 0, 0]],
  [[0, 0, 7],
   [7, 7, 7],
   [0, 0, 0]],
  [[0, 8, 0],
   [8, 8, 8],
   [0, 0, 0]]];
   
function getPiece () {
	var r = Math.floor((Math.random() * 7));
    return pieces[r];
}
```
All the pieces are represented by `n x n` matrices with nonzero values representing the piece. The different values are used for coloring, but for now it's just important to note that they are all `!== 0`. Also included is a quick function to get a random piece. If the logic isn't clear, remember that `Math.random()` returns a float between `0 (inclusive)` and `1 (exclusive)`.

The **position** of the falling piece can easily be represented as a `[x, y]` coordinate pair. One thing to be careful of is that indexing into our board with **position** in this way must be done with `board[y][x]`, **NOT** `[x][y]`.

Finally, the **state** of the game can be either `"start"`, `"playing"`, or `"gameover"`.

All of state should be moved into the actual React `this.state` of the `Game` component.

```language-javascript
 var Game = React.createClass({
  getInitialState: function() {
  	return {board: emptyBoard(),
  	        piece: [],
  	        pos:   [4, 0],
  	        state: "start"};
  },
  // ...
});
```
***Note*: Be sure to remove the `board` from the props and change references to `this.props.board` to `this.state.board`!**

### Starting a Game
Now that the initial game state is set up, we need to interact with the game to start playing. Also, we are sorely missing any actual game loop so far!

First, a quick button to put the game in the `"playing"` state:

```language-javascript
var Game = React.createClass({
  //...
  startGame: function(event) {
    this.setState({state: "playing",
    			   board: emptyBoard(),
                   piece: getPiece(),
                   pos:   [4, 0]});
    },
  //...
  render: function() {
    // ...
    return (
      <div className="container">
/* + */ <button type="button" onClick={this.startGame}>START</button>
        <div className="grid-container">
        {gridRows}
        </div>
      </div>
    );
  }
});
```

The `startGame` function is responsible for setting the state to a good initial starting state and putting the game into the `"playing"` state. We reset the board to an empty board, get a new piece, and start it in roughly the center of the board at the top. To *start* the game, we add a `button` to the `.container` and set it's `onClick` handler to our `startGame` function. This is all nicely wired up for us by React. Now this button doesn't visually do anything, but if you have [React's extension](http://facebook.github.io/react/blog/2014/01/02/react-chrome-developer-tools.html) to the Chrome dev tools, then you can inspect the `Game`'s actual React `this.state` and see what affect pressing **START** had.

{<2>}![Pressing START](/content/images/2014/May/pressing-start.png)

Now, for a falling piece. We can use `setInterval` to fire off a game loop function that will descend the active piece. This function needs to modify the state of `Game` so that is where it should reside.

```language-javascript
function outOfBounds (bx, by) {
  return by >= 21 || by < 0 || bx < 0 || bx >= 10;
}

function isValidState (board, piece, pos) {
  for (row = 0; row < piece.length; row++) {
	for (col = 0; col < piece[0].length; col++) {
	  by = row + pos[1];
	  bx = col + pos[0];
	  if (outOfBounds(bx, by)) {
		if (piece[row][col] > 0) {
	      return false;
		}
		continue;
	  }
	  if ((board[by][bx] !== 0) && (piece[row][col] !== 0)) {
	    return false;
	  }
	}
  }
  return true;
}

function mergePiece (board, piece, pos) {
  var newBoard = [];
  for (i = 0; i < board.length; i++) {
 	newBoard.push(board[i].slice(0));
  }
  //merge in the new piece, assume this is a valid state (no overlapping piece)
  for (row = 0; row < piece.length; row++) {
	for (col = 0; col < piece[0].length; col++) {
      by = row + pos[1];
	  bx = col + pos[0];
	  if (outOfBounds(bx, by))
		continue;
	  newBoard[by][bx] += piece[row][col];
	}
  }
  return newBoard;
}

var Game = React.createClass({
  // ...
  loop: function() {
    if (this.state.state !== "playing")
      return;
    var newPos = this.state.pos.slice(0);
    newPos[1]++;
    if (isValidState(this.state.board, this.state.piece, newPos))
      this.setState({pos: newPos});
    else {
      var mergedBoard = mergeBoard(this.state);
      var newPiece = getPiece();
      this.setState({board: mergedBoard,
      				 piece: newPiece,
                     pos: [4, 0]});
    }
  },
  // ...
});
```

The first 3 functions are pure game logic. `mergePiece` is used to persist the currently falling piece into the board. Note that this function makes a copy of the board then operates on that. You should not modify a components state unless through the use of `this.setState`. `isValidState` is used to check if the `piece` can occupy it's `pos` on the board. This is used for collision detection of the blocks and making sure the blocks stay within the bounds of the board.

This code is not yet active. We have to wire up the loop with `setInterval`. We can do this inside of React's `componentDidMount` lifecycle function:

```language-javascript
var Game = React.createClass({
  // ...
  componentDidMount: function() {
    setInterval(this.loop, 250);
  },
  // ...
});
```

Now, if we run this there are a few quirks. In `render` we are showing the board, but the currently falling piece isn't merged into the board until it has collided. We have to modify render to display the piece that is currently falling:

```language-javascript
var Game = React.createClass({
  // ...
  render: function() {
    var i = 0;
    var mergedBoard = mergePiece(this.state.board, this.state.piece,
    							this.state.pos, true);
    var gridRows = mergedBoard.map(function (row) {
      return <GridRow row={row} key={i++}/>;
    });
    return (
      <div className="container">
        <button type="button" onClick={this.startGame}>START</button>
        <div className="grid-container">
        {gridRows}
        </div>
      </div>
    );
  }
});
```

The only change here is using a `mergedBoard` when displaying. Note that we are not actually changing the state of the board.

### Conclusion
This post has been pretty long and there is still a good deal to go. The  game does not support moving and rotating pieces or realize when the game is over. These are left as an exercise to the reader. Hopefully this has helped show how powerful React can be!