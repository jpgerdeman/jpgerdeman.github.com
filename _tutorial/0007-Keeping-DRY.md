Let's keep DRY
==

If we look at our frontcontroller and our actions we'll see three very similar 
lines asking for information from $_GET. We'll extract that into a function this will also help us remember that some parameters might not be set.
Create a new folder library with a file `Request.php` in it. The library folder
will hold all our functions. The `Request.php` will hold the functions for accessing
the request variables. In `Request.php` create the following function

    <?php 
    function getRequestParameter($name, $default)
    {    
        $parameter = isset($_GET[$name])?$_GET[$name]:$default;
        return $parameter;
    }

Now replace every access to `$_GET` by this new function.

## Refactoring
Refactoring is the process of restructuring your code, without altering its external behavior. This restructuring is done with the intent of improving the maintainability and readability of code. Usual goals of refactoring are reducing and reusing. Reusing code, like we did with our new function, ultimately means that we can reduce lines of code and complexity of other pieces of code.

When we extracted the actions in their own files we improved the complexity of our frontcontroller. It now contains less code which makes it easier to read and understand. By extracting `getRequestParameter` we made it possible to improve a piece of code in a single function and have all users benefit from it immediately.

Also it is easier to debug and profile code, when it is broken up in smaller chunks. It is then quite simple to identify the performance sinkholes. 

Refactor - Reduce - Reuse

## Abstraction
Extracting this piece of code into a function will also make it possible to abstract the request a bit. We want our framework to be easy to use. `getRequestParameter()` makes it possible to abstract GET and POST parameters. Both are types request parameters. Why not make it possible to get a parameter from either without knowing where it comes from.

    <?php 
    function getRequestParameter($name, $default)
    {    
        $getparameter = isset($_GET[$name])?$_GET[$name]:null;
        $postparameter = isset($_POST[$name])?$_POST[$name]:null;
        
        if( !is_null($getparameter) )
        {
            $param = $getparameter;
        }
        elseif( !is_null($postparameter) )
        {
            $param = $postparameter;
        }
        else
        {
            $param = $default;
        }
        
        return $param;
    }

If we have POST and GET parameter by the same name, the GET parameter will be returned. This is of course only one naive way to do it. However it does abstract POST and GET from your user now, whom doesn't need to bother with it. 

Abstractions like this make it that much easier to write code with our framework. Of course you have to find a balance between abstracting too little and too much. Generally though abstraction is favorable. Specifications are a good indicator where abstraction can be introduced. If there is a specification for it most times you can abstract it and more often then not you also should.

Have a look at the above function. See how The use of vertical space draws the eye. The first two ternaries don't look as important as that if statement, they have less optical weight. The eye is immediately drawn to the if statement, the critical piece in our function.
## DRY

Is there any point in extracting code into functions? Yes the DRY-Principle 
(don't repeat yourself) is one of the core principles of clean code.

Our first function admittedly added another file and five lines of code
to your code. On the other hand your code is now more expressive since the
name of our function expresses our intent clearly. It has now also become 
easier to maintain our code. Why? Well If you must change the way you access
the request variables you now only have to change code in one place.

In fact we should do just that since there is a huge security hole in our framework.
