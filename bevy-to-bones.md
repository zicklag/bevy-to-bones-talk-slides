<!--
Major points to cover:

- Why we we felt the need to move away from Bevy to our own solution.
- How we moved piece-by-piece away from Bevy to Bones.
- What the technology ended up, and how it works.
 -->

<style>
.reveal h4 {
  font-size: 1.2em;
}
code {
    font-size: 0.8em;
    color: #C98BFF;
}
code::-webkit-scrollbar {
    width: 10px;
}

code::-webkit-scrollbar-track {
    border-radius: 8px;
    background-color: #e7e7e7;
    border: 1px solid #cacaca;
}

code::-webkit-scrollbar-thumb {
    border-radius: 8px;
    background-color: #D657DC;
}
</style>

### From Bevy to Bones

<!-- .slide: data-timing="8" -->

**"Accidentally" Making a Game Engine**

Notes:

- Hey everyone, I'm @zicklag.
- I've been working on Rust game dev on and off for the last 5 years, and I started using Bevy when it
  was publicly announced 4 years ago.
- I'm self-taught and have been learning everything from Rust to graphics programming just by trying
  things out, and doing whatever I needed to get stuff done.
- I'm excited to be here, and today I'm going to talk about the journey that me and the Jumpy community
  took as we "accidentally" made our own game engine.

---

<!-- .slide: data-auto-animate data-timing="8" -->

Isn't This a _Bevy_ Meetup?

Notes:

- A question you might be asking at this point is, "isn't this a _Bevy_ meetup?"
- Of course it is, but as you'll see,
- our engine, Bones, has learned so much from Bevy that it would have been completely impossible to
  make Bones without it.
- Bones is kind of like Bevy's little brother.

---

## What is Jumpy?

<!-- .slide: data-timing="10" data-background-video="jumpy-presentation.webm" data-background-video-muted="true" data-background-opacity="0.50" data-background-video-loop="true" -->

2D

Multiplayer

Shooter Game

<span class="fragment">✨ With Modding ✨</span>

Notes:

- That brings us to "What is Jumpy?"
- Jumpy is a
- ️2D
- Multiplayer
- Shooter Game
- that's all about great local play, fast reflexes, and importantly ⏭️
- ✨ _modding_ ✨

---

### Rewriting Jumpy with Bevy

Notes:

- 2 years ago, when I took over as the lead developer for Jumpy, it was already using Macroquad for
  it's engine.
- I had more experience with Bevy, though, and we were already using Bevy for another game, and we
  could share a lot of the knowledge and resources between them if we used Bevy for both.
- So we decided the best option was just to rewrite Jumpy to run on Bevy.
- I was pretty excited.

---

### Start Working! 👨‍🔧

<!-- .slide: data-timing="20" -->

<div style="display: flex; justify-content: center; align-items: center;">
    <img src="./fishfolk-logo.png" style="width: 10" />
    <div style="font-size: 4em; padding-left: .3em; padding-right: .3em;" >+</div>
    <img src="./bevy_logo_dark.svg" style="width: 40%" />
</div>

<div style="color: #C6522C;" class="poetsen-one-regular">
    <div class="fragment" style="position: absolute; top: -1em; left: -1em; transform: rotate(-25deg)">Maps Loading</div>
    <div class="fragment" style="position: absolute; top: -1em; right: -1em; transform: rotate(25deg)">Physics Ported</div>
    <div class="fragment" style="position: absolute; bottom: -2em; left: 4em; transform: rotate(0deg)">Main Menu</div>
    <div class="fragment" style="position: absolute; bottom: -2em; right: 4em; transform: rotate(-0deg)">Map Editor</div>
</div>

Notes:

- I started working right away, and things went really well.
- We got
- ⏭️ maps loading,
- ⏭️ physics ported,
- the ⏭️ main menu,
- and even had the start to an in-game ⏭️ map editor, all in pretty good time.
- Then came the real challenge, and something that I had never done before...

---

<!-- .slide: data-auto-animate data-timing="2" data-background-image="networking-cc0-upscaled.png" data-background-opacity="0.85" data-background-transition="zoom" -->

<h2><span style="background-color: hsla(0, 0%, 8%, 0.95); padding: 0.2em 0.3em; border-radius: 0.2em;">Networking</span></h3>

Notes:

- Networking...

---

<!-- .slide: data-auto-animate data-timing="18" -->

<h2 style="position: relative; top: -1em">Networking</h2>
<div style="position: relative; top: -2em; font-size: 0.9em">

<h3 style="color: #22D491;">Naïve Client-Server</h4>

- Horrible skipping
- Mixes gameplay & networking code

</div>

Notes:

- I started off trying the simplest thing I could think of: a headless bevy server that synchronizes
  transforms between clients.
- While this kind of worked, the skipping was horrible and it was unplayable.
- I also had to add a bunch of networking code all throughout the gameplay code, which was annoying
  and complicated.

---

<!-- .slide: data-timing="45" -->

<h2 style="position: relative; top: -1em">Networking</h2>
<div style="position: relative; top: -2em; font-size: 0.9em">

<h3 style="color: #22D491;">Client-Server With Latency Compensation</h4>

- Requires sync and fast-forward
</div>

Notes:

- Then I did a bunch of thinking and research realized that to get smoother gampelay you have to
  compensate for the network latency.
- That means syncing with the server and then fast-forwarding the game in the background to make up
  for the network delay.
- If I was going to go through all that trouble, I thought it was worth trying an other option first.

<!-- - After doing more research I found out that, in order to get smoother network play, I have to take into account the network latency between the client and the server.
- This means that when the client gets an update from the server, the game must realize that the update actually came from, say 100ms ago, because of the network ping, and
  then it has to actually play the game in fast-forward, behind the scenes, to catch up to the present time.
- This was doable, for sure, but if I was going to bother with syncing and fast-forwarding, this was going to be harder than I thought, and it was worth considering another networking model first. -->

