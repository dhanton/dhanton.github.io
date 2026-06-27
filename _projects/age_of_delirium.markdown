---
date: 2026-06-01
title: Age of Delirium
subtitle: Programmer
image: /assets/img/projects/age_of_delirium/main.jpg
gif: /assets/img/projects/age_of_delirium/main.webp
---

*This is the project page for the game Age of Delirium. You may be looking for the [landing page](/age-of-delirium/).*
{: style="margin-bottom: 2em;"}

Age of Delirium is a multiplayer strategy game that is played with thousands of players on [Twitch](https://twitch.tv){:target="_blank" rel="noopener"}.

![Age of Delirium 1](/assets/img/projects/age_of_delirium/secondary.jpg){: .blog-img loading="lazy"}

<h1 class="toc-label">Table of Contents</h1>
* Rendered here
{:toc}

# Introduction

Even though I had the original idea a couple years ago, and had made some early tests and prototypes, development started on April 2025. The game was made from scratch using C++, and took a little over a year. All the game and backend code (around 40 thousand lines) was written by myself.

The initial idea for the game was to make a "[God Game](https://en.wikipedia.org/wiki/God_game){:target="_blank" rel="noopener"}" where the god would be a Twitch streamer, and every follower would be a viewer of the stream. I iterated over different concepts, and eventually landed on having multiple streamers compete against each other in short matches. Making something that lasted longer (like [/r/place](https://en.wikipedia.org/wiki/R/place){:target="_blank" rel="noopener"}) could've also been interesting, but would've required more investment from players and myself to support a long-running event, which would be hard to do for me since I had limited resources.

I also thought having entire communities of different streamers compete against each other would be very compeling. The game is all about players creating narratives; More interesting narratives would emerge between the different communities. Making a multiplayer game can be harder in some aspects, but I had a lot of experience with them. And it also allows your game to have substantially less content (as it can be mostly provided by the players and their interactions).

###### TODO Talk a little about about initial ideas for the game. How this game is different from others in that I need a somewhat polished version before actually testing it with players. How I had to simplify most of my ideas to go faster in development. How I still wasted a lot of time on dead ends cause that's how it goes sometimes.

# Technical Details
Below are some of the most interesting technical aspect and decisions I had to make over the course of development.

{::nomarkdown}
<div class="blog-section">
{:/}

#### Multiplayer tech
{: .blog-section-title}

<video class="blog-img" autoplay loop muted playsinline>
  <source src="{{ '/assets/img/projects/age_of_delirium/multiplayer_tech.webm' | relative_url }}" type="video/webm">
  Your browser does not support the video tag.
</video>

Choosing the right network model for a multiplayer game is very important (see [this article](https://mas-bandwidth.com/choosing-the-right-network-model-for-your-multiplayer-game/){:target="_blank" rel="noopener"}). In *Age of Delirium*, there are potentially thousands and thousands of completely independent players, all controlled individually by different viewers. Since viewer commands come from Twitch chat, it made sense to use an authoritative server that reads Twitch chat, parses commands, and bundles them into "frame inputs". The game processes these frame inputs one by one to advance the game state in a deterministic fashion. Therefore, as a network model, the server needs only to send the inputs, instead of game state changes, and all clients can simulate the exact same game. This saves bandwidth and allows for thousands and even millions of entities.

In deterministic multiplayer games, if for whatever reason two clients start to diverge, the consequences can be catastrophic. Even if the divergence is insignificant at first, both clients can start to deviate more and more as time passes, eventually reaching a completely different state. To prevent this, I employed the following techniques:
- The server is authoritative. Its game state is the ground truth and is always correct.
- A checksum of a *partial* game state is calculated after every frame. For efficiency, only the most important data in the game state is used (e.g., health is included, but not a unit attack counter); errors in internal data will "bubble up" to external data, reaching the checksum eventually. Clients send their checksums to the server and, if a divergence is detected, the client asks the server for the entire game state.
- Clients automatically speed up the simulation if they're falling behind the server. This can be caused by an unstable connection (dropped packets), or if the client is an older machine that is taking longer to calculate frames, or after receiving a full game state (which takes a couple seconds).

For Twitch integration, I used the [EventSub subscription API](https://dev.twitch.tv/docs/eventsub/){:target="_blank" rel="noopener"}. Since only the server needs to receive these events, and it always sits behind a public IP, the [Webook](https://dev.twitch.tv/docs/eventsub/handling-webhook-events/){:target="_blank" rel="noopener"} transport layer was the most appropriate.

When the match starts, the server subscribes to all streamer's chats. This causes API events to be sent to the webhook for every new chat message; the server processes these into valid commands which are passed over to the game engine. The server can also write in all chats as a [chatbot](https://dev.twitch.tv/docs/chat/){:target="_blank" rel="noopener"} to inform viewers of in-game events, command errors, and cooldowns.

{::nomarkdown}
</div>
{:/}

{::nomarkdown}
<div class="blog-section">
{:/}
#### Local simulation
{: .blog-section-title}

<video class="blog-img" autoplay loop muted playsinline>
  <source src="{{ '/assets/img/projects/age_of_delirium/local_simulation.webm' | relative_url }}" type="video/webm">
  Your browser does not support the video tag.
</video>

WIP

{::nomarkdown}
</div>
{:/}

{::nomarkdown}
<div class="blog-section">
{:/}
#### World generation
{: .blog-section-title}

<video class="blog-img" autoplay loop muted playsinline>
  <source src="{{ '/assets/img/projects/age_of_delirium/world_gen.webm' | relative_url }}" type="video/webm">
  Your browser does not support the video tag.
</video>

The game world is procedurally generated from an initial seed, using a fully deterministic algorithm. It currently only generates "island-type" worlds, which have a single central land of mass, as well as some simmple geographic features spread throughout (such as lakes, forests, and deserts). Once a world size is choosen, the algorithm performs the following steps:
- Two [simplex noise](https://en.wikipedia.org/wiki/Simplex_noise){:target="_blank" rel="noopener"} textures are generated: one for terrain elevation, and another for moisture. For every tile, both noise values are combined to choose the tile type (water, ground, mountain, desert, etc.).
- All mountain tiles are iterated and, if close enough to lava tiles, become volcanos (giving nearby houses access to the *forge*).
- Similarly, fresh water tiles that are part of the ocean surrounding the island become sea water (giving nearby houses access to the *sea fishery*).
- A [fill algorithm](https://en.wikipedia.org/wiki/Flood_fill){:target="_blank" rel="noopener"} is used to calculate which tiles belong to the mainland. This is important because there might be smaller islands around the main one that are inaccessible; some gameplay systems (e.g., path finding) depend on this and need a quick way of inspecting whether or not a certain tile is in the mainland.
- Trees and rocks are placed randomly using [cellular noise](https://en.wikipedia.org/wiki/Worley_noise){:target="_blank" rel="noopener"}. These are spread out in groups that are isolated from each other and have different number of elements.

As a final post-processing step, a "chunk score" is calculated for different sections of the world. This is a number that tries to predict how good a section is based on its natural resources and other metrics. This is used to ensure starting player locations are balanced and have similar conditions (for example, that no one starts too far away from trees or rocks). It is also used to ensure players are not too close to each other at the start.

{::nomarkdown}
</div>
{:/}

{::nomarkdown}
<div class="blog-section">
{:/}
#### Path finding
{: .blog-section-title}

WIP

{::nomarkdown}
</div>
{:/}

{::nomarkdown}
<div class="blog-section">
{:/}

#### Spatial sound
{: .blog-section-title}

WIP

{::nomarkdown}
</div>
{:/}

{::nomarkdown}
<div class="blog-section">
{:/}
#### Backend
{: .blog-section-title}

WIP

{::nomarkdown}
</div>
{:/}
