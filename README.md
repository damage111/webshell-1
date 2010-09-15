Webshell: A console-based, JavaScripty HTTP client utility
==========================================================
by Evan Haas and Sean Coates
----------------------------

Includes tab completion, history, context persistence and cookies.

This is best demonstrated.

NOTE: a recent version of Node.js (>=0.1.104) is required (for improved
readline-like behaviour).

Simple HTTP requests
--------------------

    sarcasm:~/src/webshell$ node shell.js 
    webshell> GET http://google.com/
    HTTP 301 http://google.com/
    webshell> $_.headers
    { location: 'http://www.google.com/'
    , 'content-type': 'text/html; charset=UTF-8'
    , date: 'Sun, 08 Aug 2010 22:38:23 GMT'
    , expires: 'Tue, 07 Sep 2010 22:38:23 GMT'
    , 'cache-control': 'public, max-age=2592000'
    , server: 'gws'
    , 'content-length': '219'
    , 'x-xss-protection': '1; mode=block'
    , connection: 'close'
    }
    webshell> $_.headers.location
    'http://www.google.com/'
    webshell> $_.follow()
    HTTP 302 http://www.google.com/
    webshell> $_.headers.location
    'http://www.google.ca/'
    webshell> $_.follow()
    HTTP 200 http://www.google.ca/
    webshell> $_.raw.substring(0, 50)
    '<!doctype html><html><head><meta http-equiv="conte'
    webshell> ^D

Print HTTP response
--------------------
    The HTTP response itself can optionally be printed by setting
    $_.printResponse.  If $_.printResponse is a function, it will
    be called with a single argument - the response object.  Return
    true/false depending on whether the response should be printed.
    If $_.printResponse is not a function, its truth value will 
    determine whether responses are printed.  By default 
    $_.printResponse is a function which returns true for JSON
    content-type responses and false for others.
    
Store HTTP response
-------------------

    webshell> result = $_.get('http://fictivekin.com')
    GET http://fictivekin.com
    webshell> HTTP 200 http://fictivekin.com
    webshell> result2 = $_.get('http://www.google.com')
    GET http://www.google.com
    webshell> HTTP 200 http://www.google.com
    webshell> result.headers['content-type']
    'text/html'
    webshell> result2.headers['content-type']
    'text/html; charset=ISO-8859-1'
    
    
JSON processing
---------------

    sarcasm:~/src/webshell$ node shell.js
    webshell> GET http://twitter.com/users/coates.json
    HTTP 200 http://twitter.com/users/coates.json
    webshell> $_.json.name
    'Sean Coates'
    webshell> ^D

Save and load contexts
----------------------

    sarcasm:~/src/webshell$ node shell.js
    webshell> GET http://twitter.com/users/coates.json
    HTTP 200 http://twitter.com/users/coates.json
    webshell> $_.saveContext("twitter-coates")
    Saved context: twitter-coates
    webshell> ^D

    sarcasm:~/src/webshell$ node shell.js
    webshell> $_.json // empty
    webshell> $_.loadContext("twitter-coates") // pick up where we left off
    Loaded context: twitter-coates
    webshell> $_.json.name
    'Sean Coates'

HTTP auth
---------

    sarcasm:~/src/webshell$ node shell.js
    webshell> GET http://coates:notmypassword@twitter.com/users/coates.json
    HTTP 401 http://coates:notmypassword@twitter.com/users/coates.json
    webshell> GET http://coates:mypassword@twitter.com/users/coates.json
    HTTP 200 http://coates:mypassword@twitter.com/users/coates.json
    webshell> ^D

Cookies
-------

    sarcasm:~/src/webshell$ node shell.js 
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> $_.raw
    'You have visited this page 1 times.'
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> $_.raw
    'You have visited this page 2 times.'
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> $_.raw
    'You have visited this page 5 times.'
    webshell> $_.saveContext("cookiedemo")
    Saved context: cookiedemo
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> $_.raw
    'You have visited this page 6 times.'
    webshell> $_.loadContext("cookiedemo")
    Loaded context: cookiedemo
    webshell> $_.raw
    'You have visited this page 5 times.'
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> $_.raw
    'You have visited this page 6 times.'
    webshell> $_.cookies.get("files.seancoates.com")
    { cookiecounter: 
      { http_only: false
      , key: 'cookiecounter'
      , value: '6'
      , expires: Sun, 15 Aug 2010 23:11:33 GMT
      , path: '/'
      , domain: 'files.seancoates.com'
      }
    }
    webshell> ^D

    sarcasm:~/src/webshell$ node shell.js 
    webshell> $_.loadContext("cookiedemo")
    Loaded context: cookiedemo
    webshell> $_.cookies.get("files.seancoates.com").cookiecounter.value
    '5'
    webshell> GET http://files.seancoates.com/cookiecounter.php
    HTTP 200 http://files.seancoates.com/cookiecounter.php
    webshell> $_.cookies.get("files.seancoates.com").cookiecounter.value
    '6'
    webshell> ^D

HTTP verbs
----------

    sarcasm:~/src/webshell$ node shell.js 
    webshell> GET http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    HTTP 200 http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    webshell> $_.json.get
    { one: '1', two: '2' }
    webshell> $_.json.verb
    'GET'
    webshell> $_.requestData = { three: 3, four: 4 }
    { three: 3, four: 4 }
    webshell> POST http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    HTTP 200 http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    webshell> $_.json.post
    { three: '3', four: '4' }
    webshell> $_.requestData = "five=5&six=6"
    'five=5&six=6'
    webshell> POST http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    HTTP 200 http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    webshell> $_.json.post
    { five: '5', six: '6' }
    webshell> $_.json.verb
    'POST'
    webshell> ^D
    sarcasm:~/src/webshell$ echo "testing some PUT data" > ~/test.txt
    sarcasm:~/src/webshell$ node shell.js 
    webshell> $_.requestData = "/Users/sean/test.txt"
    '/Users/sean/test.txt'
    webshell> PUT http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    HTTP 200 http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    webshell> $_.json.verb
    'PUT'
    webshell> $_.json.input
    'testing some PUT data\n'
    webshell> DELETE http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    HTTP 200 http://files.seancoates.com/jsonifyrequest.php?one=1&two=2
    webshell> $_.json.verb
    'DELETE'
    webshell> ^D

