## HTC Incredible S, Android, Linux, and udev(7)

So it all started because I wanted to take screen captures of my HTC Incredible S
(I will not explain how to do that here as you'll find plenty of good tutorials on the interwebs).

One of the steps involved the [Dalvik Debug Monitor Server (DDMS)](http://developer.android.com/guide/developing/debugging/ddms.html)
which provides all kind of things you could expect from a debugging tool, including a screen capture
feature. Unfortunately, I quickly noticed that it wasn't really working as expected with
my new HTC Incredible S device on my Linux laptop (running RHEL):

```
$ pwd
/full/path/to/android-sdk-linux_x86/tools
$ ./ddms

(ddms:15838): Gtk-WARNING **: gtk_widget_size_allocate(): attempt to allocate widget with width -5 and height 16

(ddms:15838): Gtk-WARNING **: gtk_widget_size_allocate(): attempt to allocate widget with width -5 and height 16
09:01:27 E/DDMS: insufficient permissions for device
com.android.ddmlib.AdbCommandRejectedException: insufficient permissions for device
    at com.android.ddmlib.AdbHelper.setDevice(AdbHelper.java:736)
    at com.android.ddmlib.AdbHelper.executeRemoteCommand(AdbHelper.java:373)
    at com.android.ddmlib.Device.executeShellCommand(Device.java:276)
    at com.android.ddmuilib.SysinfoPanel.loadFromDevice(SysinfoPanel.java:159)
    at com.android.ddmuilib.SysinfoPanel.deviceSelected(SysinfoPanel.java:126)
    at com.android.ddmuilib.SelectionDependentPanel.deviceSelected(SelectionDependentPanel.java:52)
    at com.android.ddms.UIThread.selectionChanged(UIThread.java:1684)
    at com.android.ddmuilib.DevicePanel.notifyListeners(DevicePanel.java:752)
    at com.android.ddmuilib.DevicePanel.notifyListeners(DevicePanel.java:740)
    at com.android.ddmuilib.DevicePanel.access$1100(DevicePanel.java:56)
    at com.android.ddmuilib.DevicePanel$1.widgetSelected(DevicePanel.java:357)
    at org.eclipse.swt.widgets.TypedListener.handleEvent(Unknown Source)
    at org.eclipse.swt.widgets.EventTable.sendEvent(Unknown Source)
    at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)
    at org.eclipse.swt.widgets.Display.runDeferredEvents(Unknown Source)
    at org.eclipse.swt.widgets.Display.readAndDispatch(Unknown Source)
    at com.android.ddms.UIThread.runUI(UIThread.java:492)
    at com.android.ddms.Main.main(Main.java:103)

09:01:32 W/ddms: Unable to get frame buffer: insufficient permissions for device
```
Not what I expected, but nothing that cannot be fixed. So after googling/fiddling/ranting a bit,
`udev(7)` came to the rescue. Pretty much, it seems I needed another rule to be added.
So here's how I did it:

Look at the information that pertain to my HTC Incredible S:

```
$ lsusb
Bus 002 Device 006: ID 1267:0210 Logic3 / SpectraVideo plc
Bus 002 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 009: ID 0bb4:0cac High Tech Computer Corp.
Bus 001 Device 005: ID 0a5c:217f Broadcom Corp. Bluetooth Controller
Bus 001 Device 003: ID 17ef:480f Lenovo Integrated Webcam [R5U877]
Bus 001 Device 002: ID 8087:0020 Intel Corp. Integrated Rate Matching Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

Create a specific `android.rules` file in custom rules directory `/etc/udev`
(Note that it is one single line in the `.rules` file below)

```
$ id
uid=500(xsa) gid=500(xsa) groups=500(xsa),496(desktop_admin_r)
$ sudo -s
# cat > /etc/udev/android.rules
SUBSYSTEMS=="usb", ATTRS{idVendor} =="0bb4", ATTRS{idProduct} =="0cac", MODE="0660", OWNER="root", GROUP="desktop_admin_r"
^D
# ln -s ../android.rules /etc/udev/rules.d/90-android.rules
```

Reload the `udev(7)` rules and then launch again DDMS and you should be all set.

```
# /etc/init.d/udev-post reload
[...]
# exit
$ cd /full/path/to/android-sdk-linux_x86/tools
$ ./ddms 
```

[*Published on Jun. 10, 2011*](http://x-sa.blogspot.com/2011/06/htc-incredible-s-android-linux-and.html)
