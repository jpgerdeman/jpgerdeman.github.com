Forging Ahead
=============

Now that we had time to let our first experience in object orientation sink in and had our first glimpse at patterns, let us come back to our Database code.

Right now we have one class Database, which we plan to hold all our Database functions. As long as we only interact with one table this is going to work out. However if we started to add another table, say books, we'd have to prefix all methods to specify if it works on users or books.

This is a code smell. Whenever you have a set of methods that you have to prefix that is a sure sign, that you have to refactor your code. Since we'd have to prefix all of our methods with user, we'll extract them to a new class UserFactory.

Create a new file `model\UserFactory.php` and copy the Database class into it renaming it UserFactory. Since we're moving it we might as well have a look at the code itself. The code to set up the database connection is duplicated in every function. Also we reference variables, location, database, password and user that are not explicitly set.

Create setters for these and put the connection code into its own method. Since our UserFactory is going to have state we might as well turn the methods into instance methods.

    class UserFactory
    {
        private $username;
        ...
        public function setConnectionUsername( $username )
        {
            $this->username = $username;
        }
                
        public function connect()
        {
            mysql_connect($this->location,$this->username,$this->passwd);
            mysql_select_db($this->dbname);     
        }
    
        public function retrieveUserByUname( $uname )
        {   
            $this->connect();
                
            $stmt = 'SELECT * from user WHERE uname = '.$name;
            $result = mysqli_stmt_fetch( $stmt );
            $user = mysqli_fetch_object($result);
            return $user;
        }
    
        ...
    }    

Strictly speaking all methods and properties could have stayed static, since we'll only ever have one user database.

## Hebrew Influences
Since we already have talked about the method modifiers, lets have a look at the reference keywords.

We already introduced '->' as the method dereference operator, which only works on instances. Static references use the '::' notation known as paamayim nekudotayim, scope resolution operator or simply double colon. We already saw that we can access static methods and properties using this notation (i.e. the cookie cutter of our gingerbreadmen). We can also use this notation to access properties and methods hidden deeper.

To access properties and methods we need to have a way of referencing a class or instance. When creating an instance we can reference it by the pointer, i.e. the variable we assigned the new instance to.

    $a = new SomeClass();
    $a->someMethod();

We can reference the instance from within the instance using the special `$this` variable

    class SomeClass
    {
        protected $foo = 3;
        public function someMethod()
        {
            echo $this->foo;
        }
    }

To access a static method or property we need the classname. From within we can use the special Keyword `self`

    class SomeClass
    {
        static protected $foo = 3;
        static public function someMethod()
        {
            echo self::$foo;
        }
    }
    
    SomeClass::foo();

So far this should have been pure repetition. Now to something new. PHP supports inheritance, this means a class can inherit another class and overwrite properties and methods. We call the inherting classes Children or subclasses, the inherited class is called parent. PHP only supports single inheritance, which means each class can have exactly one parent (the parent can be inheriting from another class though). 

    class Someclass
    {
        static private function bar()
        {
            echo "Someclass::bar()";
        }

        static protected function baz()
        {            
            echo "SomeClass::baz()";
        }

        static public function foo()
        {
            self::bar();
            echo "SomeClass::foo()";
        }
    }

    class MoreClass extends SomeClass
    {
        static protected function baz()
        {
            echo "MoreClass::baz()";
        }

        static public function foo()
        {
            parent::foo();
            echo "MoreClass::foo()";
            self::bar(); // Illegal. bar is private and can not be called by children.
        }
    }

So a subclass can't call the private methods and properties of its parent, it can only access the protected and public methods and properties. These however can be referenced by using the `parent` keyword as seen in the code above.

PHP does not have a `child` keyword. In most cases if you need to reference a subclass a call to `get_called_class()` should suffice.

PHP defines another keyword that was introduced in PHP 5.3. We have met `static` as a method modifier, there is also a `static` keyword, that can be used to reference the current class with respect to inheritance, which is also known as late static binding.

While not exactly true, you can still memorize static as *self + inheritance* or *self that behaves like $this*. What do I mean by that? Lets see

    class Someclass
    {
        static public function bar()
        {
            echo "Someclass::bar()";
        }

        static protected function baz()
        {            
            echo "SomeClass::baz()";
        }

        static public function foo()
        {
            self::bar();
            static::baz();
            echo "SomeClass::foo()";
        }
    }

    class MoreClass extends SomeClass
    {
        static protected function bar()
        {
            echo "MoreClass::bar()";
        }
        
        static protected function baz()
        {
            echo "MoreClass::baz()";
        }

        static public function foo()
        {
            parent::foo();
            echo "MoreClass::foo()";            
        }
    }

    Moreclass::foo(); 
    // SomeClass::bar()
    // MoreClass::baz()
    // SomeClass::foo()
    // MoreClass::foo()

So what happened? `MoreClass::foo()` called `SomeClass::foo()`. That was the easy part. `SomeClass::foo()` first calls `self::bar()`, that is the bar method on the calling class *itself*. Since it is invoked in SomeClass the method `SomeClass::bar()` is called. Next is a call to `static::baz()`, which results in a call to `MoreClass::baz()`.

`self` is bound before you call a method, so `self` can only reference the class itself; it does not know from where you're going to call it before you call it. `static` gets bound late, i.e. after you make a method call. Now it is possible for `static` to know from which scope your calling it and can factor that in. 

So where do the Hebrews come in, you ask? Well alot of time and energy was spent on PHP by a group of developers from Israel. To honour their effort the scope resolution operator is named paamayim nekudotayim.