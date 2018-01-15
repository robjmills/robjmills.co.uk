---
layout: post
title:  Automatic psr2 coding standard with php code sniffer, composer and git
date:   2018-01-14 18:14:55 +0000
categories: dev php composer git
---

![vagrant](/assets/images/php-fig.jpg){:class="img-responsive"}

Another important tool within our workflow is composer, which we use for our dependency management. 

```
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

```

```
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

```
