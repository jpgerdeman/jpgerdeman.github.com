More Organization
=================

Our frontcontroller now looks something like this

	require_once 'library/Request.php';
	require_once 'orm/library/Connection.php';
	require_once 'orm/library/Driver.php';
	require_once 'orm/model/Userfactory.php';
	require_once 'orm/model/BaseUser.php';
    require_once 'view/ViewConfiguration.php';
    require_once 'view/View.php';
    require_once 'view/LayoutView.php';
	require_once 'controller/Controller.php';
    
    $action = getRequestParameter('action', 'hello');

	$userCon = new Connection();
	$userCon->setName('user');
	$userCon->setUsername('oopbydesign');
	$userCon->setPassword('tutorial');
	$userCon->setHost('localhost:3306');
	$userCon->setDatabase('user');

	$driver = new Driver();
	$driver->addConnection($userCon);

	$userFactory = new UserFactory();
	$userFactory->setDriver($driver);
	$userFactory->setPrototype(new BaseUser());

    $controller = new Controller($action);
    $controller->addVariable('userFactory',$userFactory);
    $view = $controller->execute();
    $layout = new LayoutView($view);
    foreach( $view->getComponents as $name => $comp )
    {
        $c = new Controller($action);
        $c->addVariable('userFactory',$userFactory);
        $v = $c->execute();        
        $layout->addComponent($name, $v);
    }
    
    echo $layout->render();

Without the `executeAction` function it has become more readable. We have quite a few require statements and will get more and more with every improvement. Also your users might want to use some classes of their own. Then we have the problem, that some of our class names are too generic. `Driver` and `Connection` could mean anything outside the context of our orm. The same is true for most of the class names. 

##Namespaces

Lets take care of the class names first. We could prefix every class name with the context. `UserFactory` would become `Orm_Model_UserFactory` and make the context of the class explicit. Coincidentally it also reflects our folder path for our class. So the name also tells us, where to find the file. That's nice. In fact this is what PEAR and the Zend framework used before PHP 5.3.

PHP 5.3 introduced namespaces. Namespaces are like drawers. Classes of the same name can live side by side if they are in different namespaces (Well technically they then do have different names). `Orm_Model_UserFactory` becomes `Orm\Model\UserFactory`. 

    class Orm_Model_UserFactory{}

becomes
    
    namespace Orm\Model; #Must be first executable line in a file
    class UserFactory{}

and you can use it like this 

    new Orm\Model\UserFactory();

That's pretty much the same. So where is the difference? The difference is in handling. If you're in a namespace you can use Classes from within that namespace without writing out the namespace everytime.

    # File Baz.php
    namespace Foo\Bar; 
    class Baz{}

    #File Bar.php
    namespace Foo\Bar;
    require_once 'Baz.php';
    
    $f = new Baz();

Say you want to use the classes `Foo\Bar\Baz()` and `Fubar\Baz()` in one file. You can either write

    require_once 'Foo\Baz.php';
    require_once 'Foobar\Baz.php';

    $f = new Foo\Bar\Baz();
    $g = new Fubar\Baz();

or you can write

    require_once 'Foo\Baz.php';
    require_once 'Foobar\Baz.php';
    
    use Foo\Bar\Baz;

    $f = new Baz();
    $g = new Fubar\Baz();

or even better

    require_once 'Foo\Baz.php';
    require_once 'Foobar\Baz.php';

    use Foo\Bar\Baz as FooBaz;
    use FuBar\Baz as FuBaz;

    $f = new FooBaz();
    $g = new FuBaz();

Namespaces are much more convenient than putting the namespace in the name of the class. On the other hand namesapces won't work with a PHP version prior top PHP 5.3.

## Autoloading
We already observed, that our namespaces match the folder path of our files. This doesn't have to be so. You can name your files whatever you want. PHP only cares wether the files are loaded or not.

However it is neat to have namespaces and folders match up. It would be even nicer, if we could use that coincidence to our advantage.

PHP has two ways we can use this. `__autoload` and `spl_autoload_register`. They allow us to register functions, which get called, when a class or interface isn't found.

The simplest autoloader is surely

    function __autoload($classname)
    {
        include_once $classname.'.php';
    }

Which tries to include a file by the same name as the class, *if* that class hasn't been defined yet. So the following would work

    // Folder structure
    index.php
    Foo.php
    Driver.php
    ...

    // index.php
    # No includes necessary we're using an autoloader
    function __autoload($classname)
    {
        include_once $classname.'.php';
    }

    $f = new Foo();

That is convenient! No matter how many classes we we write, our frontcontroller does not have to change. No more include or require statements.

That autoloader should also work with our directory layout as long as the namespace reflects the path, because the namespace separator is identical to the folder separator on Dos based machines. PHP should be able to handle it. However it is probably safer to replace the namespace separator by the proper directory separator

    function __autoload($classname)
    {
        include_once str_replace('\\', DIRECTORY_SEPARATOR, $classname).'.php';
    }

Since every function may only exist once, we can't use two or three autoloaders like this. So everything has to lie in the same folder. Not unhandable, but troublesome if a user prefers another way to organize files or uses an external lib with its own autoloader.

Luckily PHP also defines a more flexible alternative `spl_autoload_register`, which allows to register several autoloaders.

    function myCoolAutoloader($classanme){}

    class MyAutoloader
    {
        static public function load($classname){...}
    }

    spl_autoload_register('myCoolAutoloader');
    spl_autoload_register(array('MyAutoloader', 'load'));

The autoload functions should return `true` or `false`. Now when an unknown class is encountered `myCoolAutolaoder` is called then `MyAutoloader::load` is called. So we can use an autoloader for each library and the user of our code can define his own. 

##PHP Framework Interoperability Group
A bunch of intelligent people from different frameworks and libraries got together and put out a recommendation - PSR-0 - on how files should be organized. It basically says that the folder path should be equal to the namespace. There are a few bells and whistles to it, but that's the essene of it. There exist a few good robust implementations of PSR-0 compatible autoloaders, that are much smarter than ours.
