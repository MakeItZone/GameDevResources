**Table of Contents**
- [Visual Studio Code](#visual-studio-code)
- [Git](#git)
- [Continuous Integration](#continuous-integration)
- [Unity Store Packages](#unity-store-packages)
- [Movement that Doesn't Go Through Walls](#movement-that-doesnt-go-through-walls)
  - [Common Settings/Tweaks](#common-settingstweaks)
  - [References](#references)
- [Organizing the Hierarcy](#organizing-the-hierarcy)
- [Health Bars](#health-bars)
- [Importing Blender Files](#importing-blender-files)
  - [General Setup](#general-setup)
  - [Animations](#animations)

# Visual Studio Code

It's worth going through the [full setup](https://code.visualstudio.com/docs/other/unity). Do it once, and then copy the few extras into your next project.

The debugger seems to be a dead project, but for now it still works with some [tweaks](https://github.com/Unity-Technologies/vscode-unity-debug/issues/202).

# Git

Follow all the instructions from [thoughtbot](https://thoughtbot.com/blog/how-to-git-with-unity).

Make sure your git client has LFS support!

Make sure any remote git servers have LFS support (and their caps- often quite low for lfs and free accounts.)

Gitea maybe your teams friend!

Make sure all assets are contained inside the directory that is in git. E.g. if you add blender files manually (by exporting FBX), make sure the orginal blender files are included in the git repo.

Probably not worth trying to merge changes in Unity scene files; know how to choose the upstream (or your) version in the git client app of your choice.

After a `pull` you may get errors re unfound prefabs, missing files, etc. Try closing Unity and deleting the `Library` directory. It contains caches etc that may not always be updated if you change things via Git. When you reopen the project Unity will rebuild the `Library` directory and things should work.

# Continuous Integration

See the [CI notes](../Continuous%20Integration/readme.md)

# Unity Store Packages

If someone on the team adds a package, even if it's a free (cost and sharable license) it will problaby mess up when imported by another teammate.

Guess: each person will have to "purchase" the package, and make sure it's available for use in their projects.

TODO: verify above guess works.

TODO: check re licensing/distribution of packages- e.g. if the content ends up in your project, if it needs to be explicitly excluded in github, etc

# Movement that Doesn't Go Through Walls

*TL;DR - don't use `transform.Translate` or even `rigidbody.MovePosition()`! Use `rigidbody.AddForce()` only!*

Super simple, but will be buggy: `transfrom.translate`. Very easy to push through walls, fall through floors. Used in a lot of samples, as it's really obvious what's going on. Also all movement directions are given in the game objects *local space*, so you can avoid discussing world vs local coordinates.

Slightly less simple, slightly less buggy: `rigidbody.Moveposition()`. Can tweak settings highlighted below to make it so that you can "push and peak" through walls, but you will be pushed back after a frame or two and not pass through the wall. Does require understanding world vs local coordinates.

Vocabulary: 
* static collider : just has a `collider` component. Will interact with rigidbodies; won't be moved by the physics sim.
* dynamic collider : has a `rigidbody` component. Will interact with rigidbodies and colliders. Will be affected by physics sim *unless* `isKinematic` is set (meaning movement is fully under script control.)

See [this page](https://docs.unity3d.com/Manual/CollidersOverview.html) for details and a handy summary table.

## Common Settings/Tweaks

* on player characters `rigidbody`: 
    * set `Collision Detection` to continuous. Slight performance penalty, but checks the physics more often, less chance to pop through objects
    * set `interpolation` to `interpolate`, especially if you're using `rigidbody.AddForces`. You'll want to put the `forces` into `fixedUpdate`, which get's called at a different rate to `update`. If this is not set, you'll probably find your player reacting jerkily to inputs. 
* `edit menu -> project settings`: 
  * `Physics -> Default Max Depenetration Velocity` - increase (not needed if using rb forces)
  * `Time -> Fixed Timestep` - fixed frame rate for the physics sim. Defaults to 50fps. More than 75fps (0.01333) tends to make thing twitchy and bad. (Not needed if using rb forces)
  * `Input Manager`, or custom input settings if using the new input manager - may want to tweak the `acceleration`, `snap to` etc for how digital inputs (wasd) are translated to analog axis. OR don't use the axis and just check for keys being pressed.
  * explore the other physics and input settings to tweak how things perform

## References
* [really good intro to using RB physics to not go through walls](https://answers.unity.com/questions/1788697/how-to-fix-my-player-from-phasing-through-walls.html) - note, mentions `rb.MovePosition` but in my limited experience, that didn't work reliably
* [more detailed example using rb to move](https://answers.unity.com/questions/1743970/make-player-not-go-through-walls.html) - note, it's set up to move character aligned to world coordinates. Doesn't work with mouse look type setups. For that you just need to add something like `transform.TransformDirection(moveInput) * moveSpeed` before using the vector with the RB forces (which are in world space)
* [explaining the different rb force modes](https://answers.unity.com/questions/789917/difference-and-uses-of-rigidbody-force-modes.html)
* [stopping a rigid body quickly - set the velocity and rotational velocity](https://answers.unity.com/questions/662811/rigidbody-how-to-stop-it-quickly.html)
* [a thread discussing ways to try and stop going through walls](https://forum.unity.com/threads/what-are-the-necessary-settings-to-prevent-objects-passing-through-each-other-at-high-speeds.384519/)

# Organizing the Hierarcy

Can use empties as "folders" to keep the hierarchy sane. It does have a performance penality, but for smaller project, it shouldn't be an issue.

# Health Bars
- Simple sprite based health bar UIs:
  - [Concise Medium Post](https://medium.com/swlh/game-dev-how-to-make-health-bars-in-unity-from-beginner-to-advanced-9a1d728d0cbf)
  - [Slightly longer form tutorial, with video](https://weeklyhow.com/how-to-make-a-health-bar-in-unity/)
  - [summerized tutorial](http://gyanendushekhar.com/2019/11/17/create-health-bar-unity-3d/)
  - [google search on the topic](https://www.google.com/search?q=unity+add+a+health+bar+to+ui)
- [discussion on how to draw a rect on the UI](https://forum.unity.com/threads/draw-a-simple-rectangle-filled-with-a-color.116348/) - more technical details, with some potential problems. Key thing: don't (re)create objects on every update/gui update call!

# Importing Blender Files

There are two workflows:

1. drop the blender file onto the Unity asset manager. Internally Unity runs Blender (why you need it installed) and converts to FBX. Don't forget to enable the "create colliders" option if you need it.

2. export as FBX from blender, then import the FBX into Unity. Blender's exporter has some useful flags for applying scale, transform, and more. Don't forget collider options on import!

Using a blender file directly is faster to develop with.

Using an explicit fbx gives more control. Can seperate editing from updating the game.

## General Setup
Unity and Blender have different handedness for their coordinate systems (z up vs y up). To make things import with zero issues, especially animations:

(NOTE: need to verify this is the right sequence):

* design in blender so -ve y is the "forward" direction
* apply all rotation & scales, make sure object origin is in the right place.
* rotate around x -90 degrees and apply rotation
* rotation around x 90 degrees *but do not apply rotation*
* export/import into Unity


## Animations

Can create multiple animation sequences for an object by using `Dope sheet->Actions`.

When you add a blender file, Unity launches blender and executes a script to convert to FBX.
By default it doesn't export all the Actions. Can edit `/Data/Tools/Unity-BlenderToFBX.py` in your Unity editor install and change `bake_anim_use_all_actions=False`  to set it to `True`. It should be around line 43. From [here](https://answers.unity.com/questions/1747701/importing-multiple-separate-animations-from-a-blen.html). 

Some references also suggest adding the actions to (multiple) NLA timelines, and manually exporting to FBX. 

Can also export each model as fbx, and then export each animation seequence as an fbx with no geo, just the animation. Name them `model@animation.fbx` and Unity should associate them correctly. Should make it easier for your devs, or to update specific animations. [Details are here](https://docs.unity3d.com/Manual/Splittinganimations.html), [and some comments](https://www.reddit.com/r/Unity3D/comments/2ojqoz/importing_multiple_animations_in_one_fbx/)

Good YouTube tutorial series for bringing in animations from blender: [here](https://www.youtube.com/playlist?list=PLq7npTWbkgVBAtQs4p4iYxlfWrKRkNc6O)
