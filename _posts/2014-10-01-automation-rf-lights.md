---
layout: post
title: "Automation of RF Light Bulbs"
author: "Richard"
tags:
---

I was given some (actually, quite a few) RF-controlled LED light bulbs for my birthday last month. Obviously the first thing that I wanted to do with them was build in some automation (turn on/off at x o'clock, control them from my phone, etc.). I was, therefore, glad to discover that the gift package I received included a "Wifi Bridge", which enables control of paired lightbulbs via a simple UDP-based protocol.

Said bridge does not support WPA Enterprise, so for the time being my apartment's wireless is using a PSK all over again. I'm a bit sad about this, as I had a neat little system that sent me notifications when certain users connected, or when people attempted to connect with incorrect credentials. At some point I may attempt to replicate the bridge's functionality using an Arduino and a 2.4GHz transceiver module, but for now, I have to live with it.

Luckily, the protocol for the bridge is documented by the manufacturer, so I don't have to use their hideous app to do any configuration. At the moment I have built a small Python script to talk to the bridge (given its IP). I also created a PHP-based AJAX web UI that can be used to control my lights, using said Python script. It's a bit roundabout, but I can control my lights from my PC and phone, which is good enough for now. Will probably work on it a bit more before I post the code up, as it's incredibly rough and has no security (past the PHP basics of "don't be dumb with user input").

Using the same Python script I have been able to create some simple shell scripts that, along with cron, automate my lights' behaviour at various times of the day. At this point in time, there's no fading in/out, or conditional stuff.

Over my summer break, I may do more work on this. Some ideas I'd like to implement:

* Develop my current Python script into a daemon that responds to certain events from a rule list (some of these events may originate from an improved web UI script).
* Add support for environmental events, such as "device x connected to Wi-Fi", "light levels dropped below threshold", and "motion detected".

I ordered some very cheap RF communication boards and a PIR motion sensor (Arduino compatible) with the intention of using a wireless motion trigger to turn on my bathroom light (that is otherwise unused). Unfortunately I now understand that, for some reason, reception quality with the receiver module on the Raspberry Pi's GPIO pins is terrible (I believe this may be to do with their maximum clock rate, but I'm not sure). I will probably have to order another Arduino board as a result, so am pushing any further work on this until after exams are done!