---

<!-- .slide: data-timing="36" -->

<h2 style="position: relative; top: -1em">Networking</h2>
<div style="position: relative; top: -2em; font-size: 0.9em">

<h3 style="color: #22D491;">Peer-to-Peer Rollback</h4>

- No servers!
- Simple gameplay code!
- Requires <span style="color: #C6522C">Determinism</span> and <span style="color: #C6522C">Snapshot / Restore</span>

Notes:

- I had heard of peer-to-peer rollback networking, which was really cool because it didn't require a
  dedicated server.
- Clients could connect directly to each-other instead.
- On top of this, the gameplay code could can stay the same during network and offline play, instead
  of having to mix network messages in with normal gameplay.
- The caveat was that the gameplay had to be deterministic, and you had to be able to
  create and restore snapshots of it.
- Jumpy is a simple game, so that didn't sound like it'd be too hard, and
- Lucky for us, Bevy already had a plugin for peer-to-peer rollback networking.

<!-- - Peer-to-peer rollback networking does a similar rewind and fast-forward thing, but it doesn't require running a headless Bevy instance on a server.
- That's really handy for making network gaming inexpensive or even free, which is a big deal for our idea of "Everlasting games".
- The caveat is that it requires the game to be deterministic, and to support snapshot and restore.
- These features are harder to get than they might seem, but there's a great payoff.
- The peer-to-peer model gets you essentially the most fair and best feeling networking out of any other solution I've found.
- It also had one other big advantage:
- The networking code could stay completely separate from the gameplay code.
- This would make it **way** easier for us to ensure that network games worked just as well as local games.
- Lucky for us, Bevy already had peer-to-peer networking plugin, called Bevy GGRS. -->
</div>

---

<!-- .slide: data-auto-animate data-timing="64" -->

### Bevy GGRS

- <span style="color: #22D491;">Handles all the networking for you!</span>
- You have to be <em style="color: #C6522C">very careful</em> not to break determinism or snapshot / restore.

Notes:

- So we started testing out the Bevy GGRS plugin and we were really happy with how much it
  simplified the game code.
- But we started to have some serious problems keeping the game deterministic.
- Bevy is designed for maximum performance and multithreading, and that is often in conflict with determinism.
- Doing normal things like iterating over a query turned into a problem, because now we had to
  collect the query results and sort them to make sure that the iteration order was deterministic.
- Many things in Bevy are impossible to snapshot and restore, so the more we did, the more gotchas we found.
- We couldn't use `GlobalTransform`s or `Local` system params, and we couldn't use events.
- These were all things that were super easy to miss, and now with every change we made, we risked
  unintentionally breaking our determinism.
- This was really bad for new contributors, and just for overall development peace-of-mind.

<!-- - As we continued to use Bevy GGRS, we ran into quite a few gotchas.
- Bevy's design is very focused on performance, and that, in most cases, comes at the expense of determinism.
- On top of that, very little in Bevy lends itself to snapshot and restore.
- This meant that there were a lot of Bevy features we either couldn't use, or had to be extra careful with.
- Things like events, `Local` system params, and `GlobalTransforms` couldn't be used.
- Almost all queries had to be collected into a vector and sorted before we could safely iterate over them, to avoid the non-deterministic iteration order.
- Finally, you have to attach rollback IDs to entities you want to sync,
- you have to be careful with how you use hierarchies,
- and storing `Entity`s in components requires implementing an extra `MapEntities` trait.
- All of these things were really easy to miss, and it was now a looming threat that with every change we made to the game, we could accidentally break determinism.
- It felt like a new kind of "undefined behavior" to avoid, and in that respect, all the code was now `unsafe`.
- These issues weren't GGRS's fault, there were just no good ways get deterministic snapshot restore in Bevy. -->

---

<!-- .slide: data-timing="30" -->

### Bones ECS

<h3 style="color: #22D491; font-size: 1.1em">A Deterministic Box</h4>

A _tiny_ ECS

With a `World` you can `.clone()`

<span class="fragment" style="position: absolute; right: 0; bottom: -4em; font-size: 0.75em; transform: rotate(-5deg)">✨ and scripting...? ✨</span>

Notes:

- So I did a lot of thinking about what it would take to put the Jumpy's core gameplay
  logic in it's own deterministic "Box", so to speak.
- We could make our own tiny ECS,
- and make the whole world `.clone()`-able so that it would be trivial to snapshot and restore.
- I started by forking Planck, the simplest Rust ECS I could find.
- It used plain vectors for storage and bitset operations for queries.
- We forked it, called it Bones ECS, ⏭️ and made just a couple tweaks to make it more suitable for
  scripting.
- But scripting would come later.
<!-- - The one thing I wanted to change was to make it capable of storing runtime defined types, a
  crucial feature for our future modding and scripting goals.
- See, the hope was that on top of determinism, we might be able to unlock some scripting potential.
- The Bevy ECS, being as complicated as it is, was a lot harder to add scripting to.
- Our new Bones ECS on the other hand, focused heavily on simplicity, so the hope was that it would
  be a lot easier to add scripting features to later on. -->

<!-- - We started off by forking Planck ECS, the simplest Rust ECS I could find.
- Planck used normal Rust vectors for storage, and used bitsets operations to do queries over components.
- The only one major modification I wanted to make to it was to allow us to store runtime-defined types.
- This was a step back in the direction of scripting.
- Because Bones ECS was going to stay small and simple, it would be much easier to implement scripting for.
- While there was promising progress with `bevy_mod_js_scripting` it was also very complicated, and there were still big questions
  about how to make certain things work in scripts, especially when it came to interacting with assets.
- By doing our own scriptable ECS for the gameplay, it would be easier to get the modding experience we were looking for. -->

---

<!-- .slide: data-timing="64" -->

### Bevy + Bones

