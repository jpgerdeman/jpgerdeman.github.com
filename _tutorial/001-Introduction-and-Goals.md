Introduction
============

This course will take you from a simple beginning to a full framework.

<blockquote>
Every PHP developer should create at least one framework.
</blockquote>

Writing your own framework is a learning experience. When writing a framework you will touch points you have never thought of before. Creating a framework also means you have to focus more on the change of perspective from writing to using it. Writing a framework means to make mistakes. Many mistakes. This in turn means there is a lot to learn.

**Do not use the resulting framework of this course in production.**

However just because you can write a framework and - at the end of this series - did, does not mean you should use it in production. I hope that by the end of this series you can acknowledge how much thought and work has been put in popular frameworks like Laravel, Symfony, ZF2 and the countless others. Frameworks that had years to mature, years of testing and years of security enhancements.

The framework we'll be creating is not meant for use in production. It's a study example to dissect and play with. Especially where security is concerned the framework will lack.

## Goals

We want to create a framework with the following goals:

 - easy to use
 - easy to extend without breaking backwards compatibility
 - support for several databases
 
We'll write a sample application in our framework to check that it works and how easy it is to use. This application should allow us to greet the user, display his profile and to add new users.


## Let's Begin
Our Journey will begin, as so many others have as well, with a greeting.

	<?php echo "Hello World!"; ?>