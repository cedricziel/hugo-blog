+++
title =  "Packaging composer libraries for TYPO3 CMS (v6)"
subtitle =  "Using the new stuff with the old stuff"
tagline =  "Using the new stuff with the old stuff"
author =  "Cedric Ziel"
template =  "post.html"
keywords =  "Cedric Ziel"
changefreq =  "monthly"
priority =  "0.9"
showdownExtensions =  ['github', 'table', 'math', 'smartypants', 'footnotes']
date = "2015-09-24T21:58:29+01:00"
draft =  false

+++

> Note: I published this as a [gist ](https://gist.githubusercontent.com/cedricziel/08ea469db5197e24552c/raw/d65b719af3c163e8e9ebec6effa0848788f2610d/README.md) some time ago

As long as composer support in CMS is "not there yet", you need to get around somehow. Or maybe you need to maintain a
old site and need to use new libraries.

Say you want to use the (awesome) markdown library, you need a way to get it in.

## How

1. Use a container extension with a private namespace
2. Create composer.json
3. Include the autoloader for both contexts
5. profit

## Why a separate composer.json?

Each extension gets a root `composer.json` sooner or later - but this one is irrelevant for now as we dont want to rely on the standard TYPO3 CMS classloader, but rather use the functionality offered by composer to create an autoloader for us.

For now, the package manager also awaited (TYPO3) packages to exist for every library you require.

## Code

Note: You have to create all this in an extension. I put emphasis on the point that it should be a separate extension.

### composer.json

Create a file `composer.json` in your container extension in `Resources/Private` (you can use `composer init`):

```json
{
	"name": "cedricziel/packagename",
	"config": {
	  "vendor-dir": "Libraries"
	},
	"require": {
		"michelf/php-markdown": "~1.4"
	},
	"authors": [
		{
			"name": "Cedric Ziel",
			"email": "cedric @t cedric-ziel.com"
		}
	]
}
```

Note the `vendor-dir` directive. This allows you to determine where composer will put the libs and the class loader you have to include.

### ext_tables.php && ext_localconf.php

I cant explain the exact sematics very well, but you will want to have both. Otherwise your classes wont load in a cached context.


`ext_tables.php`
```php
<?php
if (!defined('TYPO3_MODE')) {
	die('Access denied.');
}

$composerAutoloadFile = \TYPO3\CMS\Core\Utility\ExtensionManagementUtility::extPath($_EXTKEY)
	. 'Resources/Private/Libraries/autoload.php';

require_once($composerAutoloadFile);
```

`ext_localconf.php`
```php
<?php
if (!defined('TYPO3_MODE')) {
	die('Access denied.');
}

$composerAutoloadFile = \TYPO3\CMS\Core\Utility\ExtensionManagementUtility::extPath($_EXTKEY)
	. 'Resources/Private/Libraries/autoload.php';

require_once($composerAutoloadFile);
```

## Notes

This is a **workaround** - let me get this straight. Problems could arise if you do that overly often with many different libraries and end up with different loadable versions. I therefore propose you creat **one** such container extension per project.

The original post as a [gist ](https://gist.githubusercontent.com/cedricziel/08ea469db5197e24552c/raw/d65b719af3c163e8e9ebec6effa0848788f2610d/README.md)
