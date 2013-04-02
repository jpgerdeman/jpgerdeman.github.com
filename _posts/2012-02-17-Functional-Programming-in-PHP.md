---
layout: post
title: "Functional Programming and PHP"
---
Functional Programming and PHP
I'm not suggesting to add new paradigms to PHP. I actually believe functional paradigms like invariant variables would destroy PHP.

Nonetheless it never hurts to look at other languages and paradigms to continually improve oneself. I've been looking at functional programming lately and came across the design pattern wiki at c2.com. Today I want to talk about FunctorObjects.

Basically FunctorObjects are objects, that can be invoked like functions. PHP has the magic __invoke function, which allows

    class Functor
    {
        protected $_sum = 0;
         
        public method __invoke($arg1)
        {
            $this->_sum+=$arg1;
            return $this;
        }
     
        public function __toString()
        {
            return $this->_sum;
        }
    }
     
    $f = new Functor();
    echo $f; // 0
    echo $f(2); // 2
    echo $f(4); // 6

But what would I need objects like that for? If you have classes that have lots of methods, but only one of them is public (algorithms for example) this allows for a far more concise invocation.

Sadly in PHP FunctorObjects are not valid callbacks. But if you define higher level functions, you can allow them to accept such objects. Your code then does not have to discern between functions and objects. Filters are a nice example of this.

    class MyMail
    {
        public function filter($filter)
        {
           $filteredMail = array();
           foreach( $this->_mails as $mail)
           {
               if($filter($mail))
               {
                   $filtereMail[] = $mail;
               }
           }
            
           return $filteredMail;
        }
    }

The above code does not care if $filter is a simple function, closure or a complex object, as long as a mail is accepted and a boolean value returned.


I'll try to post more unusual patterns soon.