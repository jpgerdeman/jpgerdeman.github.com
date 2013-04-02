---
layout: post
title: "CSS Coding Conventions"
---
My colleagues and I were arguing yesterday about our CSS. Unlike in other languages, we do not have a coding standard for CSS. Up to know we’ve tried to adhere to the following loose rules:

 - Respect the order found. – This simply means that your code should fit in with the style of the old code.
 - Boy scout rule: "Always leave the grounds cleaner than you found it."
 - DRY: “Don’t repeat yourself.”

These simple rules suffice during development when everyone (or at least part of us) has the implicit rules (imposed through the first CSS code) in short term memory. When going into maintenance or adding a feature later on, the implicit rules have been forgotten or there is little time to figure them out. More CSS rules are added sometimes duplicating older rules. Even worse is when you can’t figure out what the older rules are good for (i.e.: a forgotten abbreviation).

I’ve started compiling my personal CSS coding styles into a comprehensive list and would like to know what you guys think about it.

 - Don’t use abbreviations: Take your time to spell out what you mean. If the abbreviation is well known (outside of your project) you may use it (i.e.: html). If the use of an abbreviation is useful and consistently used throughout your project, document it in the file header.
 - Use lower case dash notation instead of camelcase or underscore. Since we use a lot of jQuery this feels the most natural. Also CSS – in some Browsers – is case sensitive so all lower case selectors will prevent a lot of typos.
 - The selector name should reflect the semantics not the layout. A selector name like “grey-box” is bad! “notification-box” tells me more. Also it is not as ambiguous as “note-box”, which could be a box, where someone takes notes.
 - The selector should reflect the structure, at least a little. If you have a “logo” selector it might be better to call it “header-logo” especially if you have more than one.
 - Don’t use CSS shorthands. Put every declaration on its own line.
 - Put every selector on its own line.
 - Curly braces open on the same line as the last selector and close on a newline.
 - Every declaration should end with a semicolon.
 - Tabs are used for indentation.
 - In a declaration there is no space before a colon. There is exactly one after the colon.

List of useful posts:

 - http://www.phpied.com/css-coding-conventions/
 - http://developer.fellowshipone.com/patterns/code.php
 - http://codex.wordpress.org/CSS_Coding_Standards