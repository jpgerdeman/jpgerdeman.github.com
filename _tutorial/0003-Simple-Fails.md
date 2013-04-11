Even the Simplest Things are Full of Flaws
=============================

Try calling the code from step 2 without providing a name. As you see the code
results in an error message. We'll have to check our query parameter and
supply a default value.

## Ternary Operator and If-Statements

Of course you can use a simple if-statement to check the query parameter.

	if( isset( $_GET['name'] )
		$name = $_GET['name'];
	else
		$name = 'world';

This is where we'll start talking about coding conventions. Personally I have a problem with the above code. Omitting the enclosing braces may save a few lines of code, but will make your code harder to read.
Often when I'm reading code, looking for a bug, I unconsciously scan the code first. Using braces makes it easier to scan the code for chunks of decisive codes. Also it will usually give if statements, loops and so forth the room they deserve.

And since I'm known to often contradict myself. The above if statement is actually not very important in the long run. Granting it several lines will potentially only distract from more important code.
PHP provides another way of writing that statement using the ternary operator.

The syntax is

	if?then:else;

Applied to above if-statement

	$name = isset( $_GET['name'] ) ? $_GET['name'] : 'world';

There is an even shorter way to write it

	ifthen?:else

	$name = $_GET['name']?:'world';

There is a difference between the ternary operator and an if-statement. PHP is interpreted line for line. This means, that both sides of the ternary operator are evaluated before the operator executes. Whereas in an if-statement only those lines of code that apply are evaluated.

This makes the ternary potentially slower and potentially faster than an if-statement.

<blockquote>
The importance of your code should be reflected in the vertical space (lines) it occupies.
</blockquote>

As a rule of thumb, the importance of your code should be reflected in the lines of code. So an important (to the reader) decision in code, should occupy more vertical space then a trivial one.

### Ternary Misuse

Misuse of the ternary is punishable.

Use of the ternary to hide a major decision will earn you a sharp remark by a senior developer.

Craming more logic into the ternary will result in that piece of logic being printed out so everyone can have a good laugh at your expense. That is after someone comes running into your office fuming and yelling at you.

Using a ternery within a ternary will result in your fingers being cut of and force fed to you. You lose. No second chances. You're a goner. They'll humilate you, whip you and fire you!