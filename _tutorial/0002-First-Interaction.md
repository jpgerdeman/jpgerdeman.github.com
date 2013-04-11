First Interaction
=================

A simple "Hello World!" is rather boring. Let's change that to greet the someone, instead. We'll get the name from the queryparameter part of the URL. Our URL will look something like

	http://localhost/fhwtf/step0002/index.php?name=john

To get the name parameter from the URL use

	$name = $_GET['name'];

The `$_GET` variable is created by PHP and contains the parameters from a GET request. Likewise `$_POST` contains the parameters from a POST request. 

The difference between a POST and GET request is that a GET request transports its information in the URL, while a POST carries its information in the body. Since the length of the URL has a length restriction it means, that the information it can carry is restricted as well. Also the URL can't be encrypted. A POST request has none of these limitations. On the other hand it is simpler to create a GET request, since that is what every Browser does.

It's important to know the difference and know when it is appropriate to use which.
## RFC 3986

[RFC 3986](http://tools.ietf.org/html/rfc3986) defines the resource locators and is worth reading. I'd suggest reading it after you're comfortable eough in PHP.

For now we'll concentrate on the parts of the http-Schema, that are most relevant, when talking about URLs.

*Again this only covers the most important bits and ommits a few things.*

	schema://doma.in/my/path?queryparam1=1&queryparam2=2

So a URL is defined by a schema, domain, path and a query. The query passes two parameter `queryparam1` and `queryparam2` and is seperated from the path by the `?`-Symbol. Multiple Parameters are attached via `&`.


## A word on ?>

With the closing script-tag `?>` you indicate the end of your PHP code. If your file ends with PHP code (because it only contains PHP code) you can omit the closing script-tag and PHP will implicitly assume it at the end of the file. This is very convenient, but discouraged by a (very) few people.

Adding the closing script-tag on the other hand requires more diligence from the developer. Everything after the closing tag is interpreted as HTML, including white-space. Which can lead to inexplicable white-space in your output. This is especially annoying when your output is something other than HTML. For example an empty line in front of an XML file will cause any consuming program to choke on it, since the XML declaration *must* be the first line.

I omit the closing script-tag, as most others do.