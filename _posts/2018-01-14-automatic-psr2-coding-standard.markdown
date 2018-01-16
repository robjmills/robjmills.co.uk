---
layout: post
title:  Automatic psr2 coding standard with PHP_CodeSniffer and Git
date:   2018-01-14 18:14:55 +0000
categories: dev php git phpcs psr2
---

![vagrant](/assets/images/php-fig.jpg){:class="img-responsive"}

We've recently adopted the [psr2 coding standard][psr2-standard] on our application. We've always followed an informal coding standard but we decided it made sense to bite the bullet and fully adhere to psr2. If psr2 is new to you, it's a coding style guide that was developed by the [PHP Framework Interopability Group Group (aka PHP-FIG)][php-fig-link] to aid the sharing of code by enforcing a common standard:

> PSR-2's purpose is to have a single style guide for PHP code that results in uniformly formatted shared code.

Agreeing to adopt a coding standard is one thing, but enforcing it is another. With developers across different environments, using their own choice of editor, enforcing a set of rules can be difficult. Personally, I use PhpStorm, but occasionally i'll do something in Atom, or Vim.

We were keen to make sure that adoption of psr2 would be friction-free, but consistently adopted. In this regard it made sense that it could be added to our workflow in a way that was mandatory. Encouraging devs to add rules to their editor would be a pain to maintain, and easy to override. You just know that some dev's are really going to struggle giving up their tabs!

The popular [PHP_CodeSniffer][phpcs-link] (phpcs) tool allows checking against various coding standards, and automatic fixing via PHP Code Beautifier and Fixer (phpcbf). It made sense to make use of these external tools, enforcing their versions via Composer and hooking into that workflow.

Git is central to our development workflow. No code can be merged into our codebase without first being peer reviewed via a pull-request, and then merged by another team member. It made sense then that the phpcs checks we required were enforced by Git, and in this case a Git pre-commit hook. There's a few tricks in how we set this up (further blog post coming soon!).

First thing was to get phpcs itself installed with the required coding standard. This was done by installing the following packages into the dev section of our `composer.json`:

{% highlight json %}
{
  ...
  "require-dev": {
    "dealerdirect/phpcodesniffer-composer-installer": "^0.4.4",
    "wimg/php-compatibility": "^8.1"
  }
  ...
}

{% endhighlight %}

Once installed it was just a case of creating the hook itself in `.git/hooks/PRE-COMMIT`. Much of this code was taken from a gist i've seen in many places. I would happily credit the original author but I have no idea who it was.  

{% highlight bash %}
#!/bin/bash

PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"`
STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`

# Determine if a file list is passed
if [ "$#" -eq 1 ]
then
    oIFS=$IFS
    IFS='
    '
    SFILES="$1"
    IFS=${oIFS}
fi
SFILES=${SFILES:-$STAGED_FILES_CMD}

echo "Checking PHP Lint..."
for FILE in ${SFILES}
do
    php -l -d display_errors=0 ${PROJECT}/${FILE}
    if [ $? != 0 ]
    then
        echo "Fix the error before commit."
        exit 1
    fi
    FILES="${FILES} $PROJECT/$FILE"
done

# Make sure the environment variable has been set
if [ "$APPLY_CODE_STYLE" = "" ]
then
   echo "You must set the environment variable APPLY_CODE_STYLE"
   exit 1
else
    # Run the code style fixes if phpcs exists and environment variable is set
    if [[ -f "vendor/bin/phpcs" && "$APPLY_CODE_STYLE" == 'true' ]]
    then
        if [ "${FILES}" != "" ]
        then
            echo "Running Code Sniffer. Code standard PSR2."
            ./vendor/bin/phpcs --standard=PSR2 --encoding=utf-8 -n -p ${FILES}
            if [ $? != 0 ]
            then
                exec < /dev/tty
                echo "If you would like to commit without running phpcs, run your commit overriding the env var with: 'APPLY_CODE_STYLE=false git commit'"
                echo 'Fix automatically (where possible)? [y/N]'
                read proceed
                if [[ ! ("$proceed" == 'y' || "$proceed" == 'Y' || "$proceed" == '') ]]
                then
                    exit 1
                else
                    echo "automagically fixing..."
                    ./vendor/bin/phpcbf ${FILES}
                fi
            fi
        fi
    fi
fi
{% endhighlight %}

The original code for the hook file runs the supplied changes through phpcs and checks them against the PSR2 standard. However, I added a few additional things that were useful to us. First of all I added a dependency on an environment variable we would force all our developers to add. This variable would set your default preference for whether the check runs. As much as we wanted the check to always run, we decided on reflection there may be times where we want to disable it. This would now be possible via temporary overriding of the variable by running `APPLY_CODE_STYLE=false git commit`. The other enhancement I added was an interactive question for whether the user would like `phpcbf` to automagically fix all errors it was able to.

With this workflow in place, it became the standard that any new commit would highlight where code was not following psr2 and attempt to fix it for you. However, at this stage we still hadn't enforced the very presence of the hook we wanted to be reliant on. I will cover that in a new blog post shortly.

[php-fig-link]: http://www.php-fig.org/psr/psr-2/
[phpcs-link]: https://github.com/squizlabs/PHP_CodeSniffer
[psr2-standard]: https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md
