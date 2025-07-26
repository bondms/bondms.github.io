---
title: Google Pixel 6a Battery Cycle Count
---

# Pixel 6a Battery Performance Program

In July 2025, Google announced that some Pixel 6a devices would receive a mandatory update that would reduce battery performance in order to mitigate the risk of potential overheating. This reduction in batter performance would only take effect ater the battery reached 400 cycles. For full details, see the [Pixel 6a Battery Performance Program](https://support.google.com/pixelphone/answer/16340779).

Pixel 8a and later devices are able to display the battery cycle count natively from the "About phone" > "Battery information" settings page, see [Understand your Pixel battery](https://support.google.com/pixelphone/answer/15738128).

For those Pixel 6a devices that have exceeded 400 cycles, the "Battery" settings page gives both a "Battery replacement recommended" message and a "Charge cycle count" number, see [Reddit](https://www.reddit.com/r/pixel_phones/comments/1lxr4ir/anyway_to_disable_the_new_battery_update_on_pixel/) for an example.

Pixel 6a owners who have not yet reached 400 battery cycles may be curious to know their current battery batter cycle count in order to make a judgment on how much more use they have remaining before the update will apply the performance degration. Knowing this will help to decide if/when to replace the phone/battery or to take up Google's [Support options](https://support.google.com/pixelphone/workflow/16310202).

Unfortunately, there doesn't seem to be a convenient way to determine the battery cycle count on Pixel 6a devices that have yet to reach 400. There may be some third-party apps that provide that feature and Google may release a future update to display it natively, but an alternative for now is to access the information via [Android Debug Bridge (ABD)](https://developer.android.com/tools/adb).

# Access battery cycle count via ADB

With the Pixel 6a device connected to a workstation running ADB, the battery cycle count can be accessed via a shell command. There are several options available, but perhaps the simplest is to connect the device with a USB data cable and proceed as follows.

## Enable Developer options on the Pixel 6a device

As per [Enable Developer options](https://developer.android.com/studio/debug/dev-options#enable)

1. Go to the "About phone" settings page.
1. Tap the "Build number" option seven times. Re-authenticate to confirm you wish to become a devloper.
1. Go to the "System" > "Developer options" settings page.
1. Enable "USB debugging". Click "OK" to confirm.
1. When promted to allow USB debugging from the connected computer, click "Allow".

## Install ADB on the workstation

This will vary depending on the type of workstation available. For a Debian 12 Linux workstation, simply install the packages for ADB and the Adroid SDK platform tools:

```bash
sudo apt install adb android-sdk-platform-tools-common
```

## Fetch battery cycle count via ADB client shell command

From a terminal on the workstation, run the ADB command:
```bash
adb shell cat /sys/class/power_supply/battery/cycle_count
```

This will print an integer indicating the current battery cycle count.

# Notes

The battery cycle count is an indication of charge cycles equivant to complete 0 to 100% charges, so the count will increment by 1 only after two 50-100% charges for example.

Other battery extended information is available via ABD. Try these commands for exmaple:

```bash
adb shell dumpsys battery
adb shell cat /sys/class/power_supply/battery/cycle_counts
```

# References

* [Reddit: Checking the battery health accurately using ADB dumpsys](https://www.reddit.com/r/GalaxyS9/comments/kvcjmn/checking_the_battery_health_accurately_using_adb/)
* [How To Check Battery Cycle Count and Capacity](https://xdaforums.com/t/guide-how-to-check-battery-cycle-count-and-capacity-root.4123177/)
