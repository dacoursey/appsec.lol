---
layout: post
title: "Certificate Installation on Android in 2019"
author:
modified:
tags: [Mobile,Pentesting,Hacking,SSLPinning,Android]
---

This post is intended to be a list of commands I use to get the Burp certificate installed on Android devices version 8.0 or greater. I am not going to go into a long explanation on what they do.  Most of it was stolen from [@kevcody](https://twitter.com/kevcody) anyway so check out his full [post](https://nvisium.com/blog/2017/07/12/advantages-and-disadvantages-of-android-n-network-security-configuration.html) if you need more help.

I did add a couple bits at the end to get around errors that I was seeing on Pixel devices.

#### Prerequisites

1. Android Studio on the host.
2. Rooted Android 8.0 or greater device.
4. Burp Suite running on host.

#### Execution

```console
// Obtain the Burp CA cert by browsing to http://burp name it burpca.cer
// Convert the .cer file to .pem
$ openssl x509 -inform der -in burpca.cer -out burpca.pem

// Obtain the certificate hash of the .pem cert.
$ openssl x509 -inform PEM -subject_hash -in burpca.pem | head -1

// Copy output hash. (Mine was 7bf17d07 so replace that with yours.)
// Make a new file.
$ cat BurpCA.pem > 7bf17d07.0

// Modify the file to make it 'Android style'.
$ openssl x509 -inform PEM -text -in burpca.pem -out /dev/null >> 7bf17d07.0
```

Here's where you may run into trouble.  There are two methods that I know of:

##### Option A

```console
// Connect with adb.
$ adb root

// Prep the filesystem for copying up your cert.
$ adb remount

// Move the cert to the device.
$ adb push 9a5ba580.0 /system/etc/security/cacerts/

// Set the correct file perms on the cert.
$ chmod 644 /system/etc/security/cacerts/9a5ba580.0
```

If you run into an error like `Cannot run adb as root in production builds.` then you have to try the next method.

##### Option B

```console
// Move your cert up to a non-root place you control.
$ adb push 7bf17d07.0 /data/local/tmp/7bf17d07.0

// Shell in to the device
$ adb shell

// git gud
marlin:/ $ su

// Validate your FS type is ext4 or whatever it is
marlin:/ $ cat /proc/mounts | grep /system

// Remount the file system as RW
marlin:/ $ mount  -o  rw,remount  -t  ext4  /system

// If you want, validate you are now mounted RW
marlin:/ $ cat /proc/mounts | grep /system

// Copy your cert to the necessary location
marlin:/ $ cp /data/local/tmp/7bf17d07.0 /etc/security/cacerts/7bf17d07.0

// Make sure the OS can read it
marlin:/ $ chmod 644 /etc/security/cacerts/7bf17d07.0
```

Now whatever method you used, reboot the device and go to:

`Setttings > Security > Encryption & Credentials > Trusted Credentials`

Scroll down to Portswigger and make sure it's enabled. You should be good to go at this point. Configure the device wifi connection to go through the Burp proxy and point your browser at https://portswigger.net to test. You should have HTTPS traffic in Burp now.

If you run into other problems hit me up on twitter @dacoursey.