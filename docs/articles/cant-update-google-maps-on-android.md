## Can't Update Google Maps on Android Rooted Phone

Bummer. Every time I tried to update Google Maps via the Market it wouldn't install it.
Throwing out an error about package signature or what not. It appears that it just does not
work to update an Android stock app on a rooted phone. What then? You will first have to
delete/uninstall the Google Maps packages, then just simply go back to the Market,
search for the package, and install it. Bam; done.

Below are the commands to use from your ADB shell to remove the package.

```
 adb root
 adb remount
 adb shell rm /system/app/Maps.apk
 adb uninstall com.google.android.apps.maps
 ```
[*Published on Oct. 10, 2010*](http://x-sa.blogspot.com/2010/10/cant-update-google-maps-on-android.html)
