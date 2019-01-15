---
layout: post
title:      "Hackifying the Tic-Tac-Toe With AI Project Tests"
date:       2019-01-15 12:59:50 +0000
permalink:  hackifying_the_tic-tac-toe_with_ai_project_tests
---


Ok, don't hate me, but I used the `#instance_variable_set` method in my latest Flatiron lab. 

Why is this so bad? For one, it's fairly common knowledge within the Ruby community that `#instance_variable_set` and `#instance_variable_get` are hacky ways of creating reader/writer methods for instance variables. As Avi, the Dean of Flatirion, put it (and I'm paraphrasing here): "It's syntactic vinegar. The method name itself ends with a verb, which shouldn't feel right." And he's right--it feels very wrong, but as far as I'm aware I didn't have any other choice. 

The reason I had to use it was because of how the tests are written for the lab I was working on. In Tic Tac Toe with AI, I needed to build the logic of how a computer player would play tic tac toe. I wanted the computer player to be able to utilize some of the methods defined in the Game class (e.g. `#won?,` `#over?`) as well as the `WIN_COMBINATIONS` constant, so that the computer would know which positions to take on the board to get it closer to victory, or which positions to take to prevent a loss. This meant that I would need to either pass the instance of the `Game` class to the `Players::Computer` instance or recode all of the logic from the `Game` class into `Players::Computer`. Since it felt even more wrong to copy and paste a bunch of methods from one class to another, and since I'm lazy, I chose the former.

Ideally, I would have syntactically sugar-ified this by passing in the instance of the `Game` class _to_ the computer player upon initialization of the player, however the tests wouldn't allow for that. I would get an unexpected number of arguments error for the tests relating to the `Players::Computer` instance, because instead of receiving one argument per its expectation (the player's "token", either an "X" or an "O"), it is now receiving two, hence the failure.

To work around this, I added a few lines of code to the `Game` class's `#initialize` method, which now looks like this:

```
  def initialize(player_1 = Players::Human.new("X"), player_2 = Players::Human.new("O"), board = Board.new)
    @player_1 = player_1
    @player_2 = player_2
    @board = board
    if player_1.instance_of? Players::Computer
      player_1.instance_variable_set(:@game, self)
    end
    if player_2.instance_of? Players::Computer
      player_2.instance_variable_set(:@game, self)
    end
  end
```
*Full repo here: https://github.com/tgray017/ttt-with-ai-project-v-000.*

I know, it looks awful. I'm a terrible human being for committing such a travesty. I may as well have just cut someone off in traffic while giving them the finger. I don't know how I'll be able to live with myself knowing that this exists in my GitHub repo for anyone to see, but I do know that I'm looking forward to writing my own tests so that I can just change the expectations of the test instead of hackifying the program to pass the test. 
