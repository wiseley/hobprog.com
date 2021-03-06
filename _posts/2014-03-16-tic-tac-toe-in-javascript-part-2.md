---
layout: post
title:  "Tic Tac Toe in JavaScript, Part 2"
tags: javascript game 
---

In <a href="/2014/03/13/tic-tac-toe-in-javascript-part-1.html">the last post</a> I introduced a Tic Tac Toe game with computer player implemented as a single HTML file. The game in that post was a proof of concept, with the computer player just randomly selecting a blank square for its turn. 

In this post, we'll add two strategies to bring it from 3 year old human capability to something more like a 7 year old's capability.

The first strategy looks for the best offensive opportunity. Instead of just randomly selecting a blank box, the `strategyBestOpporunity` function looks for a row, column or diaganol that is either wide-open or solely occupied by the computer. Here's the strategy function:

{% highlight javascript %}
// looks for a rol/col/diag that poses an opportunity for a win or nearest win
function strategyBestOpportunity() {
  var ix = -1, y = -1, x = -1, d = -1;
  var count = 0, countY = 0, countX = 0, countD = 0;
  // look for the best opportunity, to be passed to iterateRows/Cols
  var seekOpportunity = function(a,b,c,i){
    var match = 0;
    var opponent = 0;
    var blank = 0;
    if (a==computer) match++; else if (a==player) opponent++; else if (a=='') blank++;
    if (b==computer) match++; else if (b==player) opponent++; else if (b=='') blank++;
    if (c==computer) match++; else if (c==player) opponent++; else if (c=='') blank++;
    if (match>0 && match<3 && opponent==0) {
      if (match > count) {
        count = match;
        ix = i;
      }
    }
  };
  // get the best row
  iterateRows(seekOpportunity);
  countY = count;
  y = ix; 
  ix = -1; count = 0;
  // get the best column
  iterateCols(seekOpportunity);
  countX = count;
  x = ix;
  ix = -1; count = 0;
  // get the best diagonal
  iterateDiags(seekOpportunity);
  countD = count;
  d = ix;
  ix = -1; count = 0;
  // determine the winning strategy (col, row or diag)
  var turnX = -1, turnY = -1;
  if (countX >= countY && countX >= countD) {
    turnX = x;
    for (ix=0; ix<3; ix++) {
      if (val(x,ix)=='') { 
        turnY = ix;
        break;
      }
    }
  }
  else if (countY >= countX && countY >= countD) {
    turnY = y;
    for (ix=0; ix<3; ix++) {
      if (val(ix,y)=='') {
        turnX = ix;
        break;
      }
    }
  }
  else if (countD >= countX && countD >= countY) {
    if (val(1,1)=='') {turnX = 1; turnY = 1;}
    else if (d == 0) {
      if (val(0,0)=='') {turnX = 0; turnY = 0;}
      else {turnX = 2; turnY = 2;}
    }
    else {
      if (val(2,0)=='') {turnX = 2; turnY = 0;}
      else {turnX = 0; turnY = 2;}
    }
  } 
  if (turnX==-1 || turnY==-1) return false;
  else return [turnX,turnY];
}
{% endhighlight %}

The second strategy I've added takes care of the basic defense, looking for an imminent win on the other side and plugging the empty space that would close the deal. Here's that function.

{% highlight javascript %}
// looks for an imminent opponent win and prevents it
function strategyAvertLoss() {
  var isPlayer = function(val) { return val==player; }
  var isBlank = function(val) { return val==''; }
  var findImminentLoss = function(a,b,c,i){
    if (matches([a,b,c], isPlayer)==2 && matches([a,b,c], isBlank)==1) {
      ret = findInArray([a,b,c], isBlank);
      return true;
    } else return false;
  };
  var ret = -1;
  var ix = -1;
  // search rows
  ix = iterateRows(findImminentLoss);
  if (ix>-1 && ret>-1) return [ret, ix];
  // search columns
  ix = iterateCols(findImminentLoss);
  if (ix>-1 && ret>-1) return [ix, ret];
  // search diagonals
  ix = iterateDiags(findImminentLoss);

  if (ix>-1 && ret>-1) return convertDiag(ix, ret);
  else return false;
}
{% endhighlight %}

