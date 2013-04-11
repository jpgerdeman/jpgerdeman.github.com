Time to Brush Up Our Look
=========================
Now that our database abstraction has reached a milestone in its evolution lets 
take another look at our site from the outside.

Well it can't do alot right now, that's for sure. Even more appaling is it's looks.
We should put a bit more polish to it. Our pages need a header which displays our 
name in huge red letters and a footer with the disclaimer. 

As we only have a handful of pages right now it would be easy to just write it into 
all our templates.

	<div class="head" style="color:red;font-size:huge;">SimpleMVC</div>
	<div>Profile:</div>
	<?php	
	printf('Name %s', $user->name);
	printf('UserName %s', $user->uname);
	printf('email %s', $user->email);
	?>
	<div class="foot" style="color:grey;font-size:tiny;">You no like it? I kill you!</div>

But changing either header or footer, or even adding a new element to each page, 
will become tedious once we have more actions. Also we don't want to repeat ourselves.
There has to be a better way. 

Maybe we need to add the repeating elements, lets call it layout, before we 
get to the views. Adding them to the actions works, but leaves us with exactly
the same dilemma. Also we have just invested time to seperate the template from 
the action.

Even before our actions come into play the frontcontroller acts. 

	...
	<div class="head" style="color:red;font-size:huge;">SimpleMVC</div>
	if( file_exists($actionfile) )
	{
		require_once($actionfile);
	}
	else
	{
		require_once( 'actions/user/hello.php' );
	}
	<div class="foot" style="color:grey;font-size:tiny;">You no like it? I kill you!</div>
	...

That actually works. All changes to our layout can be done in one place now. If 
we want to execute two actions we could always hardcode the call to that one
in our layout. Imagine we want to greet our user in the header on every page, 
while still being able to display the other actions. 

Switching layouts, e.g for printing or displaying popups, is also possible though 
bothersome. We'd need to create a new frontcontroller for each layout. Our 
frontcontroller wouldn't be a frontcontroller anymore. Also changes to our front-controller
have to be repeated several times.

So while this works, we have the following drawbacks

 - Calling further actions in the layout have to be hardcoded
 - Doubling of frontcontroller logic for each layout
 - Display and logic mixed. Didn't we want to separate that?

One improvement would be to extract the layout and the call to the action into 
its own layout file. Pass another queryparameter `layout` to the frontcrontroller, 
which now calls the appropriate layout file. That takes care of the doubling of
the frontcontroller, but now we must supply the layoutname, instead of our 
framework deciding which to use.

What do we want to achieve anyway. We have an action, that we would like to decorate 
with our layout. Hm. Wasn't their a pattern we used for decorating. Yes there was
and this is a text book use for it.

The decorator pattern is a pattern for object oriented programming so we'll need 
some kind of objects. Since we're trying to decorate the templates, they have to 
be objects. We want to decorate the layout around them, so their is another object.
That sounds easy.

Hold on. We are making our problem fit a pattern, instead of finding a pattern
that fits the problem. Putting layout and templates into classes is horrible!
Not only are we mixing things again we're also making writing HTML that much 
more difficult. Lets look at what we want to achieve another way.

We want our site to have a uniform layout (or one of few), that is draped around
the content. The content is the result of our action. We also want to include
more actions (components) in the layout to display dynamic information, that the user has not
explicitly requested. Like greeting him by name at the top of the page.

How will we know what components to include and where? Who will decide which 
layout to include? Most things could be written into a configuration, but it has
to allow to be overwritten by actions. That way actions can still influence how they
are displayed and what other components they need.

	/**
	 * Configuration of the view. 
     *
     * Each action should return a ViewConfiguration, so that the final
	 * page can be pieced together. This grants great flexibility as actions
	 * can overwrite the configuration.
     *
     * We separate actions in to groups. The action that was called and will
     * be displayed as main content is designated as the action. Actions that
     * are called to supply secondary information are calle components.
	 */
	class ViewConfiguration
	{
		protected $layout = null;
		protected $template = null;
		protected $components = array();
		protected $variables = array();

		public function setLayout( $layout )
		{
			$this->layout = $layout;
		}

		public function getTemplate()
		{
			return $this->template;
		}

		public function setTemplate( $template )
		{
			$this->template = $template;
		}
		
		public function addComponent( $viewname,  $component )
		{
			$this->components[$viewname]=$component;
		}

		/**
		 * Add a variable to be passed to the template.
		 */
		public function addVariable($name, $value)
		{
			$this->variables[$name]=$value;
		}
	}