<div>
  <img src="./bones-in-bevy.excalidraw.png" style="max-width: 90%" />
</div>

Notes:

- Now we started migrating the Jumpy gameplay code from the Bevy ECS to the Bones ECS.
- This was a big-ish rewrite, and there was lot more boilerplate than I had imagined.
- But finally we got the game working again, and now we had a very interesting architecture.
- When the game starts up, shows the menu, and it's just a normal Bevy game,
- but when you start a match, it creates a new Bones ECS world, and starts sending the controller
  inputs into it.
- The Jumpy gameplay code all runs inside the bones ECS, and it creates entities for the sprites,
  camera, and tilemap, etc.
- On each frame, a Bevy system will then read all of the sprites out of the Bones world, and render
  them in Bevy.
- So as the player, it looks like nothing has changed.
- On the inside, though, GGRS is now able to make a complete snapshot of the Bones World whenever it
  needs to, and the all the gameplay inside is completely deterministic.
- It was working, and we were excited!

<!-- - After finishing the first draft of the Bones ECS, we started porting the core Jumpy gameplay to use it instead of the Bevy ECS.
- This was a big-ish rewrite, and there was a lot more boilerplate than I imagined to get input and rendering types made for Bones.
- We also had to postpone proper asset handling, and a temporary, annoying integration with Bevy's asset server was used to get things rolling.
- The good news was that porting was straight-forward, if tedious.
- By the time we finished, we had a rather interesting architecture.
- When you started a match, it would create a new Bones world, initialize the game, and then start sending input to it.
- We created a bevy system that would render whatever sprites and tilemaps that it found in the bones world.
- For network games, GGRS would send the input to the bones world, as well as instruct it to create or restore snapshots when necessary.
- After a lot of work we finally had a nice and clean architecture, where we didn't have to think constantly about networking.
- We also got an interesting bonus, the Jumpy match gameplay was now renderer agnostic.
- Because it was isolated and deterministic, we could have rendered it in Bevy, or macroquad without changing the gameplay at all. -->

---

<!-- .slide: data-timing="47" -->

### Bones Framework

- Avoid having to use <span style="color: #C6522C">two</span> ECSs
- Use Bevy in the background for Rendering

Notes:

- Things weren't perfect though.
- For one, now we had two ECSs to deal with.
- This was confusing, especially for new contributors, and there were annoying things like having to
  put the GUI stuff in Bevy, and the gameplay stuff in Bones.
- So we started working on a unified Bones Framework.
- The framework would allow Jumpy to use Bones ECS for everything.
- But in the background, we would still use Bevy for rendering.

<!-- - The next step, was to create a unified Bones Framework.
- We would start moving pieces of Jumpy into the new framework, such as the Egui widgets and the networking logic.
- We would use a multi-`World` strategy, so that you could have one bones world for the menu that would create separate, isolated gameplay worlds for the matches.
- This multi-`World` setup let us keep all the great advantages of having a separate gameplay world, while only needing to learn one ECS.
- The end result, is that the whole entire game could be made with Bones alone.
- Then we would use Bevy as a rendering integration.
- Bones games are renderer agnostic, they take bones-formatted input and create standardized rendering components.
- Any renderer that could render sprites, tilemaps, egui, and debug lines, could be integrated as a renderer, but Bevy would be our official renderer.
- The can of worms in this whole setup was the asset server. -->

---

<!-- .slide: data-timing="35" -->

### Bones Asset Server

- Load YAML files automatically into Rust structs
- Support user asset packs

<span class="fragment" style="position: absolute; right: 0; bottom: -3em; font-size: 0.75em; transform: rotate(-5deg)">✨ and scripting...? ✨</span>

Notes:

- The first step to creating the Bones Framework was to make a proper asset server.
- We had this planned since we started working on the Bones ECS and realized that integrating with
  the Bevy asset server was not working very well.
- We also had some very specific goals around moddability.
- For one, we wanted user asset packs to be a first-class citizen.
- For two, we wanted an easy way to load all of our games config and assets easily through YAML
  files.
- We already had this setup with Bevy, where we used a derive macro to automatically generate asset
  loaders from Rust structs to laod data from YAML files with serde.
- Things got complicated though, when we wanted a YAML file to be able to reference another YAML
  file or another asset.
- It kind of worked, but it was a pain.
- Having our own asset server would let us support both of these use-cases seamlessly, with
  first-class developer experience. ⏭️
- It would also help with scripting in the future.

---

<!-- .slide: data-timing="33" -->

### Runtime Defined Types

Notes:

- When it came to our YAML asset system things were a bit harder than I expected, and that was
  because I was trying to support runtime-defined asset types.
- So, the use case is for modding.
- Currently, each of our YAML assets, were derived from a Rust struct, and the loader wouldn't load
  it if the YAML didn't have exactly the right fields.
- I wanted to keep this type checking, but allow mods to define their own types; types that didn't
  exist in Rust.
- So I wanted some like a JSON schema, for our asset types.
- After some experimentation, though, I realized that this was exactly what we needed for scripting,
  too.

---

<!-- .slide: data-timing="58" -->

### Bones Schema

- Similar to Bevy Reflect
- Designed specifically for scripting
- Describes a <span style="color: #22D491">stable memory layout</span>
- `#[repr(C)]`

Notes:

- This spawned the development of the Bones Schema crate
- Bones Schema is very similar in purpose to Bevy Reflect, but it works quite differently.
- Where Bevy Reflect uses trait objects, a Bones Schema describes a stable memory layout.
- Basically, if you derive the `HasSchema` trait, and you add the `#[repr(C)]` annotation, you get a
  schema describing the exact memory layout of the struct.
- This schema can be accessed at runtime, and can also be created for types outside of Rust.
- Once we got this working, it actually simplified our ECS code, and made the whole ECS compatible
  with runtime defined types.
- It was a **huge** step in the direction of scripting

<!-- - Interestingly, this is the _same_ thing that we needed for scripting.
- The more I thought about it, the more I realized that the asset server, our scripting solution, and even the Bones ECS itself, all needed the same thing:
- a way to describe the layout of data.
- This spawned the development of the Bones Schema crate.
- Bones Schema is very similar in purpose to Bevy Reflect, but it is implemented very differently.
- Instead of using trait objects, like Bevy reflect, we use `Schema`s to describe a C memory layout.
- By adding a `#[repr(C)]` annotation to a Rust struct and deriving `HasSchema`, you get a description of the Rust type's exact memory layout.
- This can be used by the asset server to deserialize assets that match our Rust structs.
- It can also be used by scripting implementations, allowing them to read and modify the data in the Rust types.
- Finally, it worked very elegantly into the design of Bones ECS, allowing it to store anything that has a `Schema`,
- even if that schema was registered at runtime, by a language other than Rust. -->

---

<!-- .slide: data-timing="47" -->

### Porting Jumpy to the Bones Framework

- Jumpy is 100% on Bones!
- It is also <span style="color: #22D491">Renderer Agnostic</span> 👀

Notes:

- After a lot more work to get everything tied together, we finally finished the Bones framework,
  and migrated Jumpy to it.
- Now Jumpy was 100% a bones game and there was no longer the confusion around having two
  entity-component-systems.
- On top of that, Jumpy, and the bones Framework were now renderer agnostic.
- We were currently using Bevy as our rendering backend, but we could easily use anything else.
- In bones, we have only 4 different kinds of renderables: sprites, tilemaps, debug lines, and
  egui.
- Any renderer that could display those 4 renderables could be used as a rendering backend.
- And just because we started with those 4, didn't mean we couldn't easily add new ones, as
  necessary.
- We could also integrate with Bevy plugins if we wanted to later, for things like particles, or 2D
  light effects.

<!-- - While Bones Schema is quite simple in concept, there was a lot to figure out, and it was a big task.
- It took longer than we planned, but we were accomplishing more than we planned, too.
- We had hoped to finish an asset server, and ended up laying the foundation for scripting at the same time.
- I was also really happy with how much more elegant the internals of the Bones ECS were, now that
  it was using Bones Schema to wrap up most of the `unsafe` code.
- After we finally finished Bones Schema and the Bones asset server, we were ready to start moving pieces of Jumpy into the Bones framework.
- This was also really nice, because now our other games could start to re-use things that we had developed for Jumpy.
- Once that was done, we migrated Jumpy completely to the Bones Framework, removing all direct interaction with Bevy.
- Jumpy was now fully a Bones game, and independent from it's renderer. -->

---

<!-- .slide: data-timing="66" -->

### Bones Scripting

✨ Lua scripting with `piccolo`! ✨

<div style="color: #00AFF8; margin-top: 1.5em">Even in the browser 🌐</div>

Note:

- Now that we had Jumpy migrated, we were tantalizingly close to supporting scripting.
- We had been waiting for this for a while, and we had already been in contact with the author of
  the `piccolo` lua VM.
- All we had to do now was plug it in.
- Or hypothetically anyway.
- It turned out to be more work than we thought, like every other step of the process, but it was
  worth it.
- We were finally able to demonstrate adding a brand new item to the game, with a Lua and YAML asset
  pack.
- Personally, this was a huge deal.
- I had been working on getting modding into multiple game engines for nearly 8 years, starting with
  Godot, and every time, I failed.
- Finally, it was working.

<!-- - Now that we had finished migrating Jumpy, we were tantalizingly close to supporting scripting.
- As far as we knew, everything was already in place, other than the scripting language itself.
- We decided on `piccolo` , a Lua implementation written in pure Rust.
- It had big plusses in that it could be used with without any `unsafe`, and it could target web with the `wasm32` target.
- `@kyren`, the author of `piccolo` was _super_ helpful, and worked with us to get the few features we needed in `piccolo` that weren't there yet.
- After, again, more work than we thought, we were finally able to implement a Jumpy item in Lua, and even hot reload it while the game was running!
- Personally, this was a huge milestone, as I had been trying and failing to get modding into multiple game engines for years,
- starting with Godot, then Armory3D, then Bevy, and finally, Bones.
- Finally it was working!
- And the beautiful thing about it, is that it was implemented right into the core ECS.
- Scripts were able to change any `#[repr(C)]` data in the ECS, without us having to explicitly add Lua bindings.
- Now modders and developers could operate on the same world data, without clunky, manually written API bindings. -->

---

<!-- .slide: data-timing="60" -->

### Jumpy & Bones Today

<span style="color: #22D491">Bones works great!</span>

We will try making our own renderer soon

Notes:

- And that brings us to today
- Bones continues to work really well for Jumpy and we really like it.
- It does exactly what we need and we're free to grow it however we need along with Jumpy and our
  other games that we will be building on bones.
- Something we want to do do in the near future is to try making our own renderer for it.
- Bones's rendering needs are really minimal, and right now, pulling in Bevy and it's ECS, it's
  reflection system, and it's renderer; it's just a lot more than we need for bones.
- Also, properly integrating with Bevy's renderer basically means writing a custom sprite pipeline,
  which isn't really much easier than writing our own renderer right on WGPU directly.
- We're hoping that this will cut down on dry compile times, which aren't the most important thing
  in the world, but has been an impact to new contributors.

<!-- - And that brings us to today
- Bones continues to prove its worth as we keep developing and improving Jumpy.
- Jumpy still uses Bevy as it's renderer, but we are considering writing our own renderer, too.
- Jumpy's rendering needs are very simple: we've got sprites, tilemaps, egui, and debug lines,
  that's it.
- Right now, though, we are pulling in a chunk of Bevy code like Bevy Reflect, the Bevy ECS, and the
  Bevy renderer, that is much larger than it needs to be for our use-case.
