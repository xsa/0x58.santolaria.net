## xmodmap to the rescue of PowerBook Belgian keyboard 

After `cwm(1)` release three got imported in the [OpenBSD tree](http://www.openbsd.org/cgi-bin/cvsweb/xenocara/app/cwm/)
I thought I'd give it a try someday -- That day has arrived.

My `-current` main terminal to hack is a PowerBook6,8 with a Belgian keyboard. Since I've been using X on this laptop with that keyboard I
had to bring `xmodmap(1)` into the dance. After a few hours of fiddling with all those modifier keys, I finally came up with
a `~/.Xmodmap` file that would suit my need in ways that the keyboard shortcuts would work as expected with `cwm(1)` and other applications.

Nothing fancy, but maybe this could help other wandering souls:

```
clear Shift
clear Lock
clear Control
clear Mod1
clear Mod2

keycode 14 = parenleft 5 braceleft bracketleft
keycode 20 = parenright degree braceright bracketright
keycode 23 = Tab ISO_Left_Tab
keycode 35 = dollar asterisk EuroSign yen
keycode 36 = Return
keycode 37 = Control_L
keycode 46 = l L bar bar
keycode 49 = less greater guillemotright guillemotleft
keycode 50 = Shift_L
keycode 51 = dead_grave sterling
keycode 54 = c C copyright copyright
keycode 57 = n N asciitilde dead_tilde
keycode 60 = colon slash division backslash
keycode 64 = Alt_L
keycode 66 = Caps_Lock
keycode 94 = at numbersign
keycode 107 = BackSpace Delete Delete BackSpace
keycode 115 = Mode_switch
keycode 116 = Mode_switch

add Shift = Shift_L
add Control = Control_L
add Lock = Caps_Lock

add Mod1 = Alt_L
add Mod2 = Mode_switch
```

Just add the following line to your `.xinitrc` file and you are done:

`xmodmap ~/.Xmodmap`

[*Published on Aug. 01, 2007*](https://x-sa.blogspot.com/2007/08/xmodmap-to-rescue-of-powerbook-belgian.html)
