+++
title =  "Use composer to quickly initialize your TYPO3 projects"
subtitle =  "Install TYPO3 with composer in a breeze"
tagline =  "<em>Use composer</em> to easily get yourself some TYPO3."
author =  "Cedric Ziel"
template =  "post.html"
keywords =  "Cedric Ziel"
changefreq =  "monthly"
priority =  "0.9"
date = "2013-09-01T21:58:29+01:00"
draft =  false

+++

What's probably the most annoying task when starting a new project, may be the initialization of your new project structure.
This usually involves one or more of the following Tasks:

- Downloading the TYPO3 sources
- Unpack them to a local webserver
- Configure the base parameters (Database, ImageMagick, cURL..)
- Initialize the extension list via Extension Manager (which failed often on newer 6.x branches)
- Import the extensions you need, in the version you need

Attempts to solve it
--------------------

While you can attempt to template projects (and you should really do that!), this could also lead to some yummy new trouble:

- outdated dependencies
- outdated core
- big blobs of sql (or in our case, t3d files)

Composer
--------

While [composer][1] may be not the most common way to manage dependencies in the TYPO3, it could be a real advantage, using it. Composer manages dependencies with cascading versions, suggestions and exclusions as well as platform requirements such as enabled PHP extensions and versions.

To get a quick overview, of what can be done, you can execute the following lines in your cli:

```bash
curl -sS https://getcomposer.org/installer | php
php composer.phar create-project --stability=dev ft3/fluxkitchen-distribution .
```

This will fetch a project template from a [GitHub][fluxkitchen-distribution] project of mine and installs a chain of dependencies. But first, let's examine the [composer.json](https://github.com/cedricziel/fluxkitchen-distribution/blob/master/composer.json) (I exluded the profane parts..):

```json
{
	// Defines the requirements
	"require": {
		// Thanks to the Lightwerk guys, we can define TYPO3 CMS dependencies via composer
		"typo3/cms": "6.1.x",
		// Here, a custom package is required, which has its own dependencies
		"ft3/fluxkitchen": "dev-master"
	},
	"repositories": [
		// Lightwerk composer repository
		{
			"type": "composer",
			"url": "http://composer.lightwerk.com/"
		},
		// The composer package of the FluidTYPO3 project, generated with Satis
		{
			"type": "composer",
			"url": "http://fluidtypo3.github.io/composer-repo/"
		},
		// You can even define your own packages on the fly, great for private packages
		{
			"type": "vcs",
			"url": "https://github.com/cedricziel/fluxkitchen"
		}
	]
}
```

What will it do when issueing the command?

- Install composer.phar to your local workspace
- Clone the template project to your local workspace
- run ``composer install`` to resolve dependencies, recursively (!)

The [fluxkitchen][] repository itself defines no functionality, but shows the great possibilities. Its composer dependencies are defined as follows:

```json
"require": {
	// We need the composer installers!
	"composer/installers": "*",
	"ft3/fluidcontent": "*",
	"ft3/fluidpages": "*",
	"ft3/ft3_empty": "*"
},
```

Recursive behaviour at its best. :-) Any of the above listed packages (except ``composer/installers``) has a strong dependency to the ``ft3/flux`` package, which will be resolved dynamically.

Result
------

With this single line of command, you get:

- a current TYPO3 core (``"typo3/cms": "6.1.x",``)
- a set of extensions (**most current versions**):
	- ft3_empty
	- flux
	- fluidpages
	- fluidcontent
	- fluxkitchen

What else can composer do for us?
---------------------------------

Admittedly-we didnt reach all of our goals defined above, but I will keep fiddeling around with [composer postInstall commands][].
That way, one could:

- fix permissions on a regular base (on each ``composer update for example``)
- a post-install wizard to configure the database (install tool api?)
- install extensions (not just downloading them) via Core API
- ...

Outlook
-------

Let's recall the list once again:

- ~~Downloading the TYPO3 sources~~
- ~~Unpack them to a local webserver~~
- Configure the base parameters (Database, ImageMagick, cURL..)
- ~~Initialize the extension list via Extension Manager (which failed often on newer 6.x branches)~~
  **partially solved. _All_ extensions defined a depenencies are downloaded, which circumvents the need for it ;)**
- ~~Import the extensions you need, in the version you need~~

Given you used the *Provider Extension* pattern, you could have a running TYPO3 installation with **0 overhead** in minutes.

I hope you like the approach-I will constantly try to evolve it and link follow-ups. What are your approaches to template projects?

Links
-----

* [Composer][1]
* [fluxkitchen-distribution Github Project][fluxkitchen-distribution]
* [fluxkitchen Github Project][fluxkitchen]
* [Composer Post Install][composer postInstall commands]

[1]: http://getcomposer.org
[fluxkitchen-distribution]: https://github.com/cedricziel/fluxkitchen-distribution "Github Project"
[fluxkitchen]: https://github.com/cedricziel/fluxkitchen "fluxkitchen Github Project"
[composer postInstall commands]: http://getcomposer.org/doc/articles/scripts.md#event-names "Composer Post Install"
