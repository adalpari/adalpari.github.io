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
