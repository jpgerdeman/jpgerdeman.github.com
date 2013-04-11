Crell earlier today [tweeted](https://twitter.com/Crell/status/317387225430114305) about expectations and return values of functions. The tweet sparked a lively discussion, so he decided to write a [blog post](http://www.garfieldtech.com/blog/empty-return-values) about it.

If you read Crell's blog regularly this new post feels a bit like a letdown. It reads more like a rant, than a well researched post. So i decided to write something myself.

## What Do Functions Return
We'll start at a different corner of the story and see what common PHP functions do return.

### Returncodes vs. Exceptions
You will often hear people saying something along the lines 

<blockquote>
Return codes are evil and superfluous. They can all be replaced by throwing exceptions.
</blockquote>

Another school of thought runs something like this.

<blockquote>
Use return values to document the exit status of your function. They don't force a function to have several exit points. They cannot take down your system if you forget to check.
</blockquote>

At the end of the day it boils down to use common sense and doing your error checking. Exceptions and error codes can both be used for error handling. If you read enough discussions on the point, you'll notice that they usually all end in the same way; evangalists proclaiming their method as superior and the masses acknowledging that it is mostly a matter of taste.

So is that all? It's a matter of taste and that's it?

There is more. Consider the following function

    /**
     * Returns a foo.
     *
     * Error codes:
     *
     * 2 - Database could not be found
     *
     * @return Foo|int|null returns the found foo, an 
     *    integer error code or null, if foo wasn't found.
     */
     function retrieveFoo(){..}

    $foo = retrieveFoo();
    
    if( is_int($foo) && $foo == 2 )
    {        
        // load config and try again
    }
    elseif( $foo )
    {
        echo $foo->bar();
    }
    else
    {
        echo "Foo was not found"!
    }

The return value can now convey three types of information: the expected object, error information or nothing. Is that easy for a consumer of your code? 

So even if return codes vs. exceptions is a matter of taste, it shouldn't be your taste that decides. Read your code as if you're trying to understand and use it. Compare the above to the following

    /**
     * Returns a foo.
     *
     * @return Foo|null returns the found foo or null, if foo wasn't found.
     *
     * @throws DatabaseNotFoundException
     */
     function retrieveFoo(){..}

    try
    {
        $foo = retrieveFoo();
    }
    catch( DatabaseNotFoundException $e )
    {
        // load config and try again
    }

    if( $foo )
    {
        echo $foo->bar();
    }
    else
    {
        echo "Foo was not found"!
    }

Even if the amount of lines to write and document your code does not vary between the two methods; even if the consumer has to write the same amount of lines; this second version is more easy to grasp. We've separated our exception logic from the evaluation of the returned value. 

Also we use keywords that indicate exception handling. A 1 second glance will tell you what the code is roughly doing. We made it easier to read our code.

However we can make our function even easier to use.

### Null Values
    
    /**
     * Returns a foo.
     *
     * @return Foo returns the found foo. If no foo is found a Null Oject 
     *     is returned.
     *
     * @throws DatabaseNotFoundException
     */
     function retrieveFoo(){..}


    try
    {
        $foo = retrieveFoo();
    }
    catch( DatabaseNotFoundException $e )
    {
        // load config and try again
    }

    echo $foo->bar();
    
With this small change we made it easier to grasp the comment and also decreased the amount of lines a consumer has to write. Because a user does not *have to* check if the function returned anything. The function always returns something or throws an exception.

Your function hardly changes, but you changed your interface to make it easier for users to grasp and consume. As usual your future self is going to be the one that needs to understand how to work with your code. So ask yourself, wich code would you rather work with.

Our Null Object is an empty object, that implements the interface `Foo`, but contains no data or logic. In the case of value objects, say a record from the database, it often suffices to instantiate a new empty class; that is if `Foo` is no interface but a concrete implementation that is hydrated with a record, you can just return a new instance as null object.

    function retrieveFoo()
    {
        // execute logic 
        if( !is_null($record) )
        {
            $f = new Foo($record);
        }
        else
        {
            $f = new Foo();
        }
        return $f;
    }

If your class contains special logic, you will almost always have to provide an alternative implementation. Imagine `Foo` is a log or cache implementation. In those cases you will have to provide an implementation that implements the same interface, but does not contain logic (or just enough to make the rest of the code accept your null object).

the following `NulCache` can be used wherever a `Cache` implementation is acceped, but does not cache any values. Code using it however runs none the wiser.

    class NullCache implements Cache
    {
        public function contains( $key )
        { 
            return false; 
        }

        public function set( $key, $value )
        {
            // do nothing
        }
    }

Just make sure users have the chance to check for a null object if they have to, by using a special class or method.

    if( $f instance of NullFoo )
    ...

or 

    if( empty( $f->baz() )
    ...

### Expectations

So functions should have exactly one return type and use exceptions to handle exceptional situations. Lets look at another situation.

    /**
     * Returns the found foos.
     *
     * @return Foo[]|Foo 
     *
     * @throws DatabaseNotFoundException
     */
     function retrieveFoos(){..}

So we're back to two return types. Why treat a single `Foo` differently. It is after all a collection with exactly one item. So if only a single `Foo` is returned put it in an array. It will make the function that much easier to use.

If no `Foo`s were found should you then return an array with a null object of `Foo`? In most cases not. An empty array would suffice.

Crell's own summary on expectations is

<blockquote>
In short: Write good contracts. Obey your contracts. You get one, and only one, return type. Don't make me do extra work because you wrote a dumb contract.
</blockquote>

Good contracts. One return type. Super simple. It is only part of the story. Your trying to make it easier for consumers of your code to use your code. Like with anything else it is often more important to be consistent and clear. 

It is no excuse for not documenting your code. Having only one return type will help to make your documentation more concise. You still need to document your code, though. If going from well documented code that returns several things to code that has only one return value means the new code isn't documented. You might as well leave it as it is.

Only having one return type helps. If you can you should make each function return exactly one type. But why shouldn't it be possible to do so? One example would be legacy code, where you wish to introduce a simple change; not refactor it; not rewrite it.  Breaking existing conventions is often more harmful. While having only one return type makes testing for special cases less frequent, introducing it in a code base might actually make *it* the special case and thus defeating its purpose.

### Return Codes strike back     
Only one return type. Sounds great. Easy to read. Easy to grasp. Reality, however, is never that easy. `strpos($needle, $haystack)` is an internal PHP function, that returns the position of `$needle` in `$haystack`.

    strpos('llo', 'Hello World'); //2

If something other than strings (or things that can be handled as strings) are supplied it throws an exception. If `$needle` isn't in `$haystack` the function returns `false`.

    strpos('yolo', 'Hello World'); //false

Is that right? Shouldn't it throw an exception instead of returning `false`? Is not finding `$needle` an exceptional case? Of course we can throw an exception, but is it expeced? Is it semantically more sound?

Return codes can replace exceptions and vice versa. You can however mix them when necessary. In almost all cases you don't really need to mix them. You should stick with one methodology. If you do mix them make sure to document it. `strpos` is often used wrong, because zero evaluates to false.

I believe `strpos` way of handling the very common 'string not found' case is sound. However the frequent misuse make it clear, that mixing methodologies is dangerous and leads to confusion. 
    
## Summary

Like Larry Garfield I limit myself to one return type. The few extra lines in my functions usually save far more lines in consuming code. It also makes the documentation easier to grasp at a glance. 

Sometimes it is not that clear cut. That is no excuse to not write good contracts. It is a reminder and a warning that even in programming there are greys.




