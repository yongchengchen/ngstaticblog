<!--
Categories = ["Development", "Others"]
Description = ""
Tags = ["Development", "Others"]
date = "2018-12-24T21:47:31+10:00"
title = "MacOS Hammerspoon fast switch mouse cursor between screens"
-->

### add code below into init.lua

```lua
function moveCursor() 
    screen = nil
    pos = hs.mouse.getAbsolutePosition();
    if pos.x < 1500 then
        screen = hs.screen.find({x=1, y=0})
    else
        screen = hs.screen.find({x=0, y=0})
    end
    rpos = hs.mouse.getRelativePosition();
    if rpos ~= nil then
        hs.mouse.setRelativePosition(rpos, screen);
    end
end
hs.hotkey.bind({"ctrl", "alt"}, "left", moveCursor)
hs.hotkey.bind({"ctrl", "alt"}, "right", moveCursor)
```
