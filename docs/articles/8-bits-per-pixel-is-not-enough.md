## 8 bits per pixel is not enough

As I just moved in my new apartment, I still have not all my machines,
and thus it is about time to use the ones I use less than wanted because of role duplication.

In this case I'm speaking about laptops, and more specifically my [PowerBook's](http://www.openbsd.org/macppc.html#hardware)
(`PowerBook3,4` and `PowerBook6,8`).

So, when I booted my "old" PowerBook (yeah, the G4 800 DVI - Ti) I felt
somehow disgusted about the way the resolution looked in 8bit using the wsfb driver.
Nonetheless, as always, you can expect good documentation about nearly anything in the
OpenBSD project, and thus, I found what I wanted in the [distribution notes of the XF4 for macppc](http://www.openbsd.org/cgi-bin/cvsweb/XF4/distrib/notes/README.macppc).

Knowing my PowerBook had an ATI graphic card, this was so easy that I just had
to follow what was explained in /usr/X11R6/README. Here is the relevant line in the dmesg about the graphics card:

`vgafb0 at pci0 dev 16 function 0 "ATI Radeon Mobility M7 LW" rev 0x00, mmio`

Now I wonder why I never took the time to do that when back in the days
I used to use this laptop quite often. Lazyness I believe ;-)

Of course, this now brings us to another debate about the [X Aperture](http://marc.info/?l=openbsd-misc&m=114233317926101)
and the security concerns around it. 

[*Published on Nov. 07, 2006*](https://x-sa.blogspot.com/2006/11/8-bits-per-pixel-is-not-enough.html)
