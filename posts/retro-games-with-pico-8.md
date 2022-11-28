---
title: "Create retro games with PICO-8"
subtitle: "8-bit games are a great way to learn and expand your portfolio."
date: "2020-12-19"
cr: ["https://www.artstation.com/shizuorin"]
---

After watching [High Score](https://www.youtube.com/watch?v=B4jopG1wX88) on Netflix, I was suddenly inspired to develop a retro-style game. I think it was a combination of the aesthetics and nostalgia that motivated me.

It turned out to be a pretty fun experience, and something I'd recommend as a weekend project to any developer. With tools like [PICO-8](https://www.lexaloffle.com/pico-8.php) (a NES-like virtual console), it's really easy to get started — even if you're a beginner to programming.

In this post, I'll be discussing why you should try your hand at retro-game development, and explain some of the technical limitations you'll be facing.

![images/pico-8 demo](/images/jelpi_demo.gif)

## Why you should build a retro game

For new programmers especially, a retro game project is:

- A great way to learn programming.
- An excellent addition to your portfolio.
- An effective way of prototyping new ideas (see [CELESTE](<https://en.wikipedia.org/wiki/Celeste_(video_game)>)).

The constraints of using "retro technology" creates a harsh environment to develop in.

But at the same time, you won't have to worry about learning a hundred different libraries, monetization funnels, or how to get it running on four different platforms.

Essentially, you get to focus purely on the technical and creative problems of the game itself.

## What is PICO-8?

From the [PICO-8](https://www.lexaloffle.com/pico-8.php) official website:

> PICO-8 is a [fantasy console](https://www.lexaloffle.com/pico-8.php?page=faq) for making, sharing and playing tiny games and other computer programs. It _feels_ like a regular console, but runs on Windows / Mac / Linux.

Basically, it's a program that pretends to be a console. It costs $15 to buy. You can use it to both play _and_ create your games. You can, of course, use an external editor (like [VSCode](https://code.visualstudio.com/)) with it as well.

You write code for it in [Lua](https://www.lua.org/) (if you haven't used that language before, don't worry — you'll pick it up in a day). The art and sound can be created directly in the console's editor, to be used in your game.

![images/pico-8-code-editor](/images/pico-8-code-editor.png)

Finally, you can even export them to HTML so your friends (and recruiters?) can check it out from their phone.

I'd say the learning curve from zero to [Pong](https://en.wikipedia.org/wiki/Pong) is just a matter of hours (or days, at most).

The best way to get started is to follow the [official manual](https://www.lexaloffle.com/pico-8.php?page=manual), or [watch a video](https://www.youtube.com/watch?v=K5RXMuH54iw).

## PICO-8's technical specs

You only have a palette of `16` colours, your canvas is `128` pixels wide and your whole program needs to fit within `65536` characters. There's almost no framework library to learn — aside from a handful of helper functions that would probably fit on a [napkin if written out](https://www.lexaloffle.com/bbs/files/16585/PICO-8_Cheat-Sheet_0-9-2.png).

Comparing it to actual retro console specs, it's somewhere between a [NES](https://en.wikipedia.org/wiki/Nintendo_Entertainment_System) and an [Atari](https://en.wikipedia.org/wiki/Atari).

|                    | PICO-8    | Atari     | NES       |
| ------------------ | --------- | --------- | --------- |
| **Year**           | 2015      | 1977      | 1985      |
| **Resolution**     | 128 x 128 | 160 x 192 | 256 x 240 |
| **Colors**         | 16        | 128       | 52        |
| **Cartridge Size** | 32 kB     | 4 kB      | 128 kB    |

## Code examples

Here's a couple of snippets of PICO-8 code to give you an idea of what development looks like.

### Game Loop

The first thing to note is that the game has a special function called `_update()` which is invoked at 30 FPS. This will probably be the main driving force behind your game logic. In this snippet, we create a variable `f`, which increases by `1` each update — effectively counting the number of frames since the game loaded.

```lua
-- this is a global variable
f = 0

-- this is a special function that pico-8 invokes 30 times per second.
function _update()
    f += 1
end
```

### Rendering a sprite

You can draw sprites (images) with the pixel art editor directly in the console. In PICO-8, you have 16 colors to choose from.

![images/pico-8-sprite-editor](/images/pico-8-sprite-editor.png)

Each sprite has an ID, which can then be used to render it on the screen at the x and y position you specify. Another special in-built function of PICO-8 is `_draw()`, which also executes at 30 FPS, but is guaranteed to execute after `_update()`.

```lua
x = 64
y = 64

function _draw()
  cls(0) -- clear the screen and set it to color 0 (black).
  spr(1, x, y) -- draw the sprite ID 1 at (x, y)
end
```

This will draw the above sprite (ID 1) at (64, 64) at the centre of the screen.

![images/pico-8-draw-sprite](/images/pico-8-draw-sprite.png)

### Capturing player input

PICO-8 detects user input via the `btn(k)` function, which returns true with the button with ID `k` is being pressed by the player. `k` ranges from 0 to 6 for a single player, and each number represents either the arrow keys, or two arbitrary game-play buttons like the `A` and `B` on a NES controller.

![images/nes_controller](/images/classic_nes_controller.jpg)

Adding this snippet to the rendering one above will allow us to move the character.

```lua
function _update()
  if btn(0) then x -= 2 end -- move left
  if btn(1) then x -+ 2 end -- move right
end
```

![images/pico-8-movement](/images/pico-8-movement.gif)

## Ideas to get started

So, if you like the sound of creating your own retro-game from scratch — either to pad your CV with an extra project, or just to learn and have fun, head over to [PICO-8](https://www.lexaloffle.com/pico-8.php) to get started! I recommend first just following the [manual](https://www.lexaloffle.com/pico-8.php?page=manual).

Once you've nailed the basics, here are some classic titles you could try to implement (and possibly extend):

- [Pong](https://en.wikipedia.org/wiki/Pong) (1972)
- [Space Invaders](https://en.wikipedia.org/wiki/Space_Invaders) (1978)
- [Pac-Man](https://en.wikipedia.org/wiki/Pac-Man) (1980)
- [Snake](<https://en.wikipedia.org/wiki/Snake_(video_game_genre)>) (1997)

Or if you're feeling more ambitious, you could even try to implement a [Mario](https://en.wikipedia.org/wiki/Super_Mario_Bros.) clone!
