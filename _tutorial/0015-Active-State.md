Active State
===========
Let's look at an alternative. We'll create a class for each Model. We'll create 
a set and a get method for every attribute. Most IDEs and Editors can automatically generate
these. Our User class looks something like this.

	class User
	{
		private $name = null;

		public function getName()
		{
			return $this->name;
		}

		public function setName( $name )
		{
			$this->name = $name;
		}
        ...
	}

One way to hydrate an instance would be

	$entity = new User();
	foreach( $row as $name => $value )
	{
		$methodname = 'set'.$name;
		if( method_exists( $entity, $methodname ) )
		{
			$entity->$methodname($value);
		}
	}

Other possibilities would be adding all values to the constructor or by adding a 
fromArray method. These two possibilities give you more control over the naming and
number of methods in your classes. But that also means that getting information
about the instance to write it back to the Database is a little more involved. 
Deciding between getter, constructor or specialty injection is at least in this case a matter of taste.

For this tutorial we'll stick with setter injection. Implementing one of the other
methods is just as simple and I encourage you to try it.

Advantages?

 - By separating the hydration from the creation of the class the overall code
 is simplified and easier to understand.
 - We have full IDE Support.
 - Typesafety
 - developers using our code will understand our code and what they are expected to do more easily.

Disadvantages?

 - More code for the developer to type
 - If your database call returns more values than your entity has setters for
 those are lost.
 
Surely we could combine the automatic generation of functions (or at least 
automatic assignment of new attributes) with our concrete
implementations. OUr User would have the advantages of both methods,
but we would also inherit the disadvantages. Personally I think the disadvantages 
outweigh the advantages of such a strategy. 

I think we're almost through with the basics so let's wrap this up.

We still don't have a way to handle values from database calls that aren't part
of our entity model.

Let's imagine for a moment you have a database call where you not only want to
fetch a user a, but also the number of users that share a's first name.

We already said that we do not want to automagically create functions, so we 
have to design the functionality somewhere and also ensure typesafety. We wil 
leave the User class as is.

	class User
	{
		private $name = null;
		public function getName()
		{
			return $this->name;
		}

		public function setName( $name )
		{
			$this->name = $name;
		}
	}

We'll define a special UserAndFirstNameCount class.

	class UserAndFirstNameCount
	{
		private $name = null;
		private $firstnameCount = null;
		public function getName()
		{
			return $this->name;
		}

		public function setName( $name )
		{
			$this->name = $name;
		}

		public function getFirstnameCount()
		{
			return $this->firstnameCount;
		}

		public function setFirstnameCount( $firstnameCount )
		{
			$this->firstnameCount = $firstnameCount;
		}
	}

That would work, but we can't use this class where we could use our user class.
At least not if we use typehints in our code. 

Inheritance would solve our typesafety problem, because you can use children
wherever the parent is allowed. Another option would be using interfaces. Using interfaces
allows others to write their own implementation without the need to subclass a given class
and thus preventing tightly coupled deep inheritance trees. So we should use 
interfaces in any case.

Another problem with the above approach is that if we change User, we have to
adapt this new class (and possible others) as well. Also if we want the 
number of contributed articles we suddenly not only have two new classes but three: 
UserAndFirstNameCount, UserAndArticleCount and UserAndFirstNameCountAndArticleCount.
Granted in this short example UserAndFirstNameCountAndArticleCount would surely
suffice, but I hope you understand the point I'm trying to make.

It would be nice if our extensions were stackable and thus making the class 
combinations (UserAndFirstNameCountAndArticleCount was probably only copied 
code anyway) superfluous. 