We now have three strategies, so here is the updated computerTurn function.

{% highlight javascript %}
// computer takes its turn
function computerTurn() {
  var strategies = [];
  if (option('avertLoss')) strategies.push(strategyAvertLoss);
  if (option('bestOpportunity')) strategies.push(strategyBestOpportunity);
  if (option('random')) strategies.push(strategyRandom);
  for (var i=0; i<strategies.length; i++) {
    var turn = strategies[i]();
    if (!turn) continue;
    val(turn[0], turn[1], computer);
    break;
  }
}
{% endhighlight %}

Here is the updated game in an IFRAME (<a href="/projects/tic-tac-toe/part-2.html">direct link, if you prefer</a>). Note that the various strategies can be enabled or disabled with the check boxes below the game board.

<iframe style="width:380px; height:670px;" src="/projects/tic-tac-toe/part-2.html"></iframe>

And here is the source code thus far. In part three, I'll add another strategy or two to plug some holes that allow a player who exploits them to win repeatedly.

{% highlight html %}
<!DOCTYPE html>
<!--
Single file Tic Tac Toe against the Computer
https://gist.github.com/wiseley/9458565/
Author: Matt Wiseley
License: GPL - http://www.gnu.org/licenses/gpl.html
-->
<html>
<head>
  <meta charset="UTF-8">
  <style>
    body { font-family: arial; font-size: 1.2em; background-color:#f7f7f7;}
    #board {
      width: 306px;
      height: 306px;
      border: 1px solid #ccc;
    }
    #board div {
      float: left;
      height: 100px;
      width: 100px;
      border: 1px solid #aaa;
      text-align: center;
    }
    #board div span {
      font-family: arial;
      font-size: 70px;
      margin-top: 10px;
      display: block;
    }
    .winner { background-color: lightgreen; }
    #status { font-size: 1.2em; font-weight: bold; }
  </style>
  <script src="http://code.jquery.com/jquery-1.11.0.min.js"></script>
  <script>
    var computer = 'X'; // the computer is X
    var player = 'O'; // you are O
    var turn = player; // the player goes first
    var xwins = 0, owins = 0; // win tallies
    
    /* =============================== */
    /* GAME BOARD NAVIGATION FUNCTIONS */
    
    // test each cell in each row with a function and return the index of
    // the first whose values causes the function to return true; 
    // return -1 if no row returns true
    function iterateCols(func) {
      return findInGrid(true, func);
    }
    function iterateRows(func) {
      return findInGrid(false, func);
    }
    function findInGrid(isRow, func) {
      for (var i=0; i<3; i++) {
        var a = isRow ? val(i,0) : val(0,i);
        var b = isRow ? val(i,1) : val(1,i);
        var c = isRow ? val(i,2) : val(2,i);
        if (func(a,b,c,i)) return i;
      }
      return -1;
    }
    // returns 0 if diag starting at 0,0 matches, 2 if starting at 2,0 matches, else -1
    function iterateDiags(func) {
      if (func(val(0,0), val(1,1), val(2,2),0)) return 0;
      else if (func(val(2,0), val(1,1), val(0,2),2)) return 2;
      else return -1;
    }
    // from top, left to right diag is 0, right to left is 2.
    // convert the [a,b,c] index of the diagonal based which way its going
    function convertDiag(iterateVal, ix) {
      //These:   00 01 02 20 21 22
      //become:  00 11 22 20 11 02
      if (iterateVal==0) {
        if (ix==0) return [0,0];
        if (ix==1) return [1,1];
        if (ix==2) return [2,2];
      }
      else {
        if (ix==0) return [2,0];
        if (ix==1) return [1,1];
        if (ix==2) return [0,2];
      }
    }

    // get the value of the square at x,y coords
    function val(x,y,newVal) {
      if (typeof newVal == 'undefined') {
        return $("#s"+x+""+y+" span").text();
      } 
      else {
        $("#s"+x+""+y).html("<span>"+newVal+"</span>");
      }
    }


    /* =================== */    
    /* GAME PLAY FUNCTIONS */

    function init() {
      // register event handler for player turns 
      $(document).ready(function() {
        $("#board div").click(function() {
          if (turn==player) {
            if (!$(this).find("span").length>0) {
              $(this).html("<span>"+player+"</span>");
              processTurn();
            }
          }
        });
      });
    }
    
    // check the puzzle for a win and swap turns
    function processTurn() {
      // winner requires 3 in a row of same value with no blanks
      var winnerTest = function(a,b,c) {
        return a==b & a==c && a!='';
      };
      // check straight across
      var y = iterateCols(winnerTest);
      if (y > -1) {
        finish(val(y,0), [[y,0],[y,1],[y,2]]);
        return;
      }
      // check up and down
      var x = iterateRows(winnerTest);
      if (x > -1) {
        finish(val(0,x), [[0,x],[1,x],[2,x]]);
        return;
      }
      // check diagonals
      var d = iterateDiags(winnerTest);
      if (d == 0) {
        finish(val(0,0), [[0,0],[1,1],[2,2]]);
        return;
      } 
      else if (d == 2) {
        finish(val(2,0), [[2,0],[1,1],[0,2]]);
        return;
      }
      // if there are no blanks, finish with a draw
      if (iterateCols(function(a,b,c) { return a=='' || b=='' || c==''; }) == -1) {
        finish();
      }
      // otherwise proceed to the next turn
      else {
        if (turn==computer) {
          turn = player;
        }
        else if (turn==player) {
          turn = computer;
          computerTurn();
          processTurn();
        }
        // else: the game is over, do nothing
      }
    }

    // looks for an imminent opponent win and prevents it
    function strategyAvertLoss() {
      var isPlayer = function(val) { return val==player; }
      var isBlank = function(val) { return val==''; }
      var findImminentLoss = function(a,b,c,i){
        if (matches([a,b,c], isPlayer)==2 && matches([a,b,c], isBlank)==1) {
          ret = findInArray([a,b,c], isBlank);
          return true;
        } else return false;
      };
      var ret = -1;
      var ix = -1;
      // search rows
      ix = iterateRows(findImminentLoss);
      if (ix>-1 && ret>-1) return [ret, ix];
      // search columns
      ix = iterateCols(findImminentLoss);
      if (ix>-1 && ret>-1) return [ix, ret];
      // search diagonals
      ix = iterateDiags(findImminentLoss);

      if (ix>-1 && ret>-1) return convertDiag(ix, ret);
      else return false;
    }

    // looks for a rol/col/diag that poses
    // an opportunity for a win or nearest win    
    function strategyBestOpportunity() {
      var ix = -1, y = -1, x = -1, d = -1;
      var count = 0, countY = 0, countX = 0, countD = 0;
      // look for the best opportunity, to be passed to iterateRows/Cols
      var seekOpportunity = function(a,b,c,i){
        var match = 0;
        var opponent = 0;
        var blank = 0;
        if (a==computer) match++; else if (a==player) opponent++; else if (a=='') blank++;
        if (b==computer) match++; else if (b==player) opponent++; else if (b=='') blank++;
        if (c==computer) match++; else if (c==player) opponent++; else if (c=='') blank++;
        if (match>0 && match<3 && opponent==0) {
          if (match > count) {
            count = match;
            ix = i;
          }
        }
      };
      // get the best row
      iterateRows(seekOpportunity);
      countY = count;
      y = ix; 
      ix = -1; count = 0;
      // get the best column
      iterateCols(seekOpportunity);
      countX = count;
      x = ix;
      ix = -1; count = 0;
      // get the best diagonal
      iterateDiags(seekOpportunity);
      countD = count;
      d = ix;
      ix = -1; count = 0;
      // determine the winning strategy (col, row or diag)
      var turnX = -1, turnY = -1;
      if (countX >= countY && countX >= countD) {
        turnX = x;
        for (ix=0; ix<3; ix++) {
          if (val(x,ix)=='') { 
            turnY = ix;
            break;
          }
        }
      }
      else if (countY >= countX && countY >= countD) {
        turnY = y;
        for (ix=0; ix<3; ix++) {
          if (val(ix,y)=='') {
            turnX = ix;
            break;
          }
        }
      }
      else if (countD >= countX && countD >= countY) {
        if (val(1,1)=='') {turnX = 1; turnY = 1;}
        else if (d == 0) {
          if (val(0,0)=='') {turnX = 0; turnY = 0;}
          else {turnX = 2; turnY = 2;}
        }
        else {
          if (val(2,0)=='') {turnX = 2; turnY = 0;}
          else {turnX = 0; turnY = 2;}
        }
      } 
      if (turnX==-1 || turnY==-1) return false;
      else return [turnX,turnY];
    }

    // go in a randomly selected blank space
    function strategyRandom() {
      // gather all the blank spots in an array
      var blanks = [];
      for (var x=0; x<3; x++) {
        for (var y=0; y<3; y++) {
          if (val(x,y)=='') blanks.push([x,y]);
        }
      }
      // return a random entry in the array of blanks
      if (blanks.length>0) {
        var r = Math.floor((Math.random()*blanks.length));
        return blanks[r];
      } 
      else return false;
    }

    // looks for an imminent opponent win and prevents it
    function strategyAvertLoss() {
      var isPlayer = function(val) { return val==player; }
      var isBlank = function(val) { return val==''; }
      var findImminentLoss = function(a,b,c,i){
        if (matches([a,b,c], isPlayer)==2 && matches([a,b,c], isBlank)==1) {
          ret = findInArray([a,b,c], isBlank);
          return true;
        } else return false;
      };
      var ret = -1;
      var ix = -1;
      // search rows
      ix = iterateRows(findImminentLoss);
      if (ix>-1 && ret>-1) return [ret, ix];
      // search columns
      ix = iterateCols(findImminentLoss);
      if (ix>-1 && ret>-1) return [ix, ret];
      // search diagonals
      ix = iterateDiags(findImminentLoss);

      if (ix>-1 && ret>-1) return convertDiag(ix, ret);
      else return false;
    }

    // looks for a rol/col/diag that poses
    // an opportunity for a win or nearest win    
    function strategyBestOpportunity() {
      var ix = -1, y = -1, x = -1, d = -1;
      var count = 0, countY = 0, countX = 0, countD = 0;
      // look for the best opportunity, to be passed to iterateRows/Cols
      var seekOpportunity = function(a,b,c,i){
        var match = 0;
        var opponent = 0;
        var blank = 0;
        if (a==computer) match++; else if (a==player) opponent++; else if (a=='') blank++;
        if (b==computer) match++; else if (b==player) opponent++; else if (b=='') blank++;
        if (c==computer) match++; else if (c==player) opponent++; else if (c=='') blank++;
        if (match>0 && match<3 && opponent==0) {
          if (match > count) {
            count = match;
            ix = i;
          }
        }
      };
      // get the best row
      iterateRows(seekOpportunity);
      countY = count;
      y = ix; 
      ix = -1; count = 0;
      // get the best column
      iterateCols(seekOpportunity);
      countX = count;
      x = ix;
      ix = -1; count = 0;
      // get the best diagonal
      iterateDiags(seekOpportunity);
      countD = count;
      d = ix;
      ix = -1; count = 0;
      // determine the winning strategy (col, row or diag)
      var turnX = -1, turnY = -1;
      if (countX >= countY && countX >= countD) {
        turnX = x;
        turnY = findInArray([val(x,0), val(x,1), val(x,2)], 
            function(v) { if (v=='') return true; }
          );
      }
      else if (countY >= countX && countY >= countD) {
        turnY = y;
        turnX = findInArray([val(0,y), val(1,y), val(2,y)],
            function(v) { if (v=='') return true; }
          );
      }
      else if (countD >= countX && countD >= countY) {
        if (val(1,1)=='') {turnX = 1; turnY = 1;}
        else if (d == 0) {
          if (val(0,0)=='') {turnX = 0; turnY = 0;}
          else {turnX = 2; turnY = 2;}
        }
        else {
          if (val(2,0)=='') {turnX = 2; turnY = 0;}
          else {turnX = 0; turnY = 2;}
        }
      } 
      if (turnX==-1 || turnY==-1) return false;
      else return [turnX,turnY];
    }

    // computer takes its turn
    function computerTurn() {
      var strategies = [];
      if (option('avertLoss')) strategies.push(strategyAvertLoss);
      if (option('bestOpportunity')) strategies.push(strategyBestOpportunity);
      if (option('random')) strategies.push(strategyRandom);
      for (var i=0; i<strategies.length; i++) {
        var turn = strategies[i]();
        if (!turn) continue;
        val(turn[0], turn[1], computer);
        break;
      }
    }

    // highlight the square at x,y coords, a: [[x,y],..]  
    function highlightWinner(a) {
      for (var i=0; i<a.length; i++) {
        var coord = a[i];
        var x = coord[0], y = coord[1];
        var sel = "#s"+''+x+''+y;
        $(sel).addClass('winner');
      }
    }

    // finish the game
    function finish(p, highlight) {
      if (typeof p != 'undefined') { 
        $("#status").text(p + ' is the winner!');
        
      }
      else {
        $("#status").text('The game ended with a draw.');
      }
      turn = '';
      if (typeof highlight != 'undefined') {
        highlightWinner(highlight);
      }
      if (p=='X') xwins++;
      else if (p=='O') owins++;
      $("#xwins").text(xwins);
      $("#owins").text(owins);
    }

    // new game
    function newGame() {
      $("#board div").find("span").remove();
      $(".winner").removeClass("winner");
      turn = player;
      $("#status").empty();
    }

    /* ================= */
    /* UTILITY FUNCTIONS */

    // gets checkbox option true/false status
    function option(name) {
      return $("input[name='"+name+"']")[0].checked;
    }

    // returns the count of matching array members
    function matches(a, func) {
      var c = 0;
      for (var i=0; i<a.length; i++) {
        if (func(a[i])) c++;
      }
      return c;
    }

    // returns first index of matching value in array, or -1
    function findInArray(a, func) {
      for (var i=0; i<a.length; i++) {
        if (func(a[i])) return i;
      }
      return -1;
    }
    
        
    init();
    
  </script>
</head>
<body>
<h1>Tic Tac Toe</h1>
<div id="board">
  <div id="s00"></div>
  <div id="s10"></div>
  <div id="s20"></div>
  <div id="s01"></div>
  <div id="s11"></div>
  <div id="s21"></div>
  <div id="s02"></div>
  <div id="s12"></div>
  <div id="s22"></div>
</div>
<p id="status"></p>
<p>X wins: <span id="xwins">0</span><br/>
Y wins: <span id="owins">0</span></p>
<p><button onclick="newGame()">New Game</button></p>
<p>
  Strategies:<br/>
  <label><input type="checkbox" name="avertLoss" val="1" checked/> Avert Loss</label><br/>
  <label><input type="checkbox" name="bestOpportunity" val="1" checked/> Best Opportunity</label><br/>
  <label><input type="checkbox" name="random" value="1" checked/> Random</label><br/>
</p>
</body>
</html>
{% endhighlight %}
