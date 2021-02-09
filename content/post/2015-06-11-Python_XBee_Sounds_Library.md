---
title:  "Notes and Chords using Python"
date:   2015-06-11
description: Because you're too cheap for a real instrument.
<!-- url: Python_XBee_Sounds_Library/ -->
---

Life events: my wife and I will be closing on a house in a couple days.  Yay!

Things have moved quick since moving here to Seattle and I've realized I haven't been as good about posting on this blog.  Just to get content on here, I figured I would share a recent small project I've been working on.

In what time I can spare I've working on an education course for a hackerspace / STEAM education group here in Seattle.  I wanted to model the course after my Masters Thesis work, so one idea was getting youth engaged with using wireless communications.  One tool I thought would aid in engaging youth would be if they could easily program Arduino/XBee boards to play music, even chords on a remote machine.  This also aids given the interrelation of sound and radio communications having wave properties.  The basic idea is that the kids would program Arduino (or similar) microcontrollers to send messages via XBee that are translated on a central computer into music notes.

I found plenty of existing examples of generating sine waves using numpy, and turning that data into notes with pyaudio.

These examples didn't suite my situation because they all used blocking calls to play audio.  You issue the command to play a note and can't issue another until the note was finished.  I wanted to stream audio, and update the stream if a new note was entered.  This required a bit more reading into the pyaudio API.  Since this was a bit of effort for me, I figured I would share for others.

This is just a starting framework that turns a computer keyboard into a music generating keyboard.  Currently it just assigns notes at random, or sequentially (see commented line 29).  Once a key is pressed, it continues to play the same note.  If all notes are assigned, random notes are given to new keys.

<script src="https://gist.github.com/serdmanczyk/5a9389d80db94440cf15.js"></script>