This is what the Decorator pattern is for. The Decorators implement the same 
interface as the class they want to extend but proxy all calls to an instance
of that class. The decorator then can change the behaviour of the functions 
or extend the decorated instance with new functionality.

	interface User
	{
		public function setName( $name );
		public function getName();
	}

	class BaseUser implements User
	{
		private $name = null;
		public function getName()
		{
			return $this->name;
		}

		public function setName( $name )
		{
			$this->name = $name;
		}
	}

	class UserDecorator implements User
	{
		private $object = null;

		function __construct( $object )
		{
			$this->object = $object;
		}

		public function getName()
		{
			return $this->object->getName();
		}

		public function setName( $name )
		{
			return $this->object->setName($name);
		}
	}

	class UserAndFirstNameCount extends UserDecorator
	{
		private $firstnameCount = null;

		public function getFirstnameCount()
		{
			return $this->firstnameCount;
		}

		public function setFirstnameCount( $firstnameCount )
		{
			$this->firstnameCount = $firstnameCount;
		}
	}


To get the same functionality as UserAndFirstNameCountAndArticleCount we can 
do the following:

	$user = new BaseUser();
	$userWithFirstNameCount = new UserAndFirstNameCount($user);
	$userWithArticleCountAndFirstNameCount = new UserAndArticleCount( $userWithFirstNameCount );

Since the Decorators and the BaseUser all implement the User interface they can be
safely used in methods with type hints. We do not duplicate code anymore and
most developers should already be familiar with this pattern.

Is this really necessary? For just an extra value? Well I admit that the examples
are quite bland. And for what we're trying to achieve with them you're probably better
of just making a second database call. Maybe this example helps. 

To honor the birthday
of Michael Dorn you decide to display all user names in thlingon. Instead of 
changing possibly several of your templates, you could jsut write an appropriate 
decorator class and have that hydrated. None of your code has to be changed
(except the place where you swap decorators of course) since your still supplying an
instance of user. Also the change in the behaviour of the name actually belongs
into the model. You're not changing anything that influences the control logic of your
application. Also your not changing how the names are viewed, you're actually
changing the name.

That is fine and all but how do we get our database calls to hydrate such a 
decorated class?

We could decorate the instances as we get them from the factory. 

    $user = $userfactory->retrieve('Leeloo');
    $user = new UserAndFirstNameCount($user);

This is of course possible, but would also mean the user of our orm will have to do 
the decorating himself. Also if the decorator is supposed to handle an extra value we need to insert it before the hydration, not after it.

We could just have the factory return a decorated class, but if we hardwired
which decorator(s) to use users still have to possibly do some decorating 
themselves.

We could provide the name of the decorator to use. Better, but we would still 
have to hardwire the BaseUser class (it's debatable if this is actually so bad, 
since it is the only reason UserFactory exists for. Just bear with me for a 
moment longer.) If we wanted to provide more than one 
decorator, we also would have to provide the order in which to decorate. Also
the factory would have to know how to initialize the decorators, if one of them
relied on a second argument to set up.

It would be nice if we could just somehow pass or reference a template instance
to use.