- This affects compile times, which, while not the most important thing in the world, is impacting contributors.
- Also, Bevy's ECS-driven rendering strategy is not very nice to work with when we are trying to
  render things from another ECS.
- We currently have a good way to inject our sprites into Bevy's render extract phase, but a recent
  update in Bevy ruined that, so we haven't updated beyond Bevy `0.11`.
- We end up either having to write our own sprite renderer for Bevy, or
  we have to synchronize entities, which is not very efficient and is annoying and tricky.
- This means it might be cleanest for us just to make our own renderer. -->

---

<!-- .slide: data-timing="42" -->

### Bones 🤝 Bevy

We never meant to make a game engine

Bevy allowed us to make a game engine <em style="color: #C6522C">piece by piece</em>

<span style="color: #22D491">While still working on our game</span>

<!-- - We never could have done it without Bevy
- It enabled us to explore and branch out, while <span style="color: #22D491">still working on our game</span>
- Bevy allowed us to replace different pieces of it one-by-one
- Bones has learned <span style="color: #C6522C;">so much</span> from Bevy
 -->

Notes:

- So, we never meant to make a game engine.
- Even though we didn't plan on it, we are very happy with the way it turned out, and we could not
  have done it without Bevy.
- Bevy allowed us to continue working on Jumpy, while making Bones piece by piece.
- Bevy was so modular we could combine it with our own ECS, with our own asset system.
- It freed us to do whatever we needed to for our game.
- And we learned _so much_ from bevy.
- It helped inform so many design decisions we made in Bones

<!-- - So, we never meant to make a game engine.
- Our goal was to make we needed for networking and modding, in a way that we could share it with all our Rust games.
- It turned out that what we needed, hadn't been made yet, and by building it step-by-step, we were
  able to make what we needed, even while still working on our game!
- This would not have been possible without Bevy.
- Bevy's modular design allowed us to replace pieces of it while keeping other pieces that we still needed.
- Bevy also inspired many parts of bones; we learned a lot from it, and we still follow Bevy's
  journey and learn from Bevy discussions and development.
- While we may not continue to use Bevy, if we make our own renderer, Bones will forever be in Bevy's debt,
  and never would have existed without it. -->

---

## Thank You Bevy! 🕊️ 🎉

Notes:

- So, to everybody who helped make Bevy what it is, thank you.
- We couldn't have made our game without you.

---

### A Tour of Bones Schema

Notes:

- Now I want to take some time to do a short tour of the Bones Schema system.
- The schema system is probably one of the most interesting and original pieces of Bones, and it's also the piece
  that could be most useful to other projects outside of Bones,
- so it'll be fun to take a closer look at how it works.

---

<!-- .slide: data-auto-animate data-auto-animate-id="simple" data-timing="35" -->

### A Simple Schema

<pre><code data-line-numbers="|4-7|3|10-11" data-trim>
use bones_schema::prelude::*;

#[derive(HasSchema, Clone, Default)]
struct Position {
    x: f32,
    y: f32,
}

fn main() {
  let schema = Position::schema();
  dbg!(schema);
}
</code></pre>

Note:

- First we're going to peek at the simplest thing you can do. ⏭️
- Here we've got a simple `Position` struct with an `x` and a `y` field. ⏭️
- We are just going to derive the `HasSchema`, `Clone`, and `Default` on it. ⏭️
- The `HasSchema` trait just has one function and it returns the schema associated to a type.
- To see what that looks like, we're going to get our `Position` schema and debug print it. 

---

<!-- .slide: data-auto-animate data-auto-animate-id="simple" data-timing="34" -->

### A Simple Schema

<pre style="font-size: 0.6em"><code data-line-numbers="|6-7|8-13" data-trim>
[src/main.rs:11:5] schema = Schema {
    id: SchemaId {
        id: 0,
    },
    data: SchemaData {
        name: u!("Position"),
        full_name: u!("bones_schema_test::Position"),
        kind: Primitive(
            Opaque {
                size: 8,
                align: 4,
            },
        ),
        type_data: TypeDatas(
            [],
        ),
        type_id: Some(
            TypeId {
                t: (
                    15945696820052270695,
                    11636597896589064102,
                ),
            },
        ),
        ..
    },
    layout: Layout {
        size: 8,
        align: 4 (1 << 2),
    },
    field_offsets: [],
}
</code></pre>

Note:

- As you can see, we get a lot of stuff here.
- We're going to look at just a couple things for the moment. ⏭️
- First we see that the schema tells us the short name and the full name of our type, which is very
  handy for debugging, and for figuring out which type to import by name in scripts. ⏭️
- We can also see what _kind_ of data our Position is.
- In this case we see that it's an opaque primitive.
- Since it's opaque all we know about the type is it's size and alignment.
- That's not very much info, but it's just enough that we can insert it, for example, into an ECS.
- If we want any kind of scripting super powers, though, we need more info.

---

<!-- .slide: data-auto-animate data-auto-animate-id="reprc" data-timing="10" -->

### `#[repr(C)]` Schema

<pre ><code data-line-numbers="|4" data-trim>
use bones_schema::prelude::*;

#[derive(HasSchema, Clone, Default)]
#[repr(C)] // <----
struct Position {
    x: f32,
    y: f32,
}

fn main() {
  let schema = Position::schema();
  dbg!(schema);
}
</code></pre>

Note:

- That brings us to `#[repr(C)]`
- Here we have the exact same sample, ⏭️ but with an added `#[repr(C)]` annotation to our struct.
- ⏭️

---

<!-- .slide: data-auto-animate data-auto-animate-id="reprc" data-timing="15" -->

### `#[repr(C)]` Schema