We'll also need a View class which should have a `render` function and a `setTemplate`
function. The render function must pass the variables to the template and catch
the output of the template.

	public function render()
	{
        ob_start();
        extract($this->variables);
        include_once($this->template);
		$result = ob_get_clean();
		return $result;
	}

`ob_start` starts an output buffer which catches everything that is outputed by
PHP. `ob_get_clean` ends the buffer and returns the output. `extract` takes an
array and creates variables for each entry. The variable name is the key and the 
value becomes the $value of the variable. 

The `ob_*` functions and the `extract` functions are powerful and it is good to 
know them. Especially the `ob_*` functions are much more powerful than they appear
here. You might want to read about them in the manual. However they belong to
the seldom used functions.

Now we need a LayoutView, which knows how to render the actions and components and
then renders the layout. The render function of it might look something like this

	class LayoutView
	{
		public function __construct( $config )
		{
			$this->variables = array();
			$this->template = $this->layout;
			$this->actionView = new View( $config );		
		}
		
		protected function renderSubViews()
		{
			$variables = array();
			$variables['content'] = $this->actionView->render();
			foreach( $this->componentViews as $name => $cView )
			{
				$variables[$name] = $cView->render();
			}
			$this->variables = $variables;
		}

		public function render()
		{
			$this->renderSubviews();
			ob_start();
			extract($this->variables);
			include($this->template);
			$result = ob_get_clean();
			return $result;
		}
		
		public function addComponentView( $name, $view )
		{
			$this->componentViews[$name] = $view;
		}
	}

And that is our very simple view component. Our frontcontroller has to be changed a bit as well


	require_once 'library/Request.php';
	require_once 'orm/library/Connection.php';
	require_once 'orm/library/Driver.php';
	require_once 'orm/model/Userfactory.php';
	require_once 'orm/model/BaseUser.php';
    require_once 'view/ViewConfiguration.php';
    require_once 'view/View.php';
    require_once 'view/LayoutView.php';
	
    function executAction($action, $userFactory)
    {    
    	$actionfile = 'actions'.DIRECTORY_SEPARATOR.str_replace('_', DIRECTORY_SEPARATOR, $action.'.php');
    
        # Actions will update the config.
        $view = new ViewConfiguration();
    	if( file_exists($actionfile) )
    	{
    		require($actionfile);
    	}
    	else
    	{
    		require( 'actions/user/hello.php' );
    	}
        return $view;
    }
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

    $view = executeAction($action, $userFactory);
    $layout = new LayoutView($view);
    foreach( $view->getComponents as $name => $comp )
    {
        $v = executeAction($comp, $userFactory);
        $layout->addComponent($name, $v);
    }
    
    echo $layout->render();

So what have we achieved? We talked about what we wanted to achieve and *how a user could use it*. That second bit is often forgotten but it is important. Too often I see useful libraries that force a user of the library to know it inside out. Of course your users are going to be fellow developers, but you're trying to deliver a solution not a tuition. 

Our solution also lends to more improvements. Since we render each template to a string, we can just as easily save the rendered string and display that instead of running our controller and view again; a view cache. That would decrease the load on our server and make our sites faster. Changing the layout to something else is also really easy now. By defining the view configuration we decoupled the templates and the view from the actions. In other words our view is flexible and ready for future improvements. 

We also covered a lot of ground development wise. A lot more than we usually cover, but it didn't feel like that much more, ne? Once concepts in your code and more importantly in your head are available it makes talking about code easier. 

See you next time.