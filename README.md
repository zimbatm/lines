Lines - structured logs for humans
==================================

A log format that's readable by humans and easily parseable by computers.

STATUS: WORK IN PROGRESS
========================

Things are starting to stabilize.

The unit separator is probably going to change. The number format could get
some human-readable love. There's still a lot of specification needed
regarding Unicode and non-visible characters.

The rest should be pretty much stable.

Rationale
---------

All these log lines. Most of them are assembled using methods like
sprintf() which makes them readable to humans but what if you want to
analyse them ? You need custom parsers for every one of these.

Now most of your infrastructure are pieces of code that won't change
much. For example apache access logs. You write the parser once and you're
good to go. But what about your application. Wouldn't it be nice if you could
just dump some application data, have is still readable, and that it
would still be parseable by computers ? That's where Lines enters the
game.

Lines is an opinionated logging that tries to solve this problem. It
acknowledges that log lines are read by human but should also be analyzed by
software.

Why not JSON ?
--------------

[JSON](http://json.org/) is great for the web but not really readable when
compacted on a single line. We also would like to keep more data types if
possible like date/time.

Example
-------

```
at=2013-03-17T23:41:08Z app=myapp pid=3452 env=dev name='Token Load' sql='SELECT "tokens".* FROM "tokens" WHERE "tokens"."deleted_at" IS NULL ORDER BY "tokens"."id" ASC LIMIT 1' elapsed=0.941:s
at=2013-03-17T23:41:08Z app=myapp pid=3452 env=dev msg='Token not found'
at=2013-03-17T23:41:08Z app=myapp pid=3452 env=dev remote_addr=[127.0.0.1] method=GET path=/ status=400 length=28 elapsed=0.167:s
```

Syntax
------

Every log entry is a single UTF-8 encoded line.

The following syntax is described using the
[EBNF](https://en.wikipedia.org/wiki/EBNF) syntax:

```ebnf
line = pair , { space, pair } ;

space = " "

pair = ( string | literal ), '=', value ;

value = object
      | list
      | string
      | literal ;

object = '{', ( "..." | "" | pair, { space, pair } ), '}' ;

list = '[', value, { space, value }, ']' ;

string = "'", ? UTF-8 text where \' \r \n \\ are escaped ?, "'"
       | '"', ? UTF-8 text where \" \r \n \\ are escaped ?, '"' ;

literal = ? UTF-8 visible characters ? - [ "=" | "}" | "]" | space ] ;
```

A literal in a value position can then be further down-parsed into more
specific values. These are officially supported types:

```ebnf
boolean = "#t" | "#f" ;
nil = "nil" ;
number = [ "-" ], digit, { digit }, [ ".", { digit } ] ;
time =
  digit, digit, digit, digit, "-", digit, digit, "-", digit, digit, "T",
  digit, digit, ":", digit, digit, ":", digit, digit, "Z" ;
unit = number, ":", literal ;
```

Generator notes
---------------

Strings can be represented as literals if they don't contain any of the
forbidden characters. To avoid confusion during parsing it is also recommended
to avoid the literal form if the string contains ":" or other characters

If you want the generator can also generate other forms of literals that are
language specific. A parser who doesn't recognise the literal will parse it as
a string.

To avoid circular generation keep track of the depth of the tree. When passing
a certain level, replace the content of arrays with "[...]" and of objects
with "{...}"

Parser notes
------------

`unit` allows you to keep the unit with your numbers. For example `3:ms`. If
the language has no native support for the unit it can create a (number,
string) tuple or array instead.

The "{...}" format encountered when the object is too deep should be parsed as
`{"...": ""}` in JSON notation.

Semantic
--------

The semantic is quite free-format but it could be helpful to agree on a couple
of conventions:

* The "at" key contains the time at which the line was created
* The "app" key contain the process name
* The "pid" key contain the process pid
* The "elapsed" key might contain some time measurement (with unit!)

Library specifics
-----------------

The logger should never raise an exception if it doesn't understand the type
that was given to him. One possible fallback it to create a string
representation of the type instead but it's up to the library implementer.
Just don't raise exceptions.

When the parent log format already includes some of the log data it's allowed
to strip some values of the line. Eg: syslog already includes timestamp, app
name and PID.

Log levels are overrated.

Implementations
---------------

Submit a pull-request to add yours.

* [lines-ruby](https://github.com/zimbatm/lines-ruby)

Acknowledgements & Thanks
-------------------------

* Lots of inspiration from the scrolls project: https://github.com/asenchi/scrolls
* @kr for the another gist on the format:
https://gist.github.com/kr/0e8d5ee4b954ce604bb2
* The lograge project: https://github.com/roidrage/lograge
* [logfmt](https://github.com/kr/logfmt)
