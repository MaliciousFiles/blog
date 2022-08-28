---
layout: post
title: "Just Enough Combat"
date: "2022-08-28 12:18:51 -0500"
categories: "JEC Minecraft"
---

<p class="paragraph" markdown="1"> Month, week, 2 months, same thing. Either way, I'm here again, at the prodding of a friend, hopefully getting this finished before school starts in force and bowls me over with all the homework and stress. Anyway this title is... completely unrelated to anything I've talked about previously. It's the working title for a mod I'm coding (sorry, no bad pun this time), a mod to overhaul Minecraft with a focus on the PvP (player versus player) system. This is a rather large project, aiming to expand Minecraft both vertically and horizontally, which I'll explain more later, but never fear: I haven't forgotten ReLife. Let's go!</p>
<div class="alert">
<b>Post-Write Note</b>
<br>
Since school started I haven't done much coding, for a variety of reasons. To be honest, most of those were excuses I told myself, but I think I needed a break anyway. I feel like I've got some of the drive back, and right now I'm thinking I'll work on ReLife, but I'm going to be talking about the mod in this post.
</div>
<br>
<p class="paragraph" markdown="1">Ok, so what do I mean by horizontally and vertically. Well, in game development, horizontality (not sure if that's a word) is how many options you have at a particular stage in the game. For example, lets say the stages are Wood gear, Stone gear, and Iron gear. The horizontality would be how many options you have at each stage. For Minecraft, this is essentially 1: there is a single build (path you can take with your gear) that is the best, and only really, build.</p>
<p class="paragraph" markdown="1">Verticality on the other hand is progression: how far you can progress in the game, and how long it takes to do that. For Minecraft, there is a very clearly defined end: literally a dimension called The End. You fight the *End*er dragon, and then you go to *End* cities and get all the endgame loot. That's all well and good, except for the fact that there are 0 skill checks along the way. Think of it as a river with no dams: it will flow at whatever speed it does. If you do a whole lot of griding (playing a lot with a specific goal in mind), you could get to the max build in a single day. People have spedrun beating the Ender Dragon (again, the final boss) in less than 25 minutes.</p>
<p class="paragraph" markdown="1">On the other hand, progression checks would be like the dams. You have to sit in the lake between then (the stages of the game) for a bit, until you can eventually flow past the dam into the next stage. Not the best analogy, but good enough I hope. Anyway, by expanding it vertically I don't mean making there be more stuff past the End, but I do mean to make it take longer and eliminate as much grinding as I can.</p>
<p class="paragraph" markdown="1">My final goal is to make PvP more interesting: that single build I was talking about earlier is not only the only option, but *insanely* good. Like, if you are fighting each other in maxed out gear, the fight could either never end, or last half an hour. Not good. So, I aim to make it so that there are different "classes", kind of like Terraria if you know that game. They aren't official, but I'm trying to have different types of armor and weapons that fall into the categories of, say, "Warrior", or "Tank", or "Ranger", etc. This would also encourage more collaboration between players (factions, alliances, betrayals, the whole thing), as having multiple people of different classes would compliment each other.</p>
<p class="paragraph" markdown="1">Ok, that's all well and good as an idea, but you're probably wondering (or maybe not, I don't know) how this is all going to work: what does this look like, concretely. Well fortunately for you, I've already worked on it a bit. There's a fair amount to talk about, so I'll try and skip over the similar bits, but here we go for real.</p>
<h2 class="heading">Armor</h2>
<p class="paragraph" markdown="1">Ok starting with armor. The way I'm laying this out is by A) stages and B) "class". The stages I'm looking at are: Stone, Iron, Diamond, Fused Quartz (new nether thing), and Netherite. So far, I've mainly worked on Stone and Iron, but I have a few ideas I'll talk about for later stages. But before any of that, let me tell you how armor works, in general. I'm trying to make it so that every single armor set has a set bonus: if you're wearing every piece of armor, it makes it all better. They can also have item bonuses (each item gives you something), but mostly it's set bonuses. Thus, the first thing I'm going to talk about, and arguably one of the hardest things I've done so hard, is how to display that on your armor bar.</p>
<h3 class="subheading">Armor Set Glint</h3>
<img src="/assets/images/posts/armor_icons.png">
<p class="paragraph" markdown="1">Above, you can see the 3 icons Minecraft uses for the armor bar by default. Each full armor piece represents 2 armor points, therefore there's a half full one to represent 1 point. You can have a max of 20 points, so the bar consists of 10 icons. For example, if you have 13 armor, there would be 6 full ones, followed by one half full one, and then 3 empty ones. Now, notably, resource packs *can* change those textures, and often do, but I'll talk about why that's important later. Just remember it for now.</p>
<p class="paragraph" markdown="1">Before I get into how I coded it, let me explain to you have I arrived at my current idea. Originally, I thought it would just have a golden border, but a few pixels outside of it. Same shape as the black border right now, but expanded. Unfortunately tha doesn't work, because in game they are rendered right next to each other, and making the already black border gold looked bad. Next I tried just making the entire inside gold, and that also didn't lookt too good. Then I tried doing something fancy and having a gradient so that the edges looked kind of gold, but that one was the most disgusting yet. Finally, as my brother and I were brainstorming (he helped a ton in this section), we had an epiphany: a glint. Make it so that it looks gold on the inside, but make it look better by having a glint shine over it. I'll get into how that's coded next, but here's a GIF of it working.</p>
<img src="/assets/images/posts/armor_set.gif">
<p class="paragraph" markdown="1">Ok, now for how I do this. You may have heard the term "shaders" thrown around before, but you likely don't know what vertex shaders or fragment shaders are. If so, you're like me before this project. Essentially, the way OpenGL works is that you have a vertex shader, which is passed in a vertex of the thing you're rendering (i.e. if you're rendering a quad, aka quadrangle aka rectangle, the 4 corners), and then it outputs whatever variables the fragment shader needs. It then interpolates those values between the vertexes so that it only has to run the vertex shader for each vertex, not each pixel.</p>
<p class="paragraph" markdown="1">The fragment shader, on the other hand, is run every pixel. It is given  in the values that the vertex shader outputs, or the interpolated values, and then outputs what color that pixel should be. So if I overwrite what shaders Minecraft uses when it renders the armor bar, if you have a full set, then it will use my custom shaders and I can do whatever I want to the color. In case you're wondering how to write shaders, it uses GLSL: Graphics Libary Shading Language. So, here's the pseudocode, and then we're done!</p>

```
if (not border and not dark) { // aka, if it is inside of a full armor bar
  goldify() // make the color more gold

  sheenTime; // game ticks between sheens
  targetSheenTime; // when this specific pixel should sheen (be fully white), based off of sheenTime
  distance; // how far it is from sheening

  if (distance >= 0 && distance <= 1) {
    color.rgb = mix(color.rgb, vec3(1), (1-distance) * 0.8); // make it sheen, using distance to calculate the alpha, so that it looks as if it fades in then out.
  }
}
```

<p class="paragraph" markdown="1">Now you may remember, as I asked, that resource packs can change textures. The way I do this, however, works for any texture, because I just build off of the existing texture: I don't load my own.
</p>
<h3 class="subheading">Armor Set Bonuses</h3>
<p class="paragraph" markdown="1">Ok, so how I'm gonna do this is explain how I made the set bonuses work in code, and then just give you a list of the armor sets and their bonuses. I don't feel like writing about each individual armor set, and you probably don't want to read that much, deal? Alright, so how I made set bonuses is that I made a callback happen every time you don or doff a piece of armor. Yes those are real words, look it up. So when your armor changes, each armor set checks how much armor you have, and then calls the callbacks that I provide for what do do with that info. Separately, it also uses that to raise or lower your movement speed based on the class of armor: I made it so that there's Light, Medium, and Heavy armor to add another thing I can tweak for balancing.</p>
<p class="paragraph" markdown="1">Alright, here we go, all the armors I have at the moment. Syntax is`Name: weight class | item bonus if one exists | set bonus`</p>

<ul class="dashed">
    <li>Stone</li>
        <ul class="dashed">
            <li><b>Leather:</b> medium | +10% defense</li>
            <li><b>Cloth:</b> light | +20% movement speed</li>
            <li><b>Carcass:</b> medium | chance to do extra damage | inflict hunger on hit</li>
            <li><b>Bone:</b> heavy | skeleton tracking range halved (like skeleton skull)</li>
        </ul>
    <li>Iron</li>
        <ul class="dashed">
            <li><b>Iron:</b> medium | +10% defense</li>
            <li><b>Chain Mail:</b> light | +3.5 damage whilst mounted</li>
            <li><b>Studded Leather:</b> medium | some armor toughness, and when hit, attacking weapon has a chance to take more damage</li>
            <li><b>Chitin:</b> heavy | weak to *Bane of Arthropods* | when you take damage, small chance to spawn an insect that fights for you (very low health and damage, small size e.g. endermites/silverfish)</li>
        </ul>
</ul>
<p class="paragraph" markdown="1">And here are the ideas for the later stages. Not fully fleshed out, just ideas:</p>
<ul class="dashed">
    <li>Wind-Up</li>
        <ul class="dashed">
            <li>As you move, it charges up. When full, if you attack you do more damage, or you can double jump, after either of which, it empties of charge.</li>
        </ul>
    <li>Earthen</li>
        <ul class="dashed">
            <li>When you cross a health threshold (i.e. 1 heart), you turn into a mud statue until you heal, during which you have very high defense, but cannot move.</li>
        </ul>
    <li>Camo (maybe)</li>
        <ul class="dashed">
            <li>Looks at colors behind you and renders that instead?</li>
        </ul>
    <li>Possessed</li>
        <ul class="dashed">
            <li>Has eyes on it taht follow nearby players (cosmetic)</li>
            <li>When attacked, there's a chance for the eyes to come out a bit, on a purely black square as a head, and then the attacker has the eyes appear on their screen and are cursed temporarily</li>
        </ul>
</ul>

<p class="paragraph" markdown="1">Wow that was a lot. Onto some new items!</p>
<h2 class="heading">Items</h2>
<p class="paragraph" markdown="1">I've only made a few items so far, so I'll explain each one in a little more detail, though not too much, don't worry. Firstly, a lance item. It gives you more attack range, and the faster you're going while mounted, the more damage you do. The hardest part of this was the extra attack range, because that's not something built into Minecraft, unfortunately. They have a ton of constants all over their code that dictate how far you can reach, so I had to modify those, while being careful to not let you mine that far either. I took some inspiration from a library I found on GitHub, but overall not too hard.</p>
<p class="paragraph" markdown="1">Second, a Totem of Vengeance. In the base game, there is something called a Totem of Undying, which will save you from death once, if you're holding it when you die. However, you cannot craft it, and the only way to get it is by defeating a fairly difficult mob. The totem of vengeance is a twist on that: it will be easier to get, but it doesn't exactly save you. When you die, you have a period of time where you cannot take damage, but then after a bit you die anyway. Essentially it gives you a chance to get, well, vengeance. I also made a cool shader for this (this time a little different from the armor shader: that one was meant for rendering icons, this one is for your entire screen), which tints it red among other things.</p>
<p class="paragraph" markdown="1">Finally, the shuriken. This one was by far the most complicated. The idea, is that you right click with it, and it will throw a shuriken: an alternative ranged weapon to bows, crossbows, and tridents. I don't feel like going into everything I did for this, but I'll give a brief overview for each thing. First, I had to render it as an entity when you throw it. Luckily there is already a built-in item renderer, so I just do that, modify the rotation so it looks like it is spinning, and bam. Renderered. I also had to make a custom renderer so that if a shuriken hits a player, it doesn't look like the player has an arrow stuck in their body, but rather, a shuriken. Again, a simple item renderer.</p>
<p class="paragraph" markdown="1">Then, I had to make it so they could support enchantments. Because why not. The enchantments Shurikens can support are: Piercing, Loyalty, Multishot, Sharpness, Bane of Arthropods, and Smite. Piercing essentially means that they won't get destroyed after hitting one entity, not too hard to code; just copied arrow. Loyalty was a little bit harder, but similarly, I adapted the trident code. Again with multishot, I did it basically the same way that crossbows handle it, just had to make it work with my code. And finally the various damaging enchantments were the easiest. Overall not very difficult, just took some time and fanagling to get each one to work just right. That's it for items, and the final section is.... entities!</p>
<h2 class="heading">Entities</h2>
<p class="paragraph" markdown="1">The only entity I've added so far is stone golem, but boy have I spent a while on that. The idea is that it makes caves more difficult, making one of those skill checks between Stone age and Iron age. It has two states: dormant and awake. Whilst dormant, it simply looks like a formation of stone blocks, but if you get close enough, it will wake up and become a deadly mob. To be honest, the hardest part of this was figuring out Minecraft's animation system. It was rather confusing, but thanks to my brother looking into the code, and a wonderful program called Blockbench, we eventually figured it out.</p>
<p class="paragraph" markdown="1">The second hardest part was getting it to render as stone. Similar to the armor bar icons, resource packs can change the stone texture, so I couldn't just get it to look like default stone: else, you could see it easily with a resource pack. Thankfully, Minecraft has a built-in block renderer just like the item renderer. In the vanilla game, it is used for falling blocks, but I can appropriate it for my own purposes. I make it render the stone blocks in all the right places, and then I remove the inside sides, so as to make it look like one continuous texture. It took us forever to settle on that idea, though, and having another person to brainstorm and argue with over ideas was incredibly helpful.</p>
<h2 class="heading">Wrapup</h2>
<p class="paragraph" markdown="1">Well this was decently long, though not nearly like last time, so thanks for reading to the end. I'm not entirely sure when my next post will be, partly because I don't know how much I'll have time to code, and partly because I don't know how much I'll have time to write. Either way, I hope you enjoyed! If you did, consider subscribing below. Or don't. Up to you. Either way, see you all next time!</p>
