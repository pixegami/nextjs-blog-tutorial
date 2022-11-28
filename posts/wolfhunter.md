---
title: "Wolfhunter: A PICO-8 RPG"
subtitle: "Implementing Pokemon-like combat mechanics in PICO-8."
date: "2020-12-22"
---

In a [previous post](./retro-games-with-pico-8) I talked about how making retro 8-bit games is a great way for fledgling developers to learn new things whilst expanding their portfolio.

I recently gave it a go myself, making a small RPG combat game called "Wolfhunter".

I spent two weekends on the project, which—despite my prior experience in games development—still managed to surprise me with some interesting challenges.

[Source (GitHib)](https://github.com/pixegami/wolfhunter) | [Play the Game! (Web, Mobile)](https://pixegami.github.io/wolfhunter/)

![images/wolfhunter_title](/images/wolfhunter_title.png)

## Concept

Wolfhunter is an 8-bit RPG where you play as a hunter facing off against a werewolf, in a Pokemon-like combat sequence.

You have magic spells and single-use items at your disposal, but the wolf is very powerful. The only way to kill it is to weaken it with your attacks, then finish it off with a silver-knife (which you only have one shot at).

It is made with [PICO-8](https://www.lexaloffle.com/pico-8.php), and can be played directly in your browser [here](https://pixegami.github.io/wolfhunter/).

> The motif and design of the game was based on the word "Moonshot", which was the theme of [GitHub's GameOff 2020 game jam](https://github.blog/2020-11-01-github-game-off-2020-theme-announcement/).

## Gameplay

![images/wolfhunter_gameplay](/images/wolfhunter_gameplay.gif)

I set out to make a game loop that felt similar to the combat in [Pokemon Red/Blue](https://en.wikipedia.org/wiki/Pok%C3%A9mon_Red_and_Blue), which was the most popular title on the [original Nintendo Gameboy](https://en.wikipedia.org/wiki/Game_Boy).

The characters stand opposite each other, and take turns to fight. There's a bunch of other mechanics too, like strengths/weaknesses, status effects, and switching characters. Here's a scene from it:

![images/pokemon_red_fight_scene](/images/pokemon_fight_scene.png)

For Wolfhunter, I copied the scene layout, the turn based fighting, and the status effects. I didn't have a chance to implement other aspects of Pokemon — so instead I doubled down on the mechanics I did implement.

### Core Mechanics

The units take turns to fight. The game ends when one of the character's HP is reduced to 0.

Each turn, the player has two basic abilities that can be used for free.

| Ability | Effect                         |
| ------- | ------------------------------ |
| Attack  | Deal 15 damage.                |
| Defend  | Blocks 15 damage for one turn. |

### Magic

The basic abilities were good to establish some baseline gameplay, but not enough to be interesting. I also added '"magic", which costs **1 mana** to use (you start with **5 mana**, and cannot cast spells if you run out).

| Spell    | Effect                                                                     |
| -------- | -------------------------------------------------------------------------- |
| Fireball | Deals 40 damage. Ignores defence.                                          |
| Spark    | Deals 12 damage and **blinds** the enemy, causing the next attack to miss. |
| Heal     | Recovers 35 HP and clears **bleed** effects.                               |

Magic is more powerful than just attacking or defending, but are in limited supply, and designed to _interact/react_ against enemy abilities.

### Items

Items are even more powerful than magic, but only have **1 use** per fight.

| Item         | Effect                                                    |
| ------------ | --------------------------------------------------------- |
| Crossbow     | Deals 25% of the enemy's HP as damage.                    |
| Elixir       | Recover 5 mana.                                           |
| Silver Knife | Deals 5 damage, but is the only way to kill the werewolf. |

### Enemy

Finally, to tie the gameplay together, the enemy must interact well with the player's abilities, and provide some kind of strategic challenge.

| Ability       | Effect                                                                               |
| ------------- | ------------------------------------------------------------------------------------ |
| Ravage        | Deals 8 damage and causes **bleeding**.                                              |
| Strong Defend | Blocks 20 damage for one turn.                                                       |
| Dark Flight   | Skips a turn, then deals 64 damage.                                                  |
| Raging Strike | Deals 18 damage but becomes **vulnerable** after, taking double damage for one turn. |

These enemy is strong, but the player has ways to mitigate each of its threats. Pretty much the point of the entire game here is figuring out what move to use next.

## Event Sequence

From an implementation standpoint, the most important part of the game is the "event sequence". This system will control how the game flows, and sets the framework for everything else to follow.

I define an "event" as **something atomic that happens in the game**. It usually displays some description to the player, and requires the player to acknowledge. Examples:

- A character uses an ability.
- Damage is taken.
- Character is defeated.
- Turns switched.
- Status effect is applied.

This is pretty much the back-bone of a turn-based system. Everything in the game—from the menu selection, to mana consumption, to damage/block resolution—can be modelled with events.

And in this game, events are always sequential and happening in order. But new events can be added in the middle of the sequence—for example if an attack is blocked, we need a "block" event to resolve.

![images/linked_list](/images/linked_list.jpg)

This is pretty much a [linked list](https://en.wikipedia.org/wiki/Linked_list), and that's how I implemented it. The events are the nodes. Additionally, when each event resolves, it can modify the nodes directly ahead of it, or at the tail of the list.

### Sequence

This is pretty much a linked list implementation. We start it off with an `new_info_event` (display some text to the player) that says "it's your turn to move."

The `head` of the sequence is the event we are currently processing in the game.

```lua
function new_sequence()

  local first_event = new_info_event("it's your turn to move!")
  local sequence = {
    head = first_event,
    tail = first_event
  }

  -- move sequence cursor to the next event.
  sequence.next = function(this)
    this.head = this.head.next
  end

  -- add an event to the end of the sequence.
  sequence.add = function(this, e)
    this.tail.next = e
    this.tail = e
  end

  -- add an event after the head of the sequence.
  sequence.insert = function(this, e)
    e:get_tail().next = this.head.next
    this.head.next = e
  end

  return sequence
end
```

### Damage Event

Here's an example of an event that causes damage to a unit (character). The `action` is an anonymous function that executes _once_, when the event is processed.

This includes playing sound effects, animations, and actually modifying the HP values. In this case, if at run-time the damage causes a unit to die, then a `new_end_combat_event` is added to the sequence, along with a `new_info_event` to display some informative text to the player.

```lua
function new_damage_event(unit, value)

  local desc = unit.name.." takes "..value.." damage!"
  local dmg_event = new_event("damage", desc, true)

  dmg_event.action = function(this)
    unit.hp -= value
    unit:animate(new_hit_animation())
    sfx(3)
    if unit.hp <= 0 then
      unit.hp = 0
      sequence:insert(new_end_combat_event(unit.name))
      sequence:insert(new_info_event("the fight has ended!"))
    end
  end

  return dmg_event
end
```

### Sequence Control

In the `_update()` function of the game, we process the `head` event and show the appropriate information. Special event types, like `menu` will control what UI is shown.

Some events (`auto`) will be processed immediately—but most will require interaction from the player to progress.

```lua
-- update the menu if we are showing one.
if event.type == "menu" then combat_menu:update() end
if event.type == "magic" then magic_menu:update() end
if event.type == "items" then items_menu:update() end

-- each time we press x, the sequence progresses.
if btnp(5) or event.type == "auto" then sequence:next() end
if btnp(5) and (event.type == "info" or event.type == "damage") then sfx(1) end
```

## Items and Magic

I think items and magic are an interesting one. Instead of being their own event types, I think it was a good chance to apply the [decorator pattern](https://en.wikipedia.org/wiki/Decorator_pattern).

That's because items and magic can be thought of as regular abilities—wrapped with some extra conditions!

Namely—we want to wrap the concept of **mana consumption** onto magic, and **single-use** onto items. Aside from those things, they still do anything else an ability can do. So what I'm after here, is a function that can take in an **ability** (an arbitrary event), and turn it into a magic spell, or an item.

Consider we have this 'heal' event. As is, this will recover 35 HP, and doesn't have any other cost or conditions associated with it.

```lua
function new_heal_event(name, unit, value)
 -- restore 35 hp!
end
```

Now, I have a decorator function, which accepts an `event`, and adds mana usage to it.

```lua
-- wrap an event as a spell, so it costs mana to use.
function as_spell(unit, event)
  local spell_event = new_event("auto", "", true)
  spell_event.action = function(this)
    if unit.mana > 0 then
      unit.mana -= 1
      sequence:insert(event)
    else
      sequence:insert(new_event("menu"))
      sequence:insert(new_info_event("you don't have enough mana to cast this spell."))
    end
  end
  return spell_event
end
```

This will check if I have **enough mana** to use this event—and either cast the spell and deplete the mana, or cancel the event and inform me that I don't have enough mana.

Now I can wrap my `heal` event with `as_spell` and turn it into a magic spell!

```lua
as_spell(unit, new_heal_event(event_id, unit, 35))
```

## Animation

[PICO-8](https://www.lexaloffle.com/pico-8.php) doesn't really give us an animation framework. We can draw sprites at arbitrary locations and change some colour palettes, but aside from that—we're pretty much on our own.

Thankfully, in a turn-based RPG, the animations aren't very complex. I have 'units' on two corners of the screen, and they flicker and bounce around a bit when hit.

To achieve, this, assume we have a `unit` object with some `x` and `y` position—its absolute position on the screen.

```lua
function draw_unit(unit)
  local x = 128 - spr_size
  local y = 0

  -- draw the unit
  spr(1, x, y)
end
```

I can move the unit by changing `x` and `y`, but what if I cancel an animation half way through? I don't want the animation state to be 'baked' into the unit's actual position, because it will be harder for me to reset it.

My solution was to create an 'animation' object, which keeps an independent frame count `n` and transformation `x, y`, which can be then _applied_ to the unit at render time.

```lua
function new_animation()
  local animation = {
    name = "default",
    n = 0,
    frames_left = 15,
    x = 0,
    y = 0,
    color = 0,
  }
}
```

Now each update, we can also `update()` the animation and apply its transform to the unit. If the animation runs out of frames, it will disappear and things will be back to normal.

```lua
-- apply unit animation
if unit.animation then
    unit.animation:update(x, y)
    anim_spr_x = x + unit.animation.x
    anim_spr_y = y + unit.animation.y
    if unit.animation:has_ended() then unit.animation = nil end
end

-- draw the unit
spr(1, anim_spr_x, anim_spr_y)
```

However, we can also just cancel the animation at any time we want, no matter what frame it is on. This gave me a lot of modular control over the animation, but also prevented any part of the animation system having an impact on the logical game state.

## Tools

Here's some of the main tools/software I used for the job.

- [PICO-8](https://www.lexaloffle.com/pico-8.php): Virtual console, and framework to develop and run the game.
- [VSCode](https://code.visualstudio.com/): Code editor.
- [Pixaki](https://rizer.co/): iPad pixel art editor to draw and touch up the sprites (which to be honest, I mostly traced from Pokemon).

## Summary

That pretty much sums up my experience of implementing RPG combat in [PICO-8](https://www.lexaloffle.com/pico-8.php). Even with the lo-fi, 80's era constraints, I found that the core problems—like game design and event modelling—are still timeless.

With a bit of work, I think it'd be possible to implement a more Pokemon-like RPG in PICO-8, but probably on a smaller scale, because of the memory and size limits (I was almost at 30% capacity with just this).

At some point though, dealing with over 1,000 lines of code in a single `lua` run-time becomes unwieldy. So whilst something like [Unity](https://unity.com/) is better suited for serious projects, PICO-8 is still a great way to learn and prototype ideas.

> Wolfhunter's [source code](https://github.com/pixegami/wolfhunter) is public, and you can play the game directly in the browser (both web and mobile) [here](https://pixegami.github.io/wolfhunter/).
