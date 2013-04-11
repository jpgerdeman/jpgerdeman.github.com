Thanks But it's Garbage
=======================

Starting from a simple "Hello World" we made our way to a nice little framework. However it's full of security holes. It's also missing stuff that is expected of a framework. Also the way we developed it encouraged a few bad architectural decisions.

It's not safe from XSS attacks nor from SQL Injection. We never talked about asset management. We're using mysqli instead of PDO. We only partially abstracted the request. The routing is a mess. Every URL still contains `index.php`. And so on.

Was it all for naught?

No. True we ended up with a framework worth scrap. On the other hand we touched a lot of points which are important in modern software development with PHP. I guess we haven't even touched a fifth of the relevant concepts.

Software developement is about constant learning. It is a ever changing field. Read books. Learn other programming languages. Learn stuff not directly related to programming.

This course wasn't meant to be end. It was a quick tour around town. You will have to make yourself aquainted with the differnt neighbourhoods and citizens. 

So much for the tour. Hope you enjoyed it and will come back soon.

Reading List
============

What should you read next?

## Books
The absolutely necessary must read list. There are many more books, but these you must read 

  - [The Pragmatic Programmer: From Journeyman to Master by Andrew Hunt and David Thomas](http://pragprog.com/book/tpp/the-pragmatic-programmer)
- [Clean Code: A Handbook of Agile Software Craftsmanship](http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)
- [Refactoring Improving the Design of Existing Code by Martin Fowler](http://martinfowler.com/books/refactoring.html)
- [Design Patterns: Elements of Reusable Object-Oriented Software by Erich Gamma, Ralph Johnson, John Vlissides, Richard Helm](http://www.amazon.com/gp/product/0201633612)
- [Seven Languages in Seven Weeks: A Pragmatic Guide to Learning Programming Languages by Bruce A. Tate](http://pragprog.com/book/btlang/seven-languages-in-seven-weeks)


## Blogs and Sites
  - [Rasmus Lerdorf](http://toys.lerdorf.com/) is the original author of PHP.
  - [PHP Developer](http://phpdeveloper.org/) (Nice Agregation of a few Blogs)
  - [Paul Irish](http://paulirish.com/) - The front end overlord  
  - [Fabien Potencier](http://fabien.potencier.org/) has mostly stuff about the symfony framework. Some posts do concern a wider audience and usually have a huge impact on the PHP community.
  - [Nikita Popov](http://nikic.github.io/) is a PHP core developer, mostly concerned about improving core itself. Has a great introduction series to PHP core.
  - [Lorna Jane's](http://www.lornajane.net/posts/category/9-php) posts seem to be oriented at new developers at first. Her posts usually reveal a bit about PHP's surprising odds and ends.
  - [Pierre Joye](https://twitter.com/PierreJoye) is also a PHP core developer
  - [Anthony Ferrara](http://blog.ircmaxell.com/) has a series with introductory videos that start about where we left of. His main concern is security and the corresponding blog posts a must read.

And many more. I suggest to follow them on twitter. Granted there will be some extra noise, but these people talk and sometimes that can be very insightful. They will also promote other blogposts as tweets which they not do on their blogs.
## Mailing Lists
Mailing lists you must follow? Every framework, cms and library has their own mailing list. Subscribe to those that are relevant to you. The following mailing lists have global impact on the PHP world.

  - [PHP fig](http://www.php-fig.org/)
  - [PHP Internals Mailing List](http://marc.info/?l=php-internals)
