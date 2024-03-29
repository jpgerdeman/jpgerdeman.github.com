Software Artists do it with style
---
Aside from features like automatic code generation that would make our orm 
really userfriendly, we actually are missing something that renders our whole 
code useless.

##Commenting our code
Our function are short and sweet you say? No need to comment them? Wrong!
How will a new user know how to use a function, class or method? Think of our orm. A user would need to read the whole thing before he understood how it is intended to be used; if he understands it at all.

Think like a consumer of your code, what would you need to get going asap. Like it or not if you look back at your code in six months or a year you won't remember what it did. If everything works as expected and no errors occur your code might run for years before you need to touch it again. Think like a consumer now, because most likely you'll be that consumer. So do yourself a favor.

There is no official documenting guideline for PHP code. So you can do it however you want, this is PHP after all. The de facto standard is a style of comments borrowed from Java. The reference to these can be found on phpdocumentor. 

A basic comment is build as follows.

    /**
     * A one line description of the following method/class or file.
     *
     * A long description explaining details and whatnot. It does not
     * explain the internals of the function. You can use <b>HTML</b>
     * in your comments. Most IDEs will evaluate these comments and 
     * display them nicely for you, when you try to call a function.
     *
     * Markdown can be used instead of HTML. Some documentation 
     * generators support this. Sadly no IDE does at the moment.
     *
     * @link daringfireball
     *
     * @param string $name The username of the user to retrieve
     *
     * @return User an instance of User
     */
    public function retrieve( $name )
    {
        ...
    }

Comments on classes and files of course don't have parameters and return values - unless the class implements `__invoke()` in which case it would make sense. So your file should be commented, this can contain your name, email, license and hints for the documentation in which context this file belongs. Classes should be commented as well and explain not only the class, but the intent of usage. You could for example mention, that our user decorators have to be set as prototype for the factory. In the method description you document parameters and return values, if you think it's useful you also supply code example.

## Documentation Hidden in Plain Sight

Remember however comments are also code, not executed by PHP, but by consumers. Code has to be maintained. Keep the comments short, but don't make them useless. Help a consumer, don't bore him.

The comments don't stop there. The names you assign are also comments. They are the shortest description of your code. Naming things is the hardest thing in programming, comments come right after. Look at `retrieve` method of the UserFactory. From that I can already assume, that retrieve will return some kind of User object. It does not tell me how I retrieve it. If it stays the only retrieve function in the UserFactory I could leave it. Personally I'd prefer calling it `retrieveByUsername`or at least change `$name` to `$username`. The method signature then already tells me alot of what I have to know. 

There are four ways to start a comment in PHP. We've seen the first `/**` then there is `/*` which to PHP is actually the same, but some document generation tools treat them differently so I shall too. These are commonly used for comments, that should be included in the documentation. Then there are `//`and `#` which are usually ignored by documentation tools. Most often they are used by developers to comment the following line of code.

    function foo( )
    {
        ...
        # Not obvious, but important otherwise BOOM
        $n = $n * 1.001;
        ...
    }

You'll often hear people saying these types of comments are code smells. If you need to comment it, then it is either a constant your missing, or badly named variables, or a method needing extraction. That's true, but a helpful comment is always better than nothing. Maybe you just didn't like the alternatives. That too is a valid reason.

When writing code take your time to comment. It's a good habit to have. Also take time to refactor your code after you've written it. And separate it while you write it. Take time to make your code expressive.

The code above I could've written as

    /** @const The prime constant is the value of ... */
    define( 'PRIME_CONSTANT', 1.001);

    function foo()
    {
        ...
        $n = $n * PRIME_CONSTANT;
        ...
    }

Maybe that change would've sufficed to be rid of that inline comment, while supplying more and better information.

I often start writing code like this, especially if there is a lot to do.

    function doFoo()
    {
        #retrieve User
        #retrieve Book
        #assign Book to User
    }

Most times these comments translate directly to method names.

    function doFoo()
    {
        $user = $userfactory->retrieveByUsername('Leeloo');
        $book = $bookfactory->retrieveByISBN('1234567890123');
        $user->addBook( $book );
        $user->save();
    }

Without knowing how those methods work, you pretty much know what is going on. Also note, that we didn't have any low level DB stuff here - we kept one level of abstraction.

## Expectations

Code is also a lot about expectation. Decide on a style of writing code and be consistent about it. I'm not gonna tell you how to format your code. You should decide that yourself and stick to it. 

Small things can also help making consumers more comfortable with your code. Again this is PHP so there are no hard rules, but some standards have emerged. Learn those standards and make a conscious decision for or against them.

 <blockquote> Classes are Nouns, Methods are verbs.</blockquote>

This means classes nouns and that they should start with a capital letter. Methods start with a small letter and express an action.
 
  <blockquote> Constants are written in all uppercase with underscores. </blockquote> 

This is self explanatory `PRIME_CONSTANT` should be a constant `Prime_Constant` shouldn't

 <blockquote> The name should express the cost of the method  </blockquote> 

Well a method, that only returns a ready value or only does simple operations on it should start with `get` or another verb that is associated with simpleness. If heavy lifting is involved it should start with a costly verb like `compute` or `generate`.

 <blockquote>Names should be consisitent.</blockquote>

If two methods do something similar, their names should be similar too. If you start naming your methods to get a object from the database `retrieve*` you shouldn't start calling them `fetch*` later.

