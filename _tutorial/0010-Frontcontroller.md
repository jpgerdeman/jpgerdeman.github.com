Our Code Stinks
===============

Before we continue with our exploration of the object oriented paradigm, we'll cool off with some easy changes. These changes will improve our code and its organization and will make future tweaks less neccessary.

So our framework is still small and simple, but we already notice a few
code smells. Code smells are parts of code that just don't seem right and
will probably make the code unmaintainable in the future.

For now we see two major problems:

* The adduser action is ugly and hard to read. We should somehow seperate
the html code from the logic.
* Our frontcontroller also will be unwieldy very soon. It would be better to 
sepearte the logic from the actual data. Also our action names are very generic. Once we start to add more elements we're going to run into awkward names to avoid doubles. It would be nicer to group the functions somehow.
 
We'll start with the front controller and then take care of our actions.

A simple solution would be something like

    if( file_exists('actions/'.$action.'.php') )
	{
	    require_once('actions/'.$action.'.php');
	}
	else
	{
	    require_once( 'actions/hello.php' );
	}

As long as we keep to our convention of naming the actions this would work 
quite well. We just have to drop in a new action file and voilá it works. How awesome is that! Still we might run into trouble when we try to add a function `add_book`. In this case `add` is not clear enough. We could rename all actions to `user_add`, `user_profile` and `user_hello` and be done with it.
 
We could also put our current
actions into a folder `user` since they all operate on user instances. this will save us a few keystrokes in the long run and will make browsing the code easier. The 
request action parameter could be prefixed by this, e.g. 'user_profile'. 

For now this is how we'll handle it.
    
    require_once 'library/Request.php';
    require_once 'library/Database.php';
    getRequestParameter('action', 'hello');
    $actionfile = 'actions'.DIRECTORY_SEPARATOR.str_replace('_', DIRECTORY_SEPARATOR, $action);
    if( file_exists($actionfile) )
    {
        require_once($actionfile);
    }
    else
    {
        require_once( 'actions/user/hello.php' );
    }

What have we gained by this refactoring? Admittedly little. No new functionality has been added to the code; at least none a user can see. On the other hand adding actions has just become easier. You just have to add a new file at the appropriate spot and it is immediatly available. This means fewer changes in our code are necessary, none in the case of the frontcontroller.

Next we'll see how to improve the `user\add.php`.