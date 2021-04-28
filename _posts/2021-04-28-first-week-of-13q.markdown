---
layout: post
title: "First Week Working on 13Q"
date: "2021-04-28 07:56:52 -0500"
categories: "13Q Java Minecraft Plugin"
---

Over the past week, I have been working closely with my friend to start my developing journey on 13Q! So far, I did a custom explosion where it got the blocks being exploded, then generated sprites of them midair so you can see them falling down. That was just a warmup, and over the psat 3 days I have created working NPCs, where you can right and left click them to activate them, a working custom bossbar using a hidden wither, and a region manager that can tell you what "region" you're in on your scoreboard.

To elaborate on the NPCs, I had to work with making a "Fake Player". Basically, in Minecraft your client connects to the server using packets. In Spigot (which is the API I'm using to make this plugin), you can send your own packets. So, what happens when I send a create entity packet to the client... but don't tell the server to make the entity? Well this results in a "Fake Entity", which exists soley in the client. You can make a "Fake Player" by creating a Player Entity and sending it to the client. The tricky part was getting it to not show up in the "Tab List" (a list of online players that shows up when you hold the TAB key), *and* have a custom skin at the same time. I also added in a couple of listeners (that listen for game events such as one entity "interacting" with another) in order to facilitate doing things on left and rigth click.

How my custom bossbar works is that I have an invisible wither, its one of those "Fake Entities" so that the server doesn't have to deal with them, and it gets constantly added and removed so that you can't see the smoke particles that withers give off. It also constantly teleports the wither to you so that it stays in your field of view - if it isn't the bossbar won't show up. Adding text and changing the length of the bossbar is as simple as setting a custom name and health of the wither.

Finally, I have regions which are defined in a `.yml` file which is read into a map by a parser. Then, every second it checks if you are in one of those defining bounding boxes and if you are it will change your scoreboard value.

Anyway, thats what I've been doing over the past week and I'll probably post again next week with whatever I'm doing then.
