More ORM
========
Our ORM is far from perfect. Users still have to supply alot of code to 
begin. Also our factory returns user as a standard class.  It would be nicer
to have a dedicated class for that, so we have a place to put our model logic.

We need some way to add logic to our models. Calling - for example - save on 
the user (`$user->save()`) feels much more natural than going back to the 
factory to do so. (Though the logic of actually doing this is not in the
user responsibility.
 
Possibly by using anonymous functions we could achieve what we want. Somethig like

    $user->save = function()use($user){
      UserFacory::save($user);
    };

Sadly this doesn't work as expected. Calling $user->save() will result in an
error. To actually call the function you would have to

    $save = $user->save;
    $save();

That's not very handy. We could define a simple baseclass that implements the
__call magic method, which then resolves the function call nicely.

	class closureObj
	{
		public function __call( $method, $args )
		{
			$method = strtolower($method);
			if( is_callable(array($this, $method)) )
			{
				call_user_func($this->$method, $args);
			}
		}
	}

to hydrate our model instances we could do something like

	$entity = new closureObj;
	foreach( $row as $name => $value )
	{
		$getMethodName = strtolower('get'.$name);
		$setMethodName = strtolower('set'.$name);
		$entity->$name = $value;
		$entity->$getMethodName = function()use($entity,$name){return $entity->$name;};
		$entity->$setMethodName = function($value)use($entity,$name){$entity->$name = $value;};
	}

If you wonder why I change the method names to lower case. By assigning functions
to variables - which are case sensitive - the methodcalls also become case sensitive.
By forcing everything to lowercase they aren't case sensitive anymore.

What are the advantages of this?

- There is no need to write a class for every model since they are generated on the fly. Less work.
- If we make a select on the database that returns more than the usual attributes we automatically get setters and getters.

Any disadvantages?

- No IDE support. At least not in every IDE and not without documenting the 
  methods in the doc comment of closureObj, which in this case means you will 
  always be offered all methods of all entity classes.
- the properties of our instances have to be public.
- You can't be sure which functions are available. We could supply a list of 
  Methods which always have to exist. But this does not cover those values that could be added by lets say joins
  that aren't meant to always exist. So before calling a method you first have to check if it exists.
- No Typesafety. This is connected to the point above. Creating Entites like this we can't make our 
  funtions typesafe which means we loose one of the strongest methods to make our code more robust.

## Off the Deep end
Anonymous functions? Magic Methods? What's going on?

I admit I glanced over them, as I introduced those concepts. So lets have a closer look.

Anonymous functions are functions, that can be defined on the fly, without polluting the namespace. This is very handy for short functions you might only need once.

    function someLameNameBecauseIOnlyNeedItOnce($a, $b)
    {
        return $a + $b;
    }

    $someLameNameBecauseIOnlyNeedItOnce = function( $a, $b ){ return $a + $b; };
    $someLameNameBecauseIOnlyNeedItOnce(3,4); //7

So instead of thinking of possibly cryptic names (includes anotherName, someOtherName, C3P0, ...) as to prevent naming conflicts, we can assign a function to a variable. Once the variable leaves its scope it is destroyed and the function with it.

The function we defined above is a anonymous function known as lambda. Lambdas have no access to their surrounding scope.

    $c = 5;
    $f = function($a,$b){ return $a+$b+$c; };
    $f( 3, 4 ); // Will return 7 and not 12. $c is not defined inside the function and therefore null.

There is another type of anonymous function, called closure. A closure is like a lambda, except that it can access its surrounding scope. In PHP we write it like this.

    $c = 5;
    $f = function($a,$b)use($c){ return $a+$b+$c; };
    $f( 3, 4 ); // Will return 12. 

So in PHP we have to explicitly tell the closure which variables it can use. In PHP the difference between closure and lambda is marginal. In other languages the difference might be more pronounced.

With this set of tools it is possible to use another programming paradigm called "functional programming". We're not going to talk about it, because it currently has very little relevance in PHP. It is gaining however, so you might want to read up on it, once your comfortable with PHP and OOP.

Internally anonymous functions are turned into a class with a random name

    class q4we5r6ewgh8 
    {
        public function __invoke( $a, 4b )
        {
            return $a + $b;
        }
    }

    $i = new q4we5r6ewgh8 ();
    $i( 3, 4 );

Did we just use a class as if it were a function itself? Yes.

PHP defines several *[Magic Methods](http://php.net/manual/en/language.oop5.magic.php)*, which alter the way people can interact with your code. These Methods are
`__construct()`, `__destruct()`, `__call()`, `__callStatic()`, `__get()`, `__set()`, `__isset()`, `__unset()`, 
`__sleep()`, `__wakeup()`, `__toString()`, `__invoke()`, `__set_state()` and `__clone()`.

We'll only talk about a few of these. you can read upon the rest in the PHP manual.

`__construct()` is called when a new class instance is created via the `new` keyword. This is where you can initialize your class, set some defaults and [poka yoke](http://de.wikipedia.org/wiki/Poka_Yoke) your instance.

`__call()`  is executed, whenever a none existent method on an instance is called.
So you can define dynamic methods on your instance or overload a method (there are other ways to do this as well).

	class A
	{
		public function __call($name, $arguments)
		{
			if( $name == 'add' )
			{
				if( count($arguments) == 2 )
				{
					return $arguments[0] + $arguments[1];
				}
				elseif( count( $arguments ) == 0 )
				{
					return $arguments[0];
				}
			}
			
			throw new Exception( "unknown method '$name' ". implode(', ', $arguments) );
		}
	}

	$a = new A();
	$a->add(3);
	$a->add(3,4);
	$a->add(2,3,4);  // Exception unknown ....

`__set()` and `__get()` get called when an unknown proeprty is called.

`__invoke()` is called, when a instance is used as a function.