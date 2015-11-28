+++
title = "Common questions when using composer with TYPO3 extensions"
subtitle = "On how to develop TYPO3 CMS extensions with composer"
tagline = "On how to develop TYPO3 CMS extensions with composer"
author = "Cedric Ziel"
template = "post.html"
keywords = "Cedric Ziel"
changefreq = "monthly"
priority = "0.9"
showdownExtensions = ['github', 'table', 'math', 'smartypants', 'footnotes']
date = "2015-11-26T21:58:29+01:00"
draft =  false

+++

Fortunately [Composer](https://getcomposer.org/) is becoming the central piece in every TYPO3 CMS installation beginning
  with the final release of [TYPO3 CMS 7.6 LTS](https://typo3.org/news/article/announcing-typo3-cms-7-lts/). But what does
  it mean for the developers that are not used to using composer? Answers to a continous flow of questions. I will
  add them here, once i see them arise. I also plan an article about why we need composer at all. But that's for another
  day.
  
## 1. Why does composer download all of TYPO3 CMS core to my extension directory?

You develop an extension and you need to declare dependencies. you declare for example ``typo3/cms:~7.6.0`` as dependency
  to your extension as you depend on a specific version of the TYPO3 CMS core in your extension to work correctly.

**Please note:** This only happens in the development environment, when you composer-install in a production environment,
the TYPO3 core will be installed exactly one time in the top-level vendor directory. When you develop TYPO3 sites locally,
 you don't need the nested vendor directories, but they do not harm, as they do not get autoloaded when you execute the site.

Now when you install your dependencies on development with ``composer install``, you get all of TYPO3 CMS core in
  ``vendor/typo3/cms``. Why is that? Does it make sense? What if I have 10 extensions? Does it download TYPO3 CMS core 
  10 times?
  
**Composer will copy all of TYPO3 CMS core in Version ~7.6.0 to every extension you're developing and that's a good thing.**
Disk space is cheap, so that shouldnt worry you. - But only with this local vendored dependencies 
(read more on vendoring here: [Golang 1.5 vendoring experiment](https://docs.google.com/document/d/1Bz5-UB7g2uPBdOx-rw5t9MxJwkfpx90cqG9AFL0JAYo/edit)),
your project is complete and independently usable from a full-fledged TYPO3 CMS project. For example in your continuous 
integration / build environment you don't want to set up a website just to test, if your classes are formally correct 
(which also means checking interface contracts etc), unit tests and so on.

Other projects do so called ``subtree splits`` with the Git version control system to split sub-packages out of the main
source tree into distinct, read-only subtrees, so developers can depend on packages and only get that particular package
plus its dependencies delivered, when requiring one.

Let's have a look at some concrete examples:

### TYPO3 CMS Composer configuration

TYPO3 CMS, in its current state, has both a [composer base distribution](https://packagist.org/packages/typo3/cms-base-distribution)
 (The thing you can ``composer create-project`` from), and a base [framework package](https://packagist.org/packages/typo3/cms).

The composer package ``typo3/cms`` has a bunch of dependencies and *impersonates* itself as a bunch of other
packages (in the ``replace`` section). Everytime you just depend on ``typo3/cms-fluid`` (fictitious example),
you get the complete core package, as this particular package can only be found in the big package.

```json
// parts from composer.json in TYPO3 CMS 7.6 
// (https://github.com/TYPO3/TYPO3.CMS/blob/TYPO3_7-6-0/composer.json)
{
	"name": "typo3/cms",
	"description": "TYPO3 CMS is a free open source Content Management Framework initially created by Kasper Skaarhoj and licensed under GNU/GPL.",
	"type": "typo3-cms-core",
	"require": {
		"php": ">=5.5.0",
		"swiftmailer/swiftmailer": "~5.4.1",
		"symfony/console": "~2.7.0",
		"symfony/finder": "~2.7.0",
		"doctrine/instantiator": "~1.0.4",
		"typo3/class-alias-loader": "^1.0",
		"typo3/cms-composer-installers": "^1.2.2",
	},
	"require-dev": {
		"phpunit/phpunit": "~4.8.0",
		"mikey179/vfsStream": "1.6.0"
	},
	"replace": {
		"typo3/cms-core": "self.version",
		"typo3/cms-documentation": "self.version",
		"typo3/cms-extbase": "self.version",
		"typo3/cms-fluid": "self.version",
	}
}
```

* [base distribution package code](https://git.typo3.org/TYPO3CMS/Distributions/Base.git/tree)
* [framework package code](https://github.com/TYPO3/TYPO3.CMS)

### Laravel Web Framework

[Laravel](http://laravel.com/) is a Rails - inspired PHP Framework and has both a framework package, and a distribution / base
package. At night, all the components are split out from the main source tree into a new namespace (``illuminate``) to allow
installation of distinct components which otherwise would install via the framework package. Have a look at the 
[illuminate](https://github.com/illuminate) GitHub Organization to take a look at the read-only repositories there. 

* [base distribution package code](https://github.com/laravel/laravel)
* [framework package code](https://github.com/laravel/framework/blob/5.1/composer.json)

### Symfony 2 Web Framework

The Symfony community were amongst the first to adapt composer in their concept of dependency management. They drive a
very similar subtree split approach like Laravel.

### Can we fix this?

By now, you can only obey to the environment you're developing in and install all of the TYPO3 CMS package if you depend on
a specific component. Technically it's the fault of this particular package that it doesn't allow for using sub-packages.

This issue will hopefully become a pressing one when more and more developers use TYPO3 CMS in composer mode.

## 2. Should I use composer if it brings disadvantages?
 
**Hell yes.**

I call those disadvantages from above "trade-offs". You trade in a bit of free will for a managed dependency tree and 
can now use and manage foreign libraries like grown-ups.

This allows for automagically smooth automated deployments and very isolated build- and testing-environments. 
If you want one thing - it's this one.

## 3. What about the integrators, do they have to know?

Difficult question. For now the CMS team provides a compiled package (the precious ``typo3_src`` folder) which can ship
you around needing to know about composer as an integrator.

TYPO3 CMS being an _enterprise content management system_ (You know. Enterprises have technically skilled people and money.) means
a little more to me. In my humble opinion the so called _Integrators_ are not the ones to technically set projects up initially.
This is done by technically skilled people who manage the environment and know the dependencies. Developers.

Integrators should be the ones being pointed at an already set-up installation and set it up on the _configuration level_.

I know, the reality is different. Integrators claim to know the system inside out. But I think this claim also means they 
have to get comfortable with composer or step away from the _technical stuff_.

Fortunately for integrators, the CMS team provided the composer mode which can be turned off.

## More questions please

If you want some clarification on one or the other point post a comment or nag me on [Twitter](https://twitter.com/cedricziel).
