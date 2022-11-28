---
title: "Using AI to generate poetry"
subtitle: "I used GPT-3 to generate poetry and other creative content."
date: "2021-08-27"
---

I was recently granted access to the [OpenAI GPT-3 Beta](https://beta.openai.com/), a very powerful language model that can be used to do all sorts of magic like chat bot generation, article summary, and a [lot more cool stuff](https://beta.openai.com/examples).

But the application that really excited me was creative writing (stuff like SEO generation is cool and all, but a also a bit dry for my taste).

I had previously seen [AI Dungeon](https://play.aidungeon.io/) use GPT to create an interactive text adventure game (try it, it's fun!) and that got me curious about what else the AI was capable of — especially when elements of both _creativity_ and _structure_ were involved.

I decided to test it against poetry and story generation, and this is the resulting write-up of those experiments.

## What is GPT-3?

[GPT-3](https://arxiv.org/abs/2005.14165) is a language model developed by [OpenAI](https://beta.openai.com/). To the average developer (like me), it's basically a magical black-box that is extremely powerful at natural language use-cases.

It was trained on an estimated **45 TB** of text data (pretty much anything that is publicly available on the internet)!

We can literally ask it to do anything. For example, if I ask for steps to bake a cake it gives me:

```
1. Preheat oven to 350°F.
2. Grease and flour a 9-inch round cake pan.
3. Stir cake mix, oil, eggs, and water in a large bowl with a spoon until blended.
4. Pour batter into prepared pan.
```

Alternatively, I can provide it with an example of what I want. Such as:

```
[15 mins] Walk or jog lightlyy to warm up.
[10 mins] 3 sets of Squats
```

And it will understand that we're writing a work-out routine, and continue in that style — responding with something like this:

```
[10 mins] 3 sets of Push ups (use the bench/steps if you can't do regular ones)
[10 mins] 3 sets of Lunges
[10 mins] 3 sets of Sit ups
[10 mins] 3 sets of Tricep dips
[10 mins] 3 sets of
```

It can also do some pretty zany stuff, like translating from English to emoji.

```
I love pizza and beer.
😀❤️🍕🍺

I flew to Japan and ate some Sushi.
✈️🇯🇵😋 🍣

I was snowboarding in France.
🏂❄️🇫🇷
```

So as you can see, there's just endless amount of ways you can interact with the AI.

It's up to us to experiment with it and see how it can be useful for any particular problem. Let's give it a go!

## 💡 Method

Today, I want to see if this AI is able to understand and generate poetry. Why poetry? Because I think it's a relatively simple example, and has three distinct dimensions that we can easily assess it on:

- Tone
- Content
- Structure

OpenAI lets customers interact with GPT-3 via an API, so that's how I'll be using it as well.

I'll be providing both direct requests and examples (with increasing complexity), and let the AI come up with **20 responses** for each prompt.

Additionally, for my example prompts, I'll avoid using existing text—the AI may have already seen it in the training data and that might skew the results.

I'll then talk about the best samples in each batch and anything I find interesting about them.

Also I've pushed the [full set of results on GitHub](https://github.com/pixegami/gpt-3-poetry-results) if you wanted to check out the rest of the samples.

## 🧪 Results

### ✍ Short poem

As a baseline, I want to know how much the AI understands about the tone, structure and content of a 'poem.'

```
[Prompt]
Write a short poem.

[Sample AI Response]
Snow is falling,

silently,

the ground a white blanket.
Heroes dream in peace,
where they sleep.
```

It's a pretty good start! It's structured but it doesn't rhyme. The language and story also have a 'poetic' feel to it. Not bad.

Amongst the other 20 samples were also hilarious rubbish like this one 🤡 :

```
[Sample AI Response]
Life is like a beach
Where we take our coconut and whack it
or chew on it

There are also other beaches
Where we can drink some sand and ruminate
```

Not quite hitting the mark, but still pretty interesting.

### ✍ Short poem with theme

Next, I wanted to extend the prompt with some theme, just to see how it is able to assimilate that into its response.

I picked a theme that isn't typically associated with poetry: **Space Exploration** (also because I've been watching a lot of old-school sci-fi stuff recently).

```
[Prompt]
Write a short poem about space exploration.

[Sample AI Response]
I hope the stars align
For I dream and explore
Beyond what we see,
With the most amazing views
```

It's definitely found a way to put the space theme in there — not too unnaturally either, because it still links it to 'emotional' concepts like 'dream', 'hope' and 'explore.'

### ✍ Short poem with 2x themes

Now I know it can handle one theme, let's add another one to the mix and see what happens. Something _not_ commonly associated with either poetry nor space (we want to challenge the AI right?) — how about...

```
[Prompt]
Write a short poem about space exploration and parkour.

[Sample AI Response]
Taking the stars and reaching for the moon
Afraid to look down
She jumps, leaps, and lands
```

Not bad. Here's another sample.

```
[Sample AI Response]
Parkour through the cosmos
jump to the moon and land with a moonwalk
push off from the Earth, do a headspin
jump to Mars, make it look easy
```

I'm impressed that instead of just shoe-horning in the parkour theme, it's found words and phrases that have a natural connection with the space theme — like 'leap' and 'moonwalk.'

It was at this point that the [vector aspect](https://jalammar.github.io/illustrated-gpt2/) of the model started to really click for me.

It's as if it found the [cross-product](https://en.wikipedia.org/wiki/Cross_product) between the 'space' vector and the 'parkour' vector (this is a gross over-simplification), and applied that to the poem.

### ✍ Children's Story (Rhyming Verse)

My next task was to try something a bit more directed. This time I will give it a concrete example (with a tone, structure and story) to continue with.

To keep it (relatively) easy for the AI, I started with a rhyming verse, like something from a Dr Seuss book:

```
[Prompt]
It's a sunny day.
And Milky wants to play!
He looks for his ball,
but it's not in the hall.
```

> By the way Milky's the name of my dog, and this literally is the story of his life every morning.

Some of the results from this were super interesting. My favorite one is:

```
[Sample AI Response]
It's not on the stairs.
It's not in the yard.
Maybe it's in the attic,
somewhere really dark.
```

This one nails the tone — notice that it repeats the "It's not at \_\_\_" three times? That's just like what a children's book would do!

It also introduces some progression and drama into the story. Now our character has an idea where the ball is (the attic), but there is a sense of danger and risk in the way (somewhere really dark).

Structurally, "yard" and "dark" is also partial rhyme, but I think that was probably a fluke (out of the 20 samples, maybe only 2-3 managed to rhyme).

Still, I was really impressed with this result. Here's something else that I observed as well:

```
[Sample AI Response]
Then in through the window
stumbles a dog with a bark.
It's Billy, his brother.
```

_Dog!_ It figured out that Milky in this story is a dog, even though that was never explicitly mentioned in the prompt. It might be a small detail, but I was low-key amazed by that.

### ✍ Fantasy Fiction

Next let's test GPT-3's ability for creative story-telling as well. Here's a rhyming story with a bit of drama, written in a high-fantasy style:

```
[Prompt]
A mysterious curse sweeps the land,
and into townsfolk's hearts expand.
One broken link and night shall fall,
eternal sleep will enslave them all.

But there is hope
```

I ended the prompt with `But there is hope` because I was interested to see if the AI can progress (and negate!) the verse before it, rather than extend the example (which we already know it can do).

The results for this prompt were probably the most entertaining for me to read. My favorite:

```
[Sample AI Response]
a prophecy is spoken,
Which tells of one who will restore the sun.
She comes with a prince and shall break the spell,
and together save the land from hell.
```

So many things I love about this response.

- It introduces a _heroine_ to the story (+1 for gender diversity!)
- It writes a sensible continuation of the prompt.
- It even manages rhyming (between 'one/sun', and 'spell/hell').

A lot of the other samples were also quite interesting (check them out [here](https://github.com/pixegami/gpt-3-poetry-results)) and full of creative ideas.

### ✍ Hacker Rap

And finally just for a bit of fun, let's see what it can do with hacker rap 🤣 :

```
[Prompt]
When I'm tappin' I'm trapped in the screen
I escape when the codebase is clean
```

I loved it's work here as well. It did a good job picking up on both structure (rhyme) and content.

```
[Sample AI Response]
I can write code, I can code all night
You are my code and I am your light
We're in this together from dusk 'til dawn.
We got the skill to make things run
```

I also found this particular choice of words interesting:

```
The refactoring was a success, great!
And I will not commit the same crime
So I'm testin', testin', testin, testin' again
Just when I've seen enough, it happens again
```

"Commit the same crime" — does it know the significance of the word _commit_ for developers? 🤯

## Wrap Up

That's pretty much all I had to share this time around. GPT-3 has definitely impressed me with it's capabilities, and now I have a stronger understanding how this power can be wielded.

In a batch of 20 samples, not all of them are great—but the good ones are sometimes _really_ good.

If I wanted to use it in this way, I'd need a process that allows a human (me) to select and use the best samples.

I hope this was interesting! I also covered everything here in a video as well (which I geared towards less technical viewers) which you can check out [here](https://www.youtube.com/watch?v=r3zKcL0iGeA&t=47s).

Thanks for reading!
