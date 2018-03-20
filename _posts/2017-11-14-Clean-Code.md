---
layout: post
title: Let's clean the code!
---

One of the multiple _must_ that EVERY developer should do, is read: __Clean Code__ book from _Robert C. Martin_.

In that book, you will find basic and advanced rules and tips, to create good quality code in terms of readability. Easier to understand and mantain than a messy ugly code. These tips not only will make you a better developer but algo will save you great amount of time.

In order to have a "cheat sheet" about _Clean Code_ rules, I'm going to sumarize and write some key points. 

If you are reading this without have readed the book, run to your closest library!

## 2. Meaninful names
- Reveal intention or use with the name.
- Avoid use the encode type (_string, integer.._)
- Avoid _m_\_ prefix. Use the colour code of your _IDE_.
- Use one single word per concept. Don't use _remove_ and _delete_, _fetch_ and _retrieve_, and so on.
- Classes: should be named with a noun or phrase.
- Methods or functions: should be named with verb. The arguments should be nouns. That way a method will be _verb(noun)_

## 3. Functions
- Should be __small__. Perfect method is smaller than 5 lines. Yes 5 lines.
- If use blocks (_if, for.._) try to call methods inside instead of operation code. That's good for size and readaility.
- Each method should do only ONE thing.
- Do something or Answer something, not both.
- Use only one level of abstraction per function. (E.g. If  method is managging a _list_, don't modify _objects_ on it. Better call a new method for that)
- Try to write methods as a book writting. One method call another that it's immediately below. Avoid calls without any order in the code.
- Avoid side effects. Do only what the method is suppoused to do.
- Avoid output arguments. Don't use an argument to return information.
- Avoid return _ErrorCodes_. Better use _Exceptions_
- If use exceptions, try to call methods inside instead of operation code.
- Arguments:
- Avoid to use >3 arguments. Arguments are hard to remember in calls, adds complexity, easier to introduce bugs, adds more test variants..
- Avoid use flags like _booleans_ in arguments. It's very confusing. Better create two methods: one for true and another one for false.

## 4. Comments
- A comment does not fix bad or not-understandable code.
- Good code can be understandable itself. Avoid redundant comments.
- If you use comments, explain _why_ not _how_.
- By the time, comments can get outdted, because in revisions we tend to change the code but not the comments.
- Comments tend to add noise in the code.
- Do not comment code, better remove it. Other developers won't understand why it's commented and they directly wont read it.
- Try to __avoid comments__.

## 5. Formatting
- Try to have small files. __<500 lines__.
- Follow the __Newspaper Metaphore__. Your code should be readed from top to bottom. Avoid big jumps in the code structure.
- Use blank lines to separate concepts, blocks and so on.
- Related concepts or operations should be close one from each other (_related methods, related operations.._)
- Order your methods in the way that above methods call below methods (_Newspaper Metaphore_). And go from high to low level.
- Lines should be __<120 characters__.
- Surround operators with space except for diferenciate linked operations. _E.g: b\*b - 4\*a\*c_
- Use indentation to better visualize blocks.
- Avoid colapsing blocks with one line. Use new line, indentation and braces for each block.
- When you need to break one line, break by operators and put them in new line. _E.g: && on ifs, + on string breaks.._

## 6. Objects and Data Structures
- Don't expose details of your objects. Try to be as muchs __abstract__ as possible. __Hide the data and expose methods__ to operate with it. _E.g. List, Map.._
- Use polymorphism or procedural call depending on the context. 
    - Polimorphysm: all objects contain it's own methods. If you add a new object, the other will be unaffected, but if you add a new method, all objects will be affetced.
    - Procedural: Single method handles all object posibilities. If you add a new method, all objects will be unaffected, but if you add a new object, all methods will be affected.
- _Law of Demeter_: use only methods and objects you directly know. Avoid to interact with classes or methods out of the scope. _Talk with friends, not with strangers_.
- Use _DataTrasfer_ objects to handle all the data related with an object. The object will only have, private fields, constructor and getters/setters.

## 7. Error Handling
- Use _Exceptions_ instead of return error codes.
- Use __Unchecked Exceptions__. Checked Exceptions add noise to the code and implies to add _throws_ declaration to every single method which calls the one with the exception but don't handle it.
- Use informative names and group (wrap) similar exceptions into one. Use the message to add useful information.
- Avoid add _normal flow_ code inside the catch blocks. These blocks should be used for only exception puroses.
- __Don't return null__. Return null only adds posibilities of generates _Null Pointer Exceptions_. Or force the developer to add _null_ chechs. Intead of _null_, use empty collections, empty strings and so on.
- __Don't pass null__. Pass null as an argument is even worse, because theorically, one don't know what the method is going to do with it.

## 8. Boundaries
- When use a third part library, try to write all access to it in single part of your code. Avoid call it everywhere, because it can cause dependency issues. You can use mappers, wrap the access with your own class, utils, and so on.

## 9. Unit Tests
- Use TDD. (Test Development Driven)
- Apply to test code the same clean rules you apply to production code.
- Focus on readability. It's really important to understand a test with a single fast read.
- Use __Given, When, Then__ blocks (_Build-Operate-Check_).
    - Given: set the conditions for the test.
    - When: make the actions necessary to run the test.
    - Then: check that everything works as expected (_assert_)
- Try to use only one assert per test. If you write more than one assert, probably your test class is testing two different things and can be splitted in two.
- Only one concept per test. Related with last point, each test should only test one single feature at a time.
- Follow __F.I.R.S.T.__
    - Fast: tests should be fast
    - Independent: tests should not depend on each other.
    - Repeatable: tests should be repeatable in any environment.
    - Self-Validating: tests should have a boolean output.
    
## 10. Classes
- Follow Java convention when write a class. _I.e. public/private static constants, private static variables, private instance variables, public methods (private methods will be writt follow the newspaper rule)_
- Use a name which describe the class intent.
- Encapsulate the class data as private variables when possible.
- Keep it small.
- Follow the __Single Responsability Principle__. Each class should be written to handle only one concept.
- Cohesion is a grade of how methods are using the class variables. When there are several methods that are not using the variables, probably they can be extracted into another class.

## 11. Systems
- As a city, a software system starts from a small one and grows up by the time. That's why is really important separate concepts inside it.
- Clearly separate the construction of object from the use of it. Best approach is _Dependency Injection_.
- Create modules for group the features and not mix them.

## 12. Emergence
- A well designed software probably follows _Kent Beck's_ rules:
    - Runs all the tests: passing all the test of a testable software, means it is working as expected. _Also helps with refactoring_
    - No duplication: duplications means additional works, higher bugs risk and unnecesary complexity.
    - Expresive: the coude should express itself the intent of the developer who program it.

## 13. Concurrency
- Separate the code in charge of concurrency from application code.
- Protect shared resources with _synchronized_ blocks.
- Minimise thenumber or shared resources. It will help with complexity and bugs.
- Test the reliability of the coe without concurrency first.
- Do not ignore non-reproducible bugs (race conditions). They could appear at any moment.

## 14. 15. 16.
- Refactoring step by step of some complete modules and programs.

## 17. Smells and Heuristics
- Concrete examples of code smells and some ways to refactor them. This chapter talks about some concrete practises writeng along the book.