<pre><code data-line-numbers="8|10|12-14|15|22-24" data-trim>
[src/main.rs:12:5] schema = Schema {
    id: SchemaId {
        id: 1,
    },
    data: SchemaData {
        name: u!("Position"),
        full_name: u!("bones_schema_test::Position"),
        kind: Struct(
            StructSchemaInfo {
                fields: [
                    StructFieldInfo {
                        name: Some(
                            u!("x"),
                        ),
                        schema: Schema {
                            id: SchemaId {
                                id: 0,
                            },
                            data: SchemaData {
                                name: u!("f32"),
                                full_name: u!("std::f32"),
                                kind: Primitive(
                                    F32,
                                ),
                                type_data: TypeDatas(
                                    [],
                                ),
                                type_id: Some(
                                    TypeId {
                                        t: (
                                            472265404662890772,
                                            9774757227469882430,
                                        ),
                                    },
                                ),
                                ..
                            },
                            layout: Layout {
                                size: 4,
                                align: 4 (1 << 2),
                            },
                            field_offsets: [],
                        },
                    },
                    StructFieldInfo {
                        name: Some(
                            u!("y"),
                        ),
                        schema: Schema {
                            id: SchemaId {
                                id: 0,
                            },
                            data: SchemaData {
                                name: u!("f32"),
                                full_name: u!("std::f32"),
                                kind: Primitive(
                                    F32,
                                ),
                                type_data: TypeDatas(
                                    [],
                                ),
                                type_id: Some(
                                    TypeId {
                                        t: (
                                            472265404662890772,
                                            9774757227469882430,
                                        ),
                                    },
                                ),
                                ..
                            },
                            layout: Layout {
                                size: 4,
                                align: 4 (1 << 2),
                            },
                            field_offsets: [],
                        },
                    },
                ],
            },
        ),
        type_data: TypeDatas(
            [],
        ),
        type_id: Some(
            TypeId {
                t: (
                    16884739941262504889,
                    10590392532827682936,
                ),
            },
        ),
        ..
    },
    layout: Layout {
        size: 8,
        align: 4 (1 << 2),
    },
    field_offsets: [
        (
            Some(
                "x",
            ),
            0,
        ),
        (
            Some(
                "y",
            ),
            4,
        ),
    ],
}
</code></pre>

Note:

- Now, when we run it, instead of seeing an opaque primitive type, we see a `struct` type. ⏭️
- The struct info tell us the fields of the struct ⏭️
- The names of each field ⏭️
- And the schema of each field ⏭️
- Finally, by looking at the schema of the `x` field, ⏭️ we can see that it is a primitive `f32`.
- The schema now has enough info for scripts to come in and edit the memory of the struct, without
  triggering undefined behavior.
- The schema tells us what can be safely done with the data.

---

<!-- .slide: data-timing="56" -->

### `SchemaBox` for Type Erasure

<pre style="font-size: 0.5em"><code data-line-numbers="3-16|19-24|25-26|28|30|31-32|33-34" data-noescape data-trim>
use bones_schema::prelude::*;

#[derive(HasSchema, Clone, Default)]
#[repr(C)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(HasSchema, Clone, Default)]
#[repr(C)]
struct Velocity {
    x: f32,
    y: f32,
    angular: f32,
}

fn main() {
    let pos = Position { x: 10.0, y: 5.0 };
    let vel = Velocity {
        x: 2.0,
        y: 4.0,
        angular: 0.0,
    };
    let pos_box = SchemaBox::new(pos);     // Like Box&lt;dyn Any&gt;
    let vel_box = SchemaBox::new(vel);

    let boxes = vec![pos_box, vel_box];

    for b in &boxes {
        if let Ok(pos) = b.try_cast_ref::&lt;Position&gt;() {
            dbg!(pos.x, pos.y);
        } else if let Ok(vel) = b.try_cast_ref::&lt;Velocity&gt;() {
            dbg!(vel.x, vel.y, vel.angular);
        }
    }
}
</code></pre>

Notes:

- Now we're going to look at what kinds of abilities this gives us in Rust.
- We'll start by adding another struct, in this case a `Velocity` struct. ⏭️ 
- First we create an instance of both a Position and a Velocity ⏭ ️
- Then we stick them both in their own `SchemaBox`es.
- The `SchemaBox` is one of the most used types in Bones schema, and is in fact very much like a `Box<dyn Any>`.
- By putting the position and the velocity in boxes we "erase" the type, so to speak, so that even
  though there are different types on the inside, they are the same type on the outside. ⏭️
- This lets us do things like stick both of them inside the same vector, which is impossible
  normally, because every item in a vector must be the same type. ⏭ ️
- Now we can loop over the items in the vector. ⏭️
- And attempt to cast it to a specific type.
- In this case we try to cast it to a position, and if it succeeds we print it out. ⏭️
- And we can do the same thing with the velocity.
- This is not really any different than what you can already do with a `Box<dyn Any>`, though,
- so let's get a little deeper.

---

<!-- .slide: data-timing="44" -->

### `SchemaBox` for For Runtime Field Access

<pre style="font-size: 0.5em"><code data-line-numbers="30-36|31|32-34|35-39" data-noescape data-trim>
use bones_schema::prelude::*;

#[derive(HasSchema, Clone, Default)]
#[repr(C)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(HasSchema, Clone, Default)]
#[repr(C)]
struct Velocity {
    x: f32,
    y: f32,
    angular: f32,
}

fn main() {
    let pos = Position { x: 10.0, y: 5.0 };
    let vel = Velocity {
        x: 2.0,
        y: 4.0,
        angular: 0.0,
    };
    let pos_box = SchemaBox::new(pos);
    let vel_box = SchemaBox::new(vel);

    let boxes = vec![pos_box, vel_box];

    for my_box in &boxes {
        let my_ref: SchemaRef = my_box.as_ref(); //   Like &dyn Any
        let x: &f32 = my_ref
            .field("x").unwrap()
            .try_cast().unwrap();
        dbg!(x);
    }

    // [src/main.rs:33:9] x = 10.0
    // [src/main.rs:33:9] x = 2.0
}
</code></pre>

Notes:

