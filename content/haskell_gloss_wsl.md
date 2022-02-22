---
author: Linus Wæhler
title: Haskell and Gloss with WSL2
date: 2022-02-22
description: Setting upp Haskell and Gloss library on Windows, *the easier way*. Written primarily for classmates having issues with this.
math: true
comment: true
categories: ['haskell', 'gloss library', 'wsl2', 'windows', 'linux']
---

# Windows + Linux = 💕

## Setting up the WSL2 machine

**You must be running Windows 10 version 2004 and higher (Build 19041 and higher) or Windows 11.**

You can install everything you need to run Windows Subsystem for Linux (WSL) by entering this command in an administrator PowerShell or Windows Command Prompt and then restarting your machine.

```powershell
wsl --install
```

This command will enable the required optional components, download the latest Linux kernel, set WSL 2 as your default, and install a Linux distribution for you (Ubuntu by default). [^1]

If any issues appear, please refer to [Microsoft's guide.](https://docs.microsoft.com/en-us/windows/wsl/install)

[^1]: This is excerpted from Microsoft's guide to [installing WSL](https://docs.microsoft.com/en-us/windows/wsl/install).

## Installing some packages

After your reboot, you should be able to open a terminal and type wsl to enter the Linux environment. Go ahead and do this, since we want to install some packages.

#### Update

The first thing we want to do is update the package repository, do this with the following command:

```bash
sudo apt-get update
```

#### x11-apps (for testing)

The next thing we want to install is x11-apps, this is mainly for testing the X-Server later, but it will save you a headache, trust me. 😅

```bash
sudo apt-get install x11-apps
```

#### OpenGL

And the final thing we need to get Gloss library running later on is GLUT, the best version of this we can get on Ubuntu would be FreeGLUT3, which should be included in the universal repository. We will also install all of the required OpenGL libraries.

```bash
sudo apt-get install libglu1-mesa-dev freeglut3-dev mesa-common-dev
```

## Configuring the X11 server

### Windows configuration

The next step is to install an X-Server, my recommendation is [VcXsrv](https://sourceforge.net/projects/vcxsrv/), it's very simple to install and should only take you a few minutes.

Run the shortcut that appears on your desktop and set the options to:

- Multiple Windows
- Start no client
- **DISABLE** Native OpenGL
- **ENABLE** Disable access control

Press the save config option, and put it somewhere nice. You can even put this in your startup folder in order to have it run on startup.

### Linux configuration

Navigate to your home directory `cd ~` and open your `.bashrc` file in a text editor.

Add this to the bottom of it:

```bash
HOST_IP=$(host `hostname` | grep -oP '(\s)\d+(\.\d+){3}' | tail -1 | awk '{ print $NF }' | tr -d '\r')
export LIBGL_ALWAYS_INDIRECT=1
export DISPLAY=$HOST_IP:0.0
export NO_AT_BRIDGE=1
export PULSE_SERVER=tcp:$HOST_IP
```

### Verifying the X11 Server

Verify in your system tray that the X-Server is running in the background, and try typing in `xcalc` in your WSL terminal, a window should pop up in Windows 10.

Another way to verify that the X-Server is running is by running the netstat command. Open an elevated prompt (admin) and run the following command.

```powershell
netstat -abno|findstr 6000
```

which returns a table

```powershell
TCP    0.0.0.0:6000           0.0.0.0:0              LISTENING       12752
TCP    127.0.0.1:6000         127.0.0.1:55766        ESTABLISHED     12752
TCP    127.0.0.1:6000         127.0.0.1:55767        ESTABLISHED     12752
TCP    127.0.0.1:6000         127.0.0.1:55768        ESTABLISHED     12752
```

If you see something like this, it is indeed working.

## Haskell + Gloss = λ

### aptitude (recommended)

Installing cabal and ghc is very easy on Ubuntu, you can simply apt-get it!

```bash
sudo apt-get install ghc cabal-install
```

Now run a quick `cabal update` and then `cabal install gloss`.

Open up ghci and type the following commands.

```haskell
import Graphics.Gloss
```

```haskell
display (InWindow "Nice Window" (200, 200) (10, 10)) white (Circle 80)
```

If you see this window...

![Nice Window](https://i.imgur.com/rx3IuK6.png)

... you're done! 🎉

If not, well, some annoying troubleshooting is ahead my friend. Check out the [Verifying the X11 Server section](#verifying-the-x11-server), and try opening port 6000 inbound in your firewall.

If all else fails, contact me if you know me, or leave a comment down below!
