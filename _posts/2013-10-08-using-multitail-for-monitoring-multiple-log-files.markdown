---
layout: post
title:  Using multitail for monitoring multiple log files
date:   2013-08-08 11:14:55 +0000
categories: dev
---

Like many developers my job tends to include a number of low-level sysadmin tasks. I generally have terminal.app open most the day with one thing or another, whether working locally or SSH’ed into one of our remote servers. Once an app is in production its really handy to keep an eye on the server logs to see whats happening and be able to respond proactively to errors as they occur. Multitail is a great tool I found for monitoring multiple log files at the same time, helping to keep all of this monitoring in a single window.

![vagrant](/assets/images/multitail.png){:class="img-responsive"}

In simple terms multitail allows you to monitor multiple files simultaneously. In my case this is almost always the apache error_log file but it could be access logs, ftp logs or anything really.

A simple use of multitail could be:

```
multitail
-l "ssh root@REMOTE.IP.1 tail -f /usr/local/apache/logs/error_log"
-l "ssh root@REMOTE.IP.2 tail -f /usr/local/apache/logs/error_log"
```


One of the most powerful features in multitail is the ability to add exceptions based on regular expression patterns. This allows you to filter out any errors which you’re not as interested in. For example, if you’re monitoring a log for PHP errors you may be less interested in 404 errors. This can lead to a more advanced multitail usage like this which includes named windows and multitail divided into vertical columns:

```
multitail -du -C -s 2
-Ev "does not exist" -Ev "filter this" -Ev "dont show this"
-t WindowName1 -l "ssh root@REMOTE.IP.1 tail -f /usr/local/apache/logs/error_log"
-t WindowName2 -l "ssh root@REMOTE.IP.2 tail -f /usr/local/apache/logs/error_log"
-t WindowName3 -l "ssh root@REMOTE.IP.3 tail -f /usr/local/apache/logs/error_log"
-t WindowName4 -l "ssh root@REMOTE.IP.4 tail -f /usr/local/apache/logs/error_log"
```

Installation of multitail is really simple if you’re using Homebrew, simply `brew install multitail` and you’re ready to go.