- Because the schemas now describe our fields, we are able access them by their names at runtime.
- Here we have the same example as last time, but we modify the loop ⏭️
- Instead of trying to cast to a specific Rust type, now we use the `as_ref()` function get a
  `SchemaRef`, which is similar to a reference `dyn Any`. ⏭️
- Unlike `dyn Any`, though, we are able to extract fields by name, from the reference.
- We use the field function to try and get a field named `"x"` and then we try to cast it to an f32.
- ⏭️ This lets us print the `x` field of each item in our vector, even though one of them is a
  `Position` and the other one is a `Velocity`.
- This kind of ability is essential for scripting, and also very useful for things like ECS world explorers.

---

<!-- .slide: data-timing="42" -->

### `SchemaBox` for Deserializing

<pre style="font-size: 0.55em"><code data-line-numbers="19-22|25|26-28|29|31-33" data-noescape data-trim>
use bones_schema::prelude::*;
use serde::de::DeserializeSeed;

#[derive(HasSchema, Clone, Default)]
#[repr(C)]
struct Position {
    x: f32,
    y: f32,
}

#[derive(HasSchema, Clone, Default)]
#[repr(C)]
struct Velocity {
    x: f32,
    y: f32,
    angular: f32,
}

static MY_POS: &str = r#"
x: 10
y: 5.5
"#;

fn main() {
    let yaml_deserializer = serde_yaml::Deserializer::from_str(MY_POS);
    let pos_box: SchemaBox = SchemaDeserializer(Position::schema())
        .deserialize(yaml_deserializer)
        .unwrap();
    let pos: Position = pos_box.try_cast_into().unwrap();

    dbg!(pos.x, pos.y);
    // [src/main.rs:30:5] pos.x = 10.0
    // [src/main.rs:30:5] pos.y = 5.5
}
</code></pre>

Notes:

- We can also use Bones schema for serializing and deserializing.
- For example, if we have this small YAML snippet that we want to deserialize into a Position.
- We haven't derived `serde::Deserialize` for `Position`, but we do have all the info we need
  already in the `Schema`. ⏭️
- We can create a normal `serde_yaml` Deserializer from our YAML snippet. ⏭️
- And then create a `SchemaDeserializer` for our `Position` schema, and use it to load the data into a SchemaBox.
- ⏭️ After we have the box, we can cast it into a `Position`
- ⏭️ and prove that the YAML data was correctly loaded by printing it.
- This functionality is extremely useful in the Bones asset server, allowing us to automatically
  load YAML assets into Rust structs.

---

<!-- .slide: data-timing="31" -->

### `Schema` for Runtime Defined Types

`Schema`s can be derived or loaded from files at <string style="color: #22D491;">runtime</strong>

Everything works the same

Note:

- So far all of our schemas have come from Rust structs, but schemas can also be loaded from files
  at runtime, and everything works the same.
- This is crucial for scripts that need to be able to store their own components in the Bones ECS,
  even though there isn't Rust type for it.
- Beyond that every Rust type that has that `#[repr(C)]` annotation will be able to have it's fields
  read and written to from scripts, because it's schema tells the script exactly what the script is
  allowed to do with the component.

<!-- - An important thing to note at this point is that schemas can be derived, but they can also be
  loaded from files at runtime.
- For example, mods can define their own schemas for types that never existed in Rust.
- Even still, all of the schema features work the same.
- You can store the data in schema boxes, or in the bones ECS, you can access fields at runtime, and
  you can deserialize the data using serde.
- Scripts and the Bones ECS can access all of the `#[repr(C)]` data, regardless of it's source. -->

---

<!-- .slide: data-auto-animate data-timing="35" -->

### Associated Type Data

Notes:

- There's one more nifty feature that I want to show you in Bones schema, and that is associated
  type data.
- Each schema has a type data store.
- It's just a container that you can put anything you want in.
- There are all kinds of things you can use this for, but it's easiest to explain with an example.

<!-- - There's one more cool feature that I want to show you in Bones schema, and that's Associated Type Data.
- Consider the following use-case: in our game, we know know that we will spawn thousands of certain components.
- For example, maybe we have a bullet component, and there are machine guns in the game.
- We want the ECS to pre-allocate storage for those components, so we can avoid re-allocations later.
- What we can do is create a `StorageMode` enum that tells the ECS whether we want to pre-allocate
  storage or allocate on demand.
- And then we can add our `StorageMode` enum as an associated type data to our components.
- Lets see what that would look like. ⏭️ -->

---

<!-- .slide: data-auto-animate data-timing="47" -->

### Associated Type Data

<pre style="font-size: 0.55em"><code data-line-numbers="3-8|10-16|18-23|20-21|25-32|35-39|41|42-45|47-51" data-noescape data-trim>
use bones_schema::prelude::*;

#[derive(HasSchema, Clone, Default, Debug)]
enum StorageMode {
    #[default]
    OnDemand,
    PreAllocate(usize),
}

#[derive(HasSchema, Clone, Default)]
#[type_data(StorageMode::OnDemand)] // <----
#[repr(C)]
struct Position {
    x: f32,
    y: f32,
}

impl&lt;T&gt; FromType&lt;T&gt; for StorageMode {
    fn from_type() -> Self {
        let size = std::mem::size_of::&lt;T&gt;();
        StorageMode::PreAllocate(1000 * size)
    }
}

#[derive(HasSchema, Clone, Default)]
#[derive_type_data(StorageMode)] // <----
#[repr(C)]
struct Velocity {
    x: f32,
    y: f32,
    angular: f32,
}

fn main() {
    let schemas = vec![
      Position::schema(),
      Velocity::schema(),
      String::schema()
    ];

    for schema in schemas {
        let storage_mode: Option<&StorageMode> = schema
          .type_data
          .get();

        dbg!(storage_mode);

        // [src/main.rs:40:9] storage_mode = Some(OnDemand)
        // [src/main.rs:40:9] storage_mode = Some(PreAllocate(12000))
        // [src/main.rs:40:9] storage_mode = None
    }
}
</code></pre>