We could have a seperate class EntityProviderService that knew how to setup 
such a blank template instance (aka prototype) given a name. We then could pass
that name to the factory which asks the service for the prototype to use. 

    class EntityProviderService
    {
        static public function getUserPrototype( $name )
        {
            $registeredProtos = array(
                'firstnamecount' => new UserAndFirstNameCount(new BaseUser());
            );
            
            return clone isset( $registeredProtos[$name]? $registeredProtos[$name] : new BaseUser();
        }
    }

    class Userfactory
    {
        ...
        protected function hydrate( $row, $protoname = null )
        {
            $entity = EntityProviderService::getUserPrototype($protoname);
	        foreach( $row as $name => $value )
	        {
		        $methodname = 'set'.$name;
		        if( method_exists( $entity, $methodname ) )
		        {
			        $entity->$methodname($value);
		        }
	        }
            return $entity;
        }
        ...
    }

    $user = $userfactory->retrieve( 'Leeloo', 'firstnamecount' );

    

That 
sure is an improvement - wait for it - but the factory now has to know about 
the service. So our users now have to write the factory, the entity, possible
decorators and configure a service. Writing the factory and the entity can
probably be automated by inspecting the database and generating appropriate 
code, but the service has to be configured to know about every possible 
combinations of decorators, which also have to be named. All that just for 
supplying a prototype to use? One that in most applications will 
always be the same?

Why don't we just provide that prototype ourselves and if our users have need
for such a ServiceClass they can still use it, they just have to call it before
the factory starts its work.
 
Extend the UserFactory and include a `setPrototype` function. A user can now supply a prototype as follows

    $userfactory->setPrototype( new UserAndFirstNameCount(new BaseUser()) );
    $userfactory->retrieve('Leeloo');

Don't forget to clone the prototype before using it in the hydration.

That's been alot of hard work for something as simple as connecting to a 
database. It is still far from perfect. We have no automatic code generation
and no caching layer. But there is something even more important that is 
missing, which we'll cover the next time.

## Duck Contractors

Up to now we relied on PHP's *duck typing* ability. We did not enforce a certain class, rather we expected the provided class to behave as expected.

    class Fowl
    {
        public function sound()
        {
            echo "quack";
        }

        public function swim()
        {
            echo "paddle";
        }
    }

    class Duck extends Fowl{};

Through inheritance Duck has all methods of Fowl. So the following will work
    
    foreach( array( new Duck(), new Fowl()) as $duck )
    {
        $f->sound();
    }

But as long as a class behaves as a function expects it it won't care if it's processing birds or not.

    class Lion
    {
        public function sound()
        {
            echo "roar";
        }

        public function pounce()
        {
            echo "nomnom";
        }
    }

    foreach( array( new Duck(), new Fowl(), new Lion()) as $duck )
    {
        $f->sound();
    }

That works, although Lion is clearly no Fowl. So long as it looks and behaves like a duck, the above code won't mind. If an instance does not behave as expected, we will during the runtime of the function.

PHP can be made to be a bit more strict about this however.

    function sound( Fowl $f )
    {
        $f->sound();
    }

    foreach( array( new Duck(), new Fowl(), new Lion()) as $duck )
    {
        sound( $f );
    }

Now PHP will check if the passed object belongs to Fowl or one of its subclasses, *before* the function is executed. That means the above code will sound Duck and Fowl, but throw an error at the Lion, because the Lion is no Duck.

This forced check is called type hinting and is useful in object oriented code, because the compiler will enforce this rule and it will throw an error before executing the function. Or expressed another way "The function sound vows to work as expected, if you promise to give it a Fowl".

The function `sound` does not rely on all of Fowls methods, indeed it would work on Lion and cars and catchphrase characters. As long as they implement sound. We could create a class that only implements the `sound` method and have each class inherit from it.

But their are some drawbacks to this

 - long inheritance chains
 - cars, lions, catchphrase characters and fowl are not related. 
 - PHP only suports single inheritance, what if we need to implement another function, only a few classes need?

Those points are not unsolvable but there is a more elegant solution. We can tell PHP that all these classes are things that make sounds. We do this by implementing interfaces.

Interfaces are a contract. They ensure that certain abilities are available and implemented.

    interface Clamor
    {
        public function sound();
    }

An interface defines the publicly available method signatures. A method signature is the name of a method with all its modifiers, parameters and typehints. This means, that a method may never alter, omit or append these modifiers, parameters and typehints.

    class Fowl implements Clamor
    {
    ...
    }

We can now type hint against Clamor and make our lion roar again.

Is this all really necessary? Your code now more clearly expresses your intent and actual relations. A class can also implement multiple interfaces. That is not the same as multiple inheritance, but it is powerful enough for almost any usecase.

Also interfaces can grant your code additional features. PHP itself defines a good handful of interfaces. Which can make an object accessible like an array or make it possible to iterate over the object. just like magic methods interfaces can dramatically alter the way people interact with your code.

