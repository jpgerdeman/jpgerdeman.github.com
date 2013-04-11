Benefits of keeping DRY
==
Try calling our framework with index.php?name=anonymous;DROP TABLE user;
Since we pass the parameter around without escaping it, it is easy for 
attackers to insert malicious code. This type of attack is called a Cross Site
Scripting (XSS) attack.

Since we extracted the code to get a parameter from the Request into its own 
function it is now really easy to correct that. 

    function getRequestParameter($name, $default)
    {
        $parameter = isset($_GET[$name])?$_GET[$name]:$default;    
        return htmlspecialchars($parameter);
    }

We should also shield our database code from such an attack. Extract the database calls to their own file `lib\Database.php`. And change your actions to use this new function.
    
    <?php
    function retrieveUser( $name )
    {
        $name = mysql_escape_string( $name );
	    mysqli_connect( $server, $username, $password );
	    $stmt = 'SELECT * from user WHERE uname = ' . $name;
	    $result = mysqli_stmt_fetch( $stmt );
	    $user = mysqli_fetch_object( $result );
	    return $user;
    }

Don't forget to include the new file at the beginning of your `index.php`.

Again we see that our code has become more readable because of the refactoring.
Also our actions have become much smaller.

##Clean Code and Refactoring
Writing clean code is essential. You can write bad code and get away with it for some time. You or another developer will pay the cost once it stops working. Reading bad code is hard and takes time. It is difficult to correct and in most cases impossible to extend.

Writing clean code doesn't take longer than writing bad code. It takes practice however and experience.

The idea of writing clean code isn't new, but was formalized in "*Clean Code: A Handbook of Agile Craftsmanship*" by Robert C. Martin. 

About half of the book is dedicated to higher concepts of programming. The first half is dedicated to simpler ideas:

 - Be consistent; use a Coding Styleguide. The styleguide should tell you how to format your code (Tabs, Spaces, Brace Placement), what naming strategies to use, how to document code and may even include recommendations on coding practices. PHP doesn't come with a styleguide as other languages do, so you are free to choose your own. Look at the styleguides of other projects (e.g. Zend Framework 2 or PSR-1 and PSR-2) as a starting point, or simply adopt one.
 - Keep your code blocks short. The function retrieveUser has nine lines including braces and is very easy to understand.
 - Use good names. `retrieveUser` might be better be named `retrieveUserFromDatabase`, or we put in our styleguide, that the prefix retrieve is only used for database operations. `$stmt` however should really be `$statement` and `$name` `$username`. `$name` might seem fine, when looking from inside the function, however looking from the outside you see `retrieveUser($name)` and `$name` has become ambiguous. Names are part of the documentation saving a few keystrokes might seem bothersome at first but pays off in the long run.
 - Write documentation. (We'll come to this again later)

Another book that touches a related subject is "*Refactoring: Improving the Design of Existing Code*" by Martin Fowler. Refactoring is the process improving the structure of existing code, without altering its behavior and interface. We've used an informal (and error prone) way of applying a refactoring method called "extract function". Fowler introduces more refactoring methods and explains why testing is essential to the process. 

Both books try to help you improve your code; both by helping you improve yourself. They are on my must read least, so I encourage you to read them. Maybe not know but in half a year or year. They will help you go from functions that are hundreds of lines long to something like this.

    function renderUserTable()
    {
        $userSet = retrieveUsers();
        $userTableData = prepareUserTable();
        return renderTable( $userTableData );
    }

Without me telling you; without further documentation; without thought you should already know what the function does and which steps are required and what those do.
