---
layout: post
title:  "ProFTPD on Mac"
date:   2018-07-09
---

<p class="intro"><span class="dropcap">I</span> didn't expect to find it so hard to run ProFTPD as a daemon on the Mac. Without further ado, here is the right `Launchctl` script to get it running on startup on MacOS 10.13 High Sierra.</p>

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>KeepAlive</key>
      <dict>
        <key>SuccessfulExit</key>
          <true/>
      </dict>
    <key>Label</key>
      <string>org.proftpd.proftpd</string>
    <key>ProgramArguments</key>
      <array>
        <string>/usr/local/sbin/proftpd</string>
        <string>-n</string>
      </array>
    <key>RunAtLoad</key>
      <true/>
  </dict>
</plist>
{% endhighlight %}

Save the above script as `/Library/LaunchDaemons/org.proftpd.proftpd.plist`.

{% highlight bash %}
$ sudo launchctl load /Library/LaunchDaemons/org.proftpd.proftpd.plist
{% endhighlight %}

Unfortunately, the example provided at [ProFTPD: Stopping and Starting](http://www.proftpd.org/docs/howto/Stopping.html) is currently slightly outdated.
