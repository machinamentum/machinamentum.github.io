---
layout: post
title: Proposition for a better OS
categories: [OS]
---

## Introduction
Over the past few years I've been increasingly losing interest in interacting with major operating systems for my day-to-day needs. Major operating systems have become largely bloatware that is more intrusive to the user's experience than it is useful. I will try to outline the surface features at this time. Features that are non-user facing (within the kernel, programming interface, etc..) will be outlined at a later time.

These are the basic philosophies for designing this new OS:
 1. Operating systems should be designed to help the user get from point A to point B or perform task XYZ with as little friction as possible.
 2. A design choice should not compromise philosophy(1), or philosophy(2).
 3. The OS should be stripped of all bloatware.
 4. The OS should be as unintrusive to the user as possible.
 5. Upgrading the OS should improve the user's experience, not degrade it.
 6. The OS should be as easy and as accessible as possible; it should support multiple points of interaction by default. Whether the user wants to use a keyboard and mouse, a touch screen, or even a gamepad, the OS should be agnostic to these common points of interaction and should provide a pleasant experience.

## The Terminal
Currently, command terminals support basic IO, command execution, and simple text rendering (coloring, italics, bold face, etc..). These aren't bad features at all, this small feature set is what makes interacting with consoles still a pretty good experience. However, this limiting feature set induces poor implementation choices of software that uses the console as a primary user interface. A prime example of a such program is the GNU debugger. GDB is an otherwise good debugger, but it's console-based user interface makes even the most trivial of debugging tasks impossible to efficiently get done. We could do better.

#### Bitmap rendering
I first heard about this idea from Gary Bernhardt of Destroy All Software in his talk [A Whole New World](https://www.destroyallsoftware.com/talks/a-whole-new-world). Although Gary mentions a few interesting uses for terminal raster graphics, there's much wider implications of what this feature would be capable of. This enables full-featured UI's from within a terminal. A terminal app could render a webpage, draw game graphics, or display a powerpoint. Typically, a terminal app like (for example) [gource](http://gource.io/) would have to open a separate window to draw its output. The user then has to exit the newly opened window the click back into the terminal. This is unneeded friction that does nothing but waste the user's time.

#### Mouse/agnostic pointer input
Since we have to support philosophy(6). The terminal should be able to do more with a pointer device than just copy-paste. As a natural, harmonious progression with bitmap rendering, a terminal app should be able to capture basic pointer input like position and clicking/tapping.

#### Scripting language
The terminal should support a robust scripting language to allow the user to automate tasks or interact with the system. Although this is a current feature of most terminals, many terminal languages are designed to do most of their work in the form of executing other terminal programs. Scripts in this language should be native to the system; they should be able to work like any other program without having to invoke other terminal programs to perform a task.

## Windowing system
Although the features proposed in *The Terminal* would make windowing obsolete, there's still a major use for apps exclusively designed around a window system. The terminal is typically thought of as a power-user-y interface; it inherently requires a keyboard and a bit of learning to effectively use it. A windowing system still provides a smaller amount of friction than the terminal.

iOS and Window's MetroUI are forward-thinking systems. They maximize simplicity while minimizing (or trying to) friction. A simple tile-based UI would be an ideal solution to support philosophy(6). It would allow the user to use a keyboard and mouse, a touch screen, a gamepad, or any other abstract input devices without any compromises. This could be further extended to allow navigating the entirety of the filesystem from this interface.

If well-executed, we could do away with desktop environments. Although they're highly preferred now on desktop PC's, I believe power-users will become accustomed to these low-friction systems.

## Developer Tools
Developer tools for many OS' tend to be bloated and not very user friendly. Although we can't strictly avoid legacy toolchains, we can at least provide a better user-experience for developers as part of the OS ecosystem.

#### Debugger
One of the things Windows does right that Unix-based OS' do wrong is that Windows provides debugging utilities as part of the kernel. Anyone wishing to implement a custom debugger can simply link against the kernel's public interface library and can get language-agnostic debugging callbacks that any user program can easily interpret and manipulate. We can take this core idea of the debugger being a part of the OS a little further by having executable object parsing, op-code disassembling, and symbol information loading being shipped as part of the debugging API. User-built debuggers don't have to rely on these OS provided utilities, but the advantage to having a standard API for these is that anyone trying to implement a reasonable debugger without large dependencies, can just depend on the OS provided libraries and they should just work.

#### API deprecation
Some API designers tend to believe that deprecation is "mark for removal". I don't agree with this idea. A deprecated API should remain available as long as there is software to depend on it. If the API in question breaks one of the core philosophies, it may be justifiable to remove the API from the core OS, but it must still be accessible to any user who wishes to use it through some other means (such as a compatibility package).

## Adoption
This will probably be the trickiest part to execute. If we look at Steam's [hardware survey](http://store.steampowered.com/hwsurvey) alone, Windows dominates the market at 95% of users. Although this is just based on gamers, I'm sure these numbers tip even more in favor of Windows for casual PC users and business users. I don't believe Mac OSX is a real competitor in this space since the target for this OS would be PC users (where Mac OSX is solely designed for Mac devices). Although Linux is somewhat popular in the open source space, I don't think it is a direct competitor. Adoption for Linux is still very small and still suffers from some of the problems I'll outline below.

#### GPU drivers and games
This will be one of the most important aspects of the OS to get the gaming space to adopt. Unfortunately, the main issue here is a chicken and egg problem. In order to get GPU vendors to implement drivers for our OS we need to have a significant standing in the gaming space by having games and users. In order to have games and users, we need to have good driver support to get developers to even consider porting their games. Fortunately, we can still get the process started by implementing our own drivers for a subset of hardware (Intel's integrated graphics for example have [open specifications](https://www.x.org/docs/intel/). Although this is less than ideal, this would be a step in the right direction.

#### Pre-installed on devices
This would be a goal to reach much farther in development, but it is very important to get the OS established as software that is pre-installed on machines. This is one of the main aspects of Linux that causes its low adoption rate (although Ubuntu tried this for awhile and failed).

I'll explain the importance of this with an analogy. When users purchase an Android device for example, most do not buy the device with the intention of side-loading a different flavor of Android. Most users purchase an Android phone and use the pre-installed version of Android because it is just there and there's no friction involved in having to set it up. Likewise, if Android phones only shipped with vanilla versions of Android with the option of installing manufacturer versions, the install base for manufacturer-based versions (i.e., Samsung's TouchWiz or HTC's Sense) would be much lower, perhaps even near non-existent. To further expand on this thought, CyanogenMod (arguably the largest non-OEM Android version, which isn't typically pre-installed to devices) claimed in [August 2015 to have 50M+ users](https://www.instagram.com/p/6IUuRVNH_b/). According to [The Verge](http://www.theverge.com/2015/8/20/9181269/gartner-q2-2015-smartphone-sales), Android phone sales had hit ~271M units _in just the quarter leading up to_ August 2015. The number of Android devices with pre-installed versions far, _far_ out weigh the number with custom OS versions installed. 
