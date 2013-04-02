---
layout: post
title: "PHPUnit PITA"
---

Well not PHPUnit itself of course. Sebastian Bergmann has developed a wonderful tool after all. Rather the environment of tools around PHPUnit seem overly bothersome. PHPunit, Code Sniffer, phpmd. phploc and Code Coverage are all great tools, but trouble arises if you try to install them using PEAR.

It is, if you don't already know, an apt-get like tool for php to install packages. Apt-get relies on repositories, which guarantee coherence of the contained packages, and gives you all kinds of neat features to not accidently destroy your packages, but still make upgrading or deinstallation of packages hassle free. PEAR has none of that. Instead every package owner is encouraged to run their own repository, which sounds nice in theory because sharing is made easier, in practice though it hardly ever works as advertised.

I tried upgrading to the most recent PHPUnit using PEAR. The upgrade process forced me to upgrade a bunch of other packages first (which was okay). But some packages were now newer than PHPUnit relied on. So I had to deinstall those and all dependency and reinstall the packages in the correct version and order. Finally done with all that I looked forward to using the new PHPUnit. So I fired it up and was greeted by an error. Seems PHPUnit depends on a package version which is in none of the repositiries yet.

Luckily I had backed up my PEAR folder before the upgrade, otherwise I would've had to replace a keyboard and a few monitors by now.

What is my point anyway?

My point is this. Sharing your packages with PEAR is great, but the current installer lacks a bunch of features.

What if you need more than one version of a package? Why does installing the latest version of something not always work? Why isn't there a rollback option?

There should be a pear server, which one can safely install from. It does not necessarily have to host the files itself. It could just point at the correct versions. Not necessarily bleeding edge, but stable.

There should be a rollback option. A way to back up the version numbers and channels of the installed packages. If something breaks you just rollback. Something like

	pear set-rollback // creates a new rollback xml file with the current time and information about installed packages
	pear rollback last// resets all pear packages to the way they were at the last rollback point.

The installer should have a install as standalone phar option.