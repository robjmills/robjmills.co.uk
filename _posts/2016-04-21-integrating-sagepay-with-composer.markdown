---
layout: post
title:  Integrating Sagepay using Composer ...or using Composer to manage a non-Composer compatible library
date:   2016-04-21 18:14:55 +0000
categories: dev php composer
---

![vagrant](/assets/images/composer-lock.png){:class="img-responsive"}

_originally posted on blog.devteaminc.co_

We've been using Sagepay, previously known as Protx, as the primary PSP within Liquidshop for a long time. We've had a solid integration for a long time too, but all based on completely bespoke code. With some recent updates to Sagepay it's become time to update this integratation to make use of some newer features.

I was aware that there was an official Sagepay integration kit for PHP. As with all new PHP jobs these days, the first place I looked was [Packagist][packagist-link]. Looking around there was no official SDK listed, there were a few other options though. Firstly there was an Omnipay package for Sagepay. I'd been following this library for a while. It's built and maintained by the PHP League and has some solid devs behind it. However, it has only recently updated to Sagepay Protocol Version 3 and doesn't have full support of the features we required. Namely, VSP Form and Server iFrame integration for more integrated payment pages. It would have been great to contribute to this package but commercial restraints prevented this.

Having looked at the official integration kit I could see it wasn't compatible with Composer. On packagist there was a listed Sagepay package with Composer support https://github.com/alvsystems/sagepay-php-sdk. This wasn't an official package though, and whilst i'm sure there's nothing dodgy happening here I wasn't keen on implementing unofficial 3rd Party code for this.

This left me with the official, non-Composer compatible code. All 3rd party code within Liquidshop is contained within the /vendor directory as it's all managed by Composer. The approach of downloading a 3rd-party library and dumping it somewhere into our core application code felt dirty. I may aswell have been installing something from phpclasses - eww...

This got me thinking about approaches for loading non-composer compatible code, or code not found on Packagist. A quick look around and I was reminded about the concept of respositories within a composer.json. The Sagepay code is available as a .zip file download, this could therefore be managed as a package within our Composer file. Perfect. As shown on the Composer site, a package definition can be as simple as:

```
{
    "name": "smarty/smarty",
    "version": "3.1.7",
    "dist": {
        "url": "http://www.smarty.net/files/Smarty-3.1.7.zip",
        "type": "zip"
    }
}
```

We could certainly recreate this with our Sagepay .zip file. By defining respositories with a package, we could bring in the Sagepay code and keep it within vendor. This prevents pollution of our core code, and keeps 3rd party self-contained.

On a first workthrough this is what I ended up with:
```
"repositories": [
  {
    "type": "package",
    "package": {
      "name": "sagepay/sagepay",
      "version": "3.0",
      "dist": {
        "url": "http://www.sagepay.co.uk/file/9981/download-document/VspPHPKit.zip",
        "type": "zip"
      },
      "autoload": {
        "files": ["lib/sagepay.php"]
      }
    }
  }
]
```

There's a few things here i'm not sure about. I've created a package name here of `sagepay/sagepay`. This is somewhat arbitrary but makes sense to me. This then allows me to add the package to my list of requirements in my `composer.json` so I can install, and update using the Composer CLI:

```
"require": {
    "sagepay/sagepay": "*"
    // etc etc
},
```  

The version number I have used reflects the protocol version this code is to use. As far as I can tell the code itself is not versioned in this way, and there is no reference to them adhering to Semver. I assume the version is used for the managing of updates so this is something i'll have to look at more. The other questionable element is the autoload section. The Sagepay code itself contains it's own spl autoloader which is launched by lib/sagepay.php. This file also creates a PHP Constant which is used to restrict access to the other code within their kit. For this reason i'm just using the autoload files option to load this single file, which in turn autoloads all that is needed. Effectively I now have 2 autoloaders being used but i'm comfortable that the positives outweigh the negatives here.

By now running composer update `sagepay/sagepay` I can now pull in this code direct from the Sagepay server into my vendor directory. From here i'm now good to get on with completing the integration, having used a non-composer compatible library within Composer. Pretty awesome.

[packagist-link]: https://packagist.org/
