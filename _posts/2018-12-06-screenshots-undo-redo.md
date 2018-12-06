---
layout: post
title:  "Undo/Redo in Firefox Screenshots"
date:   2018-12-06 17:18:41 -0500
excerpt: In which I discuss implementing undo/redo in Firefox Screenshots.
---

One of the features I implemented for [Firefox Screenshots](https://screenshots.firefox.com/) is the undo/redo in the image editor on the web site.  I'll explain how that works.

(This post will not include any code.  Firefox Screenshots is open source, so visit [its repo](https://github.com/mozilla-services/screenshots/) if you want the code.)

There are at least two obvious but less than optimal solutions:
 - keep a full image of every edit
 - record and replay user edit commands

### Keep All the Images

This is the simplest approach.  It's obvious and obviously bad: keeping all the images around will utilize a lot of memory.

### Replay User Commands

Since the editor know all the editing actions performed by the user, it could save them and replay them back to achieve undo/redo.  This approach definitely will use far less memory than keeping `n` canvas elements around.  And it should work well when there are very few edits.

But it will not scale.  If a user made `n` edits, and perform an undo, then the editor would need to replay `n-1` commands to achieve it.  (It also needs to be done in the background while the user get to view a lovely spinner...)  This is problematic when `n` is large enough for the given computer.

### Maybe a Compromise?

What if we combine the two?  The editor could keep some full size images as key frames, and then replay commands from a key frame to achieve the desired undo/redo.  This reduces memory usage and improves user-perceived latency.

Now what should be the frequency of the key frames?  The crop action nautrally produces key frames, so those should be used.  Of course, the user might not crop the image at all, so the editor will still need to save a key frame periodically.  With this approach, when the user asks for an undo at `x`, and if there's a key frame saved at `x-5`, then the editor only need to replay four edit commands to accomplish the undo action.

So we have some key frames and all the edit commands.  And we need to figure out which key frame to use, and how many commands to replay.  And when to add a key frame.  And if the user make an edit after a undo/redo, then we need to drop the key frames and recorded commands that are after the applied undo/redo.

It's getting just a little complicated.

### Making a Difference

Without a lot of data on user image editing behavior, I want to keep it simple, at least for the initial implementation.  (And I figure that people won't be doing _that_ much editing with a simple web based image editor.)  That leads us back to the first approach.

But it's not necessary to save the _full_ size image on every edit; the editor could save only the affected area in order to perform the undo and redo (which is an undo of an undo).  For crops, full images are saved.

Saving and applying "diffs" is what ended up in the Firefox Screenshots annotations editor for undo/redo.
