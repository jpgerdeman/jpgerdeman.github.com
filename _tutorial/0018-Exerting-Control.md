Exerting Control
=================
Lets have another look at our frontcontroller.


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

So we have our ORM and our View. The function `executeAction` is a nice wrapper around our controllers. We could put that in a class as well. Especially since we might need to insert more than our user factory. It could look something like:

    class Controller
    {
        public function __construct($actionname){...};
        public function addVariable($name, $value){...};
        protected function getFilenameForAction(){...};
        public function execute()
        {
            extract($this->variables);
               
            # Actions will update the config.
            $view = new ViewConfiguration();
       		require($this->getFilenameForAction());
    
            return $view;
        }

    }

Quite nice. With this in place we can make the handling for the consumer of our code a bit more convenient.

    /**
     * Return a preconfigured ViewConfiguration instance.
     *
     * Fills out the ViewCfg based on some conventions. Can still be changed in 
     * the action.
     *
     * @return ViewConfiguration
     */
    protected function getViewCfg()
    {        
        $view = new ViewConfiguration();        
        $view->setTemplate($this->getDefaultTemplateName());
        return $view;
    }

    /**
     * Returns the default template name of this action.
     * 
     * Based on the convention, that the templates are in a template subfolder
     * and named like the action.
     * 
     * @return string
     */
    protected function getDefaultTemplateName()
    {
        $dirparts = explode('_', $this->actionname);
        $template = array('actions',$dirparts[0], templates, $dirparts[1].'.php');
        return implode(DIRECTORY_SEPARATOR, $template);
    }

We can even do something like this

    public function execute()
    {
        extract($this->variables);
           
        # Actions will update the config.
        $view = new ViewConfiguration();
   		require($this->getFilenameForAction());

        foreach( get_defined_vars() as $k => $v )
        {
            $view->addVariable($k,$v);
        }

        return $view;
    }

So a consumer doesn't even has to worry about assigning a template or a variable to the view configuration, we do it for him. How awesome is that! Again we see that by introducing a more general solution to a problem, like we did with inserting variables to the controller; we can also exploit it to further enhance our code, like the way we made it easier to write actions.

