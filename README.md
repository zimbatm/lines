Lines - structured logs for humans
==================================

A log format that's readable by humans and easily parseable by computers.

Rationale
---------

All these log lines. Most of them are assembled using methods like
sprintf() which makes them readable to humans but what if you want to
analyse them ? You need custom parsers for every one of these.

Now most of your infrastructure are pieces of code that won't change
much, like apache access logs, you write the parser once and you're good
to go. But what about your application, wouldn't it be nice if you could
just dump some application data, that it formatted nicely and that it
would still be parseable by computers ? That's where Lines enters the
game.

Lines is an opinionated logging that tries to solve this problem. It acknowledges that log lines are read by human but should also be analyzed by software.

Why not JSON ?
--------------

JSON is great for the web but not really readable when compacted on a single line. We also would like to keep more data types if possible like date/time.

Example
-------

```
at=2013-03-17T23:41:08Z app=myapp pid=3452 env=dev name="Token Load" sql='SELECT "tokens".* FROM "tokens" WHERE "tokens"."deleted_at" IS NULL ORDER BY "tokens"."id" ASC LIMIT 1' elapsed=0.941s
at=2013-03-17T23:41:08Z app=myapp pid=3452 env=dev msg="Token not found"
at=2013-03-17T23:41:08Z app=myapp pid=3452 env=dev remote_addr=[127.0.0.1] method=GET path=/ status=400 length=28 elapsed=0.167s
```

Syntax
------

Every log entry is a single UTF-8 encoded line.

```ebnf
# EBNF
line = pair , { space, pair } ;
pair = key, '=', value ;
value = object | list | string | number | boolean | nil | time | litteral ;
object = '{', pair, { space, pair }, '}' ;
list = '[', value, { space, value }, ']' ;
string = '"', ? UTF-8 text where \" \r \n \\ are escaped ?, '"' ;
number = [ "-" ], digit, { digit }, [ ".", { digit } ] ;
unit = number, { letter } ;
boolean = "#t" | "#f" ;
nil = "nil" ;
time =
  digit, digit, digit, digit, "-",
  digit, digit, "-",
  digit, digit, "T",
  digit, digit, ":",
  digit, digit, ":",
  "Z" ;
litteral = ? UTF-8 visible characters ? ;
space = " ";
```

Litterals can be further parsed with language-specific formats and
default to string when no match is found.

`unit` allows you to keep the unit with your numbers. For example `3ms`. If
the language has no native support for the unit it can create a
(number, string) tuple or array instead.

Semantic
--------

The semantic is quite free-format but it could be helpful to agree on a couple of conventions:

* The "at" key contains the time at which the line was created
* The "app" key contain the process name
* The "pid" key contain the process pid
* The "env" key contain the environm where the app was deployed

Library specifics
-----------------

The logger should never raise an exception if it doesn't understand the type that was given to him. One possible fallback it to create a string representation of the type instead but it's up to the library implementer. Just don't raise exceptions.

When the parent log format already includes some of the log data it's allowed to strip some values of the line. Eg: syslog already includes timestamp, app name and PID.

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