Notes:

- Let's consider a potential use-case in our ECS.
- When you add components to an entity, it allocates enough storage for that component on-demand.
- If you were to spawn a bunch of entities, then it would re-allocate storage multiple times.
- But if you knew that you were going to, for example, have thousands of our `Position` component,
  it would be more efficient to allocate storage for say 10,000 positions up front.
- We could indicate this by adding a `StorageMode` type data, to our `Position` component.
- For example, here we defined a simple storage mode enum that has an `OnDemand` option and a
  `PreAllocate` option. ⏭️
- Then we come down to our `Position` component and we our chosen storage mode in with a
  `#[type_data]` annotation.
- In this case, we say we want positions to be allocated on demand.
- And it's that simple.
- We can add any rust expression inside the type data annotation, so we make function calls to get
  the type data, or add structs with parameters, it really flexible. ⏭️
- Another thing we can do is we can derive type data.
- In order to make our `StorageMode` enum derivable, we have to implement the `FromType` trait for
  it.
- This is a contrived example, ⏭️ but here we are saying that if you derive `StorageMode` on any type
  `T`, it will `PreAllocate` 1000 times the size of the the type `T`. ⏭️
- After we've done that, we can derive the type data with the `derive_type_data` attribute.
- It's all super simple. ⏭️
- And now we want to see how we'd use the type data.
- Lets pretend that we're the ECS, and we have a list of schemas that we need to allocate storage for.
- Notice I threw the schema for `String` in there, to, which doesn't have a `StorageMode` type data. ⏭️
- We loop over each schema. ⏭️
- And for each one we get the `StorageMode` out of it's type data. ⏭️
- And in this case, we just print it out.
- We can see that we have the storage modes for `Position` and `Velocity`, and that for `String` it wasn't set.


<!-- - First we define our `StorageMode` enum, we have an `OnDemand` variant and a `PreAllocate` variant. ⏭️
- Next we can use the `#[type_data]` attribute to set the storage mode to on-demand for our `Position` struct.
- Adding type data is super easy!
- You can even add the attribute multiple times to associate more type data.
- The `type_data` attribute comes from the `HasSchema` derive macro, and it lets you put in any Rust expression.
- This lets you make things extra ergonomic by letting you put function calls in the annotation.
- We use this in the bones asset server for setting custom asset loaders.
- It's cool because feels almost like making custom attributes without having to make a new macro for each new attribute.
- To make things even _more_ ergonomic certain type data can be _derived_. ⏭️
- For example, lets say we wanted to let you derive the `StorageMode` type data.
- To do that we implement `FromType` for `StorageMode`. In this example implementation, ⏭️
- we get the size of the type, ⏭️
- And then we tell it to pre-allocate 1000 times the size of our type.
- This is definitely a contrived example, but it illustrates the point. ⏭️
- Now, instead of setting the `type_data` directly, we can use the `derive_type_data` attribute to
  automatically gernate the `StorageMode` for that type.
- This is very convenient.
- Now, we can use our type data by getting it from the schema. ⏭️
- We come back to our example of looping over schema boxes. ⏭️
- This time, we get the schema, and load the `StorageMode` from it's type data. ⏭️
- And we can print out the storage mode for each item. ⏭️
- The cool part about all of this is that you can use it for whatever you want!
- It's kind of like the bones schema version of implementing a trait. ⏭️ -->

---

### Use Cases for Type Data

- Asset loaders
- Custom Deserializers
- Bindings for scripts

Notes:

- There are a lot of possible uses for type data.
- In bones we use them to specify asset loaders and custom deserializers when the default one isn't
  sufficient.
- Eventually you'll be able to use them specify customized bindings to scripting languages.
- They're basically the traits of the Bones Schema world, and like everything else in the crate,
  focus on giving you more access at runtime, to things that normally only work at compile time.

---

<!-- .slide: data-timing="12" -->

### That's All! 👋

<svg height="32" aria-hidden="true" viewBox="0 0 16 16" version="1.1" width="32" data-view-component="true" style="fill: white; transform: scale(2)">
    <path d="M8 0c4.42 0 8 3.58 8 8a8.013 8.013 0 0 1-5.45 7.59c-.4.08-.55-.17-.55-.38 0-.27.01-1.13.01-2.2 0-.75-.25-1.23-.54-1.48 1.78-.2 3.65-.88 3.65-3.95 0-.88-.31-1.59-.82-2.15.08-.2.36-1.02-.08-2.12 0 0-.67-.22-2.2.82-.64-.18-1.32-.27-2-.27-.68 0-1.36.09-2 .27-1.53-1.03-2.2-.82-2.2-.82-.44 1.1-.16 1.92-.08 2.12-.51.56-.82 1.28-.82 2.15 0 3.06 1.86 3.75 3.64 3.95-.23.2-.44.55-.51 1.07-.46.21-1.61.55-2.33-.66-.15-.24-.6-.83-1.23-.82-.67.01-.27.38.01.53.34.19.73.9.82 1.13.16.45.68 1.31 2.69.94 0 .67.01 1.3.01 1.49 0 .21-.15.45-.55.38A7.995 7.995 0 0 1 0 8c0-4.42 3.58-8 8-8Z"></path>
</svg>

<https://github.com/fishfolk/bones>

Notes:

- And that wraps things up.
- I'm super excited about what we've accomplished so far, I'm really grateful for all the work that
  has gone before us, and I can't wait to see what's next.
- Bones and everything in it are public on GitHub so if you're interested you can check it out!
- We'll have a Q&A session here if you have any questions, and if we don't to to your question,
  definitely feel free to reach out to us on Discord or GitHub.
- We're always happy to chat.
- Thanks for having me on here, and thanks for listening everyone!
