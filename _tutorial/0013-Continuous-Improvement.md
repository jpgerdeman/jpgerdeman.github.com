Continuous Improvement
=======================

We shouldn't be satisfied with our UserFactory just yet. It contains code
that has to be duplicated for every other Factory we're going to implement
and we have to duplicate alot of code for executing sql statements to
the database.

We could solve this by creating a class BaseFactory from which all Factories inherit. That would be a solution by inheritance. A valid solution. We'll choose to solve this in a different manner, namely composition.

If we look at the methodnames of our factory we notice that we have methods
that concern user instances, but most methods clearly operate on the mysql 
connection. Obviously our factory has two responsibilities. In general classes
and methods/functions should have a single responsibility.

So we'll move the code for setting up the connection back into library/Database.
Simply moving it back though would still leave us with a problem. How would we
handle connecting to another table or database? Creating a database class for
every possible connection would not only be a hurdle, but it would also 
go against the DRY Principle. If we look at our connection methods we notice
that only one actually is about the connection. Most of the other methods
configure the connection. So we need a way to set the connection configuration 
per database call. So we'll have another object called 
library/DatabaseConfiguration we'll create instances of this with the different
settings we have. 

	class Connection
	{
		private $name = null;
		private $username = null;
		private $password = null;
		private $database = null;
		private $host = null;

		public function getName()
		{
			return $this->name;
		}

		public function setName( $name )
		{
			$this->name = $name;
		}

		public function getUsername()
		{
			return $this->username;
		}

		public function setUsername( $username )
		{
			$this->username = $username;
		}

		public function getPassword()
		{
			return $this->password;
		}

		public function setPassword( $password )
		{
			$this->password = $password;
		}

		public function getDatabase()
		{
			return $this->database;
		}

		public function setDatabase( $database )
		{
			$this->database = $database;
		}

		public function getHost()
		{
			return $this->host;
		}

		public function setHost( $host )
		{
			$this->host = $host;
		}
	}


How will we pass the connection configuration to the Database though?
We could pass the configuration to our UserFactory which hands it to the Database
on every call. While this is possible, it would also be awkward. UserFactory
should not be concerned with connecting to the database at all. UserFactory is
for generating users.

But how will we tell the Database to use the proper Connection? 
We could let the the Databse class evaluate the calling class, but that will
make the Database know about the factories. This would mean we'd have to 
change the class with each new factory and any possible child classes.
It would be nice if the database knew what connection it should use, but not how
to connect. So we'll give each connection configuration an unique name, which 
the factories will have to know. The factories will now tell the database 
where to get the information from, not how. Also by giving each connection
configuration a name you can change it without worrying about the factories 
or the database class since they reference the configurations by name.

Since UserFactory shouldn't now a thing about connecting to the database it 
should also not know of any of the
mysqli_* functions. It shouldn't know anything about SQL either, but since most
php developers are also fluent in it anyway, we'll leave that as it is. Otherwise
developers would have to learn another language. In fact if you wish to change
from an SQL database or want to offer other databases you can still parse
the SQL and generate your statements from that.
 
Also did you notice how I had to write database class to reference the class
and distinct it from the actual database. Also connection cofiguration is also 
an unwieldy name. Lets change library/Database.php to orm/library/Driver.php 
and connection configuration to orm/library/Connection.php.
 
Also let's move the model folder to the orm folder as well.
 
OK lets create that orm/library/Driver.php


	class Driver
	{
		private $connections = array();

		public function addConnection( $connection )
		{
			$this->connections[$connection->getName()] = $connection;
		}

		public function select( $stmt, $connectionname )
		{
			$con = $this->openConnection($connectionname);
			$result = mysql_query( $stmt, $con );
			$resultList = array();
			while ($line = mysql_fetch_array($result))
			{
				$resultList[] = $line;
			}
			mysql_free_result($result);
			$this->closeConnection($con);
			return $resultList;
		}

		public function insert( $stmt, $connectionname )
		{
			$con = $this->openConnection($connectionname);
			$isSuccess = mysql_query( $stmt, $con );
			$this->closeConnection($con);
			return $isSuccess;
		}

		private function openConnection( $connectionname )
		{
			$con = $this->connections[$connectionname];
			mysql_connect($con->getHost(),$con->getUsername(),$con->getPassword());
			mysql_select_db($con->getDatabase());
		}

		private function closeConnection($con)
		{
			mysql_close($con);
		}
	}

To instantiate our factory the following code is necessary

	require_once 'library/Request.php';
	require_once 'orm/model/Userfactory.php';
	require_once 'orm/library/Connection.php';
	require_once 'orm/library/Driver.php';

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

That is alot of code we added, without adding new features. So what does the change improve?

For once our database calls are now shorter

        $stmt = 'SELECT * from user WHERE uname = '.$uname;
        return $this->driver->select( $stmt, self::$connectionname );
        
Also implementing global differences can be easily achieved by changing the driver. What could those differences be? Up to now we've been working against MySQL, but what if you have to change to PostgreSQL, MSSQL, SQLlite or some other SQL database? That is easily achievable now. Just write a new driver and you should be good to go (if you haven't used some MySQL specialties).

Actually, if you have to change your database completely, even that is possible now. From outside you call the Factory and get an user object. If you change the Factory to read and write to a file or a webservice, none of your code has to change as long as the factory looks and behaves the same.

By decoupling our code, i.e. reduce the interaction to a fixed set of methods and behaviour, we made it possible to make massive changes with minimal effects on consuming code.