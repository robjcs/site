# 'Hand and Brain' Chess Bot
### *Experimenting with Lichess APIs*

![alt text](/writing/images/chessbot-endgame.png)

There are a couple variants of Chess in which more than two people can play. Most famous is [Bughouse](https://en.wikipedia.org/wiki/Bughouse_chess), where teams of players can place pieces from their teammates "graveyard" onto their board. 
Alternatively, there are other game variants such as Hand and Brain. Hand And Brain consists of teams of two players per side of the board: one player suggesting the kind of piece to move (the brain), and a player picking where to move the piece (the hand).

The rules of this game are that:
- The Hand can pick any piece of that type to move, if there are more than one on the board of their teams' color
- The Brain cannot tell the Hand where to move a piece, and cannot say specifically which one to move (again, if there are more than one on the board of their teams' color)

This is a fun gamemode to play with friends or in clubs, especially when there is a big strength differential between the Hand & Brain!

The goal of this project was to try to implement something similar in a Lichess bot.

Due to the constraints of what is possible logistically with a Lichess Bot, I settled on the following design:

>HandAndBrainBot will send you a message in the chat giving you a hint when it's your turn to move. HandAndBrainBot will only tell you the name / type of piece (e.g. pawn, rook, knight). It will not tell you where to move it, and which piece to move (if you have more than one of that type on the board).<br>HandAndBrainBot is powered by Stockfish, and plays moves at a low skill level, but suggests moves to you with the maximum strength of the engine!


Not *exactly* "Hand and Brain", but close enough! 

In a game, it looks like this:
![alt text](/writing/images/chessbot-hints.png)

## Lichess API
![alt text](/writing/images/chessbot-black.png)
Lichess has a very powerful and well-documented [API](https://lichess.org/api).

For this project I focused on accomplishing these goals with the API:
- Start a game (`/api/challenge/{challenge_id}/accept`)
- Wait for opponent's (challenger's) moves (`/api/bot/game/stream/{game_id}`)
- Read a position (`/api/bot/game/stream/{game_id}`+`Board().fen()`)
- Execute a move (`/api/bot/game/{game_id}/move/{move_uci}`)
- Send a Chat message (`POST /api/bot/game/{game_id}/chat`)

The API and the Python [Chess](https://python-chess.readthedocs.io/en/latest/) library made all of this very easy to implement.



## Stockfish

Integrating the engines were extremely easy. I created two instances of Stockfish:
```python
from stockfish import Stockfish
...
# Bot's engine -- weak
self.bot_stockfish = Stockfish()
self.bot_stockfish.set_skill_level(3) # or some other lower value

# Stronger engine for suggestions to opponent
suggestion_stockfish = Stockfish()
suggestion_stockfish.set_skill_level(15) # max
```

And then to use them to get the best move,
```python
engine.set_fen_position(fen)
best_move = engine.get_best_move()
```
I think, with newer engines becoming available that play more like humans (or, in more interesting ways at least, like [lc0 (Leela)](https://lczero.org)), there is some room here to use different engines and to have more of a "coaching" experience with this bot.

## Ollama (Bonus)

To make the bot feel a little more alive during games, and not just some Stockfish out-of-the-water, I added a class that listens for Chat events, and responds to messages (with some personality 😄).

In the `stream` API, I could listen for `event_type == 'chatLine'`. I was able to use these events, and the `text` object attached to them, as LLM input.

I am running `gemma3` with ollama on the bot host machine.

![chat screenshot](/writing/images/chessbot-noise.png)
![chat screenshot](/writing/images/chessbot-stalling.png)

-----


Hopefully this is a valuable way to play through some training games with Stockfish, especially in positions where it is hard to find a move to progress.

*Check out the code [here.](https://github.com/robjcs/libre-hand-and-brain)*