---
title: Some useful customizations for macOS.
date: 2024-05-14
permalink: /posts/2024/05/useful-macos-customizations/
tags:
  - macOS
  - apple
  - customizations
---

I started using a Mac back in 2018. My computer, an fairly standard HP laptop I'd bought back in 2013, was beginning to become a source of troubles. The charger wouldn't charge, the keyboard wouldn't hold the keys in place, and generally the computer wouldn't compute. This was a problem, given that I was a destitute student. To my surprise, older MacBooks (the white poylycarbonate MacBooks made back in 2010) where dirt cheap, even when bought from overseas and thus with shipping and custom costs included in the price. I bought one for about 130€. It lacked the Euro (€) key and I couldn't type words containing an [ñ](https://en.wikipedia.org/wiki/Ñ) directly, but it felt like a small inconvenience and I quickly found workarounds. That little laptop proved hardy and powerful enough for my daily tasks, and it carried me through a computational physics course during which the rubber in the bottom peeled due to the excessive heat emanating from its dying processor. It was a pretty nice machine.

I've been using a MacBook since then and after the initial love affair weaned, the machine revealed to me some of its flaws. I do not think they are important flaws, and all of them can be fixed almost instantly. Since I need to go back to them every once in a while, I think having them here would be more convenient than searching through StackExchange. So here they go!

1. I find macOS' standard key speed a little slow. If you want to delete a word letter by letter, or type twenty spaces, or go ten lines up in the same paragraph, pushing a key and waiting for it to be executed N times takes longer than it should. Luckily, the waiting time between repetitions can be taken down from the standard value with the following line pasted onto the terminal:
```bash
defaults write NSGlobalDomain KeyRepeat -int 1
```

2. The dock takes a fraction of a second to appear once you hover near the side of the screen where it's located. Furthermore, the animation with which it emerges takes another aditional fraction of a second, resulting in a somewhat slow experience. Both delays can be toned down so that the dock shows instantly and emerges with a quick animation. I found my ideal combination of settings to be the following:
```bash
defaults write com.apple.dock autohide-time-modifier -float 0.50; defaults write com.apple.dock autohide-delay -float 0; killall Dock
```

3. The central key of the mouse cannot be used to scroll down. Or, well, it can, but the behaviour is so bad I'd rather just click on the slider. That's less than ideal. A quick solution that makes the mouse behave like it should can be achieved using [Hammerspoon](https://www.hammerspoon.org), an incredibly handy app that lets you use [Lua](https://www.lua.org) scripts to change or expand the behaviour of macOS. [Alex Burdusel](https://superuser.com/users/219382/alex-burdusel) @ StackExchange came up with [this script](https://superuser.com/questions/303424/can-i-enable-scrolling-with-middle-button-drag-in-os-x), which works beautifully. It maps the vertical and horizontal movement of the mouse into a scrolling action only when the scrolling button is being pressed.

```lua
-- HANDLE SCROLLING WITH MIDDLE MOUSE BUTTON PRESSED
local deferred = false

overrideOtherMouseDown = hs.eventtap.new({hs.eventtap.event.types.otherMouseDown}, function(e)
    deferred = true
    return true
end)

overrideOtherMouseUp = hs.eventtap.new({hs.eventtap.event.types.otherMouseUp}, function(e)
    if (deferred) then
        overrideOtherMouseDown:stop()
        overrideOtherMouseUp:stop()
        hs.eventtap.middleClick(e:location(), pressedMouseButton)
        overrideOtherMouseDown:start()
        overrideOtherMouseUp:start()
        return true
    end
    return false
end)

local oldmousepos = {}
local scrollmult = 2 -- negative multiplier makes mouse work like traditional scrollwheel, for macOS, use positive number.

dragOtherToScroll = hs.eventtap.new({hs.eventtap.event.types.otherMouseDragged}, function(e)
    deferred = false
    oldmousepos = hs.mouse.getAbsolutePosition()
    local dx = e:getProperty(hs.eventtap.event.properties["mouseEventDeltaX"])
    local dy = e:getProperty(hs.eventtap.event.properties["mouseEventDeltaY"])
    local scroll = hs.eventtap.event.newScrollEvent({-dx * scrollmult, -dy * scrollmult}, {}, "pixel")
    -- put the mouse back
    hs.mouse.setAbsolutePosition(oldmousepos)
    return true, {scroll}
end)

overrideOtherMouseDown:start()
overrideOtherMouseUp:start()
dragOtherToScroll:start()
```

4. Opening TextEdit does not create a blank file. Instead, window asking which file to open comes up. To make it show a blank file directly, the following works:

```bash
defaults write com.apple.TextEdit NSShowAppCentricOpenPanelInsteadOfUntitledFile -bool false
```

I still have some pending customizations to add. This post will be in continuous evolution, as long as I keep using macOS, so feel free to email me if you find some error in the code provided!
