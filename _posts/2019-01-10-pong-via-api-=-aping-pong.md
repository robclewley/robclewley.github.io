---
layout: post
title: "Pong via API = APIng Pong"
comments: true
tags: ['AI','data science','pong','gaming','education','bot','serious game']
redirect_from: ""
permalink: /2019/01/10/pong-via-api-=-aping-pong
references:
---

{% assign ref=page.references %}

## "Data analyst" is not the runner-up prize to "data scientist"

Data science in real life is not much like it is portrayed in the courses that I see taught and in the media. I also think the domain of "data analysis" is underrated compared to "data science", which gets more attention from the hype of machine learning. [HBR recently echoed my thoughts on this nicely.](https://hbr.org/2018/12/what-great-data-analysts-do-and-why-every-organization-needs-them) Meanwhile, data science course offerings tend to follow the money/hype and often focus more on algorithms and the latest technologies compared to developing solid fundamental data skills that are useful every day. The success of the most sophisticated ML lives or dies by the quality of the input data and the relevance and meaningful design of the transformations and interpretations of data going in and out.

But, unlike a lot of data science competition scenarios, in real life you are often presented with highly novel situations and black box systems that you have to assess from scratch. As with most software development, technicians start by building their tools. Similarly, you'll need to spend some time observing game behavior, probably visualizing it, before iterating on your control algorithm -- like a real scientist would!

## APIng Pong is like a Bird Box Challenge for playing vintage pong

Games can be great at motivating the learning of new skills. The classic videogame ["pong"](https://en.wikipedia.org/wiki/Pong) may not present a great challenge to either human or AI players, but it's a perfectly fun and educational challenge when you're learning how to code and work with data. _Especially if you can't easily see what you're doing._

The [games I've been developing in the past couple of years](https://transient-dynamic.itch.io/dragonflyops) have been light on graphics and heavy on challenging the player to figure out what they need to get started. Recently, I've experimented with purely online games to focus on interactions between real players in real time.

For my own entertainment and practice in building performant python web applications with [Flask](http://flask.pocoo.org/), [Dash](https://dash.plot.ly), and [Heroku](http://heroku.com), I made a [real-time online pong game](https://aping-pong.herokuapp.com/) that's meant to be played by bots. You can compete with other people/bots online and put yourself on a leaderboard if you're good. This is no Kaggle competition, but it still sounds enriching and entertaining, right? You might want to recall your [trigonometry classes](https://www.youtube.com/results?search_query=trigonometry+for+gaming), though.

<center><img src="https://media.giphy.com/media/tJc0Sq1jilB8A/giphy.gif" alt="pong" width="250" height="100"></center>

So, here's the challenge. Pick single-player (squash) or two-player (pong). Declare your intention to play via the API. For squash, the game will then countdown to start, otherwise you'll wait until there's a second player to join in the lobby before the countdown.

When the game is ready, the dimensions of the court and paddle(s) are randomized, as is the initial ball trajectory. The game simulates _in real time_, so you make API calls to fetch the game's current position data and status and to move your paddle to keep the ball bouncing as long as you can. Game stats are recorded and the best are posted on the landing page's dashboard.

No downloads, sign up, or authentication are necessary to start. You just need access to the internet and a scripting language. **Hot tip:** If you keep note of a unique player ID that you are provided and reuse it, you can maintain an identity with the server and claim the positions on the leaderboard by associating a name tag to that ID code.

Here's a live update of the game stats from the server:
<center>
    <iframe width="550" height="40" src="https://aping-pong.herokuapp.com/stats_totals" frameborder="0">
    </iframe>
</center>

## Your pongbot can help with your resume

If you want to have fun while sharpening your critical thinking and problem-solving skills, try experimenting in the pong lab. For a leg up, I've even prepped an [impoverished Jupyter notebook visualization tool](https://github.com/robclewley/aping-pong-jupclient) to get you started.

Writing an effective pong bot and posting it on github could be a great resume-building project. Just sayin'.
