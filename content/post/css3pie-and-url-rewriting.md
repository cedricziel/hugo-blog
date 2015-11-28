+++
title =  "Using CSS3Pie with URL rewrites"
subtitle =  "You never know, when rewriting hits you in the .."
tagline =  "Using CSS3Pie with URL rewrites can produce you troubles you never had.."
author =  "Cedric Ziel"
template =  "post.html"
keywords =  "CSS3 CSS3Pie OldIE Apache NGINX polyfill"
changefreq =  "monthly"
priority =  "0.9"
date = "2014-01-04T21:58:29+01:00"
draft =  false

+++

So you created this nice and shiny new hippster round-edges site for your client and allways press your thumbs
on deployment to make it magically work on the production system. But what if we deploy to a different Webserver
software?

Now you might rapidly approach troubles and pain as you never had before, winding your brain about why the crap your
CSS3 Polyfill accomplished with [CSS3PIE](http://css3pie.com 'CSS3Pie Polyfill') doesn't work as expected (and tested on
your dev-site). This can cause a serious case of grey hair!

## Your stylesheet

```css
/**
* Bootstrap 3 generic conditional stylesheet
* taken from https://gist.github.com/coliff/5618329
*/
.img-rounded{behavior:url(/scripts/PIE.htc)}
.img-circle{behavior:url(/scripts/PIE.htc)}
.table-bordered{behavior:url(/scripts/PIE.htc)}
select,textarea,input,code,pre{behavior:url(/scripts/PIE.htc)}
.input-group-addon{behavior:url(/scripts/PIE.htc)}
.btn{behavior:url(/scripts/PIE.htc)}
.dropdown-menu{behavior:url(/scripts/PIE.htc)}
.panel{behavior:url(/scripts/PIE.htc)}
.well{behavior:url(/scripts/PIE.htc)}
.nav-tabs > li > a{behavior:url(/scripts/PIE.htc)}
.nav-pills > li > a{behavior:url(/scripts/PIE.htc)}
.navbar{behavior:url(/scripts/PIE.htc)}
.navbar-nav > li > a{behavior:url(/scripts/PIE.htc)}
.navbar-toggle{behavior:url(/scripts/PIE.htc)}
.navbar-toggle .icon-bar{behavior:url(/scripts/PIE.htc)}
.breadcrumb{behavior:url(/scripts/PIE.htc)}
.pagination{behavior:url(/scripts/PIE.htc)}
.pager li > a,.pager li > span{behavior:url(/scripts/PIE.htc)}
.modal-content{behavior:url(/scripts/PIE.htc)}
.tooltip-inner{behavior:url(/scripts/PIE.htc)}
.popover{behavior:url(/scripts/PIE.htc)}
.popover-title{behavior:url(/scripts/PIE.htc)}
.alert{behavior:url(/scripts/PIE.htc)}
.thumbnail,.img-thumbnail{behavior:url(/scripts/PIE.htc)}
.label{behavior:url(/scripts/PIE.htc)}
.badge{behavior:url(/scripts/PIE.htc)}
.progress{behavior:url(/scripts/PIE.htc)}
.accordian-group{behavior:url(/scripts/PIE.htc)}
.carousel-indicators li{behavior:url(/scripts/PIE.htc)}
.jumbotron{behavior:url(/scripts/PIE.htc)}
```

Pretty neat, huh? [WorksOnMyMachine](http://worksonmymachine.org)! Freaks out in production.

## The hunt

So you did everything right, the SauceLabs connect tool is d'accord with it, but there's something fishy..

### Checklist

- .htc files uploaded? **CHECK**
- Webserver delivers correct mimetype? **CHECK**
- Works as expected? **NOPE**

### Correct Checklist

- .htc files uploaded? **CHECK**
- Webserver delivers correct mimetype? **CHECK**
- **Make sure the webserver doesn't redirect you, whilst requesting the .htc file**
- Works as expected? **SURE**

## Conclusion

Never let your IE friends be redirected while requesting the .htc file. The server **has** to return a ``200`` response
code.
