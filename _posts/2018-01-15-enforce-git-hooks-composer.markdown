---
layout: post
title:  Enforcing Git hooks with Composer
date:   2018-01-14 18:14:55 +0000
categories: dev composer git
excerpt: Using composer install to enforce installation of Git hooks.
---

![vagrant](/assets/images/git.png){:class="img-responsive"}

In a [previous article]({% post_url 2018-01-14-automatic-psr2-coding-standard %}) I discussed about automatically applying the PSR-2 coding standard using a Git pre-commit hook. In that post, I wrote about the importance of making sure these standards were followed, but also about how to make this easy for the developers.

At the time I was working on this, we had a number of things happening in our `repo/.git/hooks/pre-commit` file. When a new developer joined the team we had to make sure they added the hooks, alongside everything else. I was sure we could do better here.

The solution I implemented was to leverage the power of Composer scripts. When working on a shared PHP application, best practice informs that the `composer.lock` file should be version controlled. This way, each developer will be using the exact same version of each library (assuming they're on the same branch). Change branch, run `composer install` and you're all good.

A less well used feature of Composer in the scripts section, and the corresponding events. Given the first thing a new developer will do is run `composer install` this seemed to make sense as the starting point for applying our hooks. I added a shell script that would run following the install command and symlink the directory containing our hooks into the repo hooks.

{% highlight json %}
  {
    "require-dev": {
      "dealerdirect/phpcodesniffer-composer-installer": "^0.4.4",
      "wimg/php-compatibility": "^8.1"
    },
    "scripts": {
      "post-install-cmd": [
        "./scripts/symlink.sh"
      ]
    }
  }
{% endhighlight %}


{% highlight bash %}
  #!/bin/sh
  if [ ! -L .git/hooks ];
  then
      echo ".git/hooks is not symlink"
      echo "copying .git/hooks to .git/old_hooks"
      mv .git/hooks .git/old_hooks

      echo "symlinking ../scripts/git-hooks .git/hooks"
      ln -s ../scripts/git-hooks .git/hooks
  else
      echo ".git/hooks is already a symlink"
  fi
{% endhighlight %}

The first step was adding these hooks as a set of individual files within `repo/scripts/git-hooks`. This meant each hook could be encapsulated individually. These in turn would be loaded through the `pre-commit` file:

{% highlight bash %}
  #!/bin/bash

  currentdir="${0%/*}"

  source ${currentdir}/hooks/commit_guard.sh
  source ${currentdir}/hooks/coding_style.sh

  exit 0
{% endhighlight %}


In this way, we enforced the psr2 check, and another hook we were already using to prevent commits on branches that can only be updated via pull requests.
