---
layout: post
title:  Restart Docker for Mac from Terminal
date:   2019-03-22 11:30:55 +0000
categories: cli docker tips
excerpt: Quick tip for restarting docker for Mac from Terminal
---

Quick tip on how to restart Docker for Mac from your Terminal. There's no CLI for Docker for Mac, so to restart Docker you need to do this from the menu bar app. I created a useful alias that would restart docker and also give me a visual prompt about this (since it can take a minute or 2)

{% highlight bash %}
alias dres="osascript -e 'quit app \"Docker\"' && open -a Docker &&
  while ! docker system info > /dev/null 2>&1; do sleep 1; done && osascript -e 'display notification \"Docker restarted\" with title \"Info\"'"
{% endhighlight %}

This is an alias for ZSH but it _should_ work on Bash too.
