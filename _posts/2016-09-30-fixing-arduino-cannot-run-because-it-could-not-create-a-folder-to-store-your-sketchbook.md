---
layout: post
title: Fixing "Arduino cannot run because it could not create a folder to store your sketchbook"
---

Today at work I was trying to install the Arduino IDE on my computer and kept running into this issue. ![Arduino Error](/images/capture.png). 

This is very annoying as it keeps the IDE from even starting, so you can't go into the settings and change the default sketchbook folder. The problem was the fact that, this being a work computer, a network drive was acting as my home directory. This meant that my documents folder was no longer `C:\Users\<user>\Documents`, but was instead something like `\\\<remote_drive>\<user>\Documents` and the Arduino IDE could not get access to it, even when run in administrator mode. One way to fix this by changing the location of the folder in a preferences file as described [here](http://forum.arduino.cc/index.php?topic=49554.0) but that wasn't working for me. The preferences file was either not being created or was ending up in a nonstandard place. Instead I temporarily changed the registry key pointing to the my documents by following the instructions [here](http://www.liutilities.com/products/registrybooster/tweaklibrary/tweaks/10470/). I'll describe the process here for completeness: 

  1. Open `regedit` by hitting `Windows Key + R`, typing it in there, and hitting `OK`
  2. Now navigate to `HKEY_CURRENT_USER > SOFTWARE > Microsoft > Windows > CurrentVersion > Explorer > Shell Folders`
  3. Find the property called `Personal` and note its value.
  4. Now change this value to some temporary folder (e.g. C:\temp). Make sure this folder exists. Don't close the `regedit` window yet.
  5. Now run Arduino, it _should_ run now. Go to `File > Preferences` and change the `Sketchbook location` entry to some local folder that exists and you have write access to. Apply/save as needed.
  6. Go back to the `regedit` window and change the property back to its original value.
  You should be able to use the Arduino IDE now and your network mounted personal folder should work as expected as well :). [1]

## Comments

**[S Charlesworth](#43 "2018-03-27 16:18:17"):** I tried this, and while it did let me start the app (every 10th time I tried), I couldn’t install libraries, which made it basically useless to me. I ended up going old-school with the zip download. It works anyway…

**[Vivek A N](#34 "2017-12-03 19:09:26"):** I thank you very much, ive been spending the whole week trying to fix this problem. You are a genius…..

**[maatthc](#35 "2017-12-12 15:27:25"):** Hi there, You can just set the correct folder location using the file : C:\Users\\(username)\AppData\Local\Arduino15\preferences.txt .

**[Roman Frołow](#33 "2017-09-22 21:44:21"):** This worked for me: In C:\Users\\\AppData\Local\Arduino15 create file preferences.txt with content: sketchbook.path=C:\Users\\\Documents\Arduino

