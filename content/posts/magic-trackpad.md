---
title: "Configure Apple Magic Trackpad 2 Gestures on Manjaro KDE"
date: 2020-05-17T21:59:33-04:00
draft: true
toc: false
images:
tags:
  - linux
  - kde
  - manjaro
---
### Intro

I haven't seen any full-length tutorials for getting the Apple Magic Trackpad 2 working in Manjaro or more specifically KDE yet so I figured I'd write down how I did it.  The device works out of the box with left, right, and middle-click as well as 2 finger scrolling. But to get the rest of the features you need to set a few things up. YMMV on whether or not everything in this works for you.

---

### Getting Started
  
The very first thing you should know is that for fully supported drivers you need to be on at least Linux version *4.20*, I am running on *5.4.43-1* which is Manjaro's LTS kernel release.  
  
Once you are on a kernel version that supports the Trackpad drivers, plug in the device and it should immediately start working for the most part. However, the multi-finger gestures do not work, as nothing is currently bound to them. For that, we need *libinput-gestures*. I've seen a couple forum posts to suggest using other input libraries but the only one that works in this scenario I've seen is libinput.

```sudo pacman -S libinput-gestures```

Go ahead and install *libinput-gestures*. The first thing we should do is add our user to the *input* group.

```sudo gpasswd -a $USER input```

Form there we can run the following commands:

```bash
libinput-gestures-setup autostart
libinput-gestures-setup start
```
Alright awesome, now that its up and running we can go ahead and shut it back down and start building our config.

```libinput-gestures-setup stop```

Your config file exists in 2 places: `/etc/libinput-gestures.conf` and `~/.config/libinput-gestures.conf`. The former is for default gestures, all users on the system will share these gestures. The one in *.config* is personal to your user. I will be using the one in *.config* but it shouldn't matter.

```nano .config/libinput-gestures.conf```

Here is a paste of my config file:

```
# Swipe threshold (0-100)
swipe_threshold 0

# Gestures
gesture swipe up 3 konsole
gesture swipe down 3 xdotool key Super_L+Page_Down
gesture pinch out 3 xdotool key Ctrl+F9
gesture swipe right 3 xdotool key Ctrl+Shift+F2
gesture swipe left 3 xdotool key Ctrl+Shift+F1
```

The format for the gestures is pretty straight forward: `gesture` means just that, create a gesture. Then you define the gesture type. These can be `swipe` and `pinch`. Then specify the direction with `up` `down` `left` and `right`. Next you define the number of fingers to perform the gesture with. I have not testedwith any more than 3 fingers for gestures, I would imagine 4 works though I dont know about 5 and above. Finally you give it the command to run. For this I would suggest *xdotool* as you can pretty much do anything that the mouse and keyboard can do with that command. Mine just do keybinds though you can do some more complex macro'ing if youre into that sort of thing.

Once you save your config file run:

```libinput-gestures-setup start```

Doing the defined gestures should run the command you specified. Should you notice that its not a quick way to ensure that libinput is running the command is to stop *libinput-gestures* again and then run:

```
libinput-gestures -d
(Ctrl+C to stop)
```
This will show you the commands that would be ran once a recognizable gesture is detected. If all is good go ahead and start *libinput-gestures* again.

### Is that it...?

Thats it! At this point the trackpad will work and you can bind gestures to your hearts content... However, you'll soon notice that if you connect the Trackpad through Bluetooth, and have to reconnect the trackpad, that gestures stop registering. The gestures will also stop working sometimes once waking from a lock or sleep. 

The only way to fix this is by restarting the gesture service. You can do this by hand, but using *dbus-action* is a more elegant solution. Here is how to set that up.

---
### Funny Title Here


