# json-examples

Examples of JSON parsing / generation and http requests in various languages

# Solutions Explored

  0. [jQuery](http://jquery.com) (in Browser, all the hard stuff is handled)
  0. [NodeJS](http://nodejs.org) (Best HTTP and JSON support)
  0. [Ruby](http://ruby-lang.org)
  0. [Python](http://python.org)
  0. [GoLang](http://golang.org)
  0. [luvit](http://luvit.io) (lua)
  0. C#
  0. Java
  0. curl (exec from any language)
  0. bash
  0. C

ProTip: Use an Application Language (NodeJS, Python, Ruby) to do all of your HTTP / JSON work coupled with some sort of message passing (such as packing a struct) if you need to use a Systems Language (C, Java, C#). Or just do it all in Go. That would be the best language.

# IMPORTANT

The parser you choose **MUST** be configureable meet the following criteria:

  * IGNORE unexpected and unused fields (new fields may be added at any time)
  * ACCEPT `null` as a non-value for any field type

Additionally:

  * when a number is encountered where a string was expected, the number should be treated as a string
  * if an expected field does not exist, it should be treated the same as if the value were `null`

# Examples

The eventual goal is to have examples for all of the following in each language:

  * HTTP GET request
  * HTTP POST request
  * JSON parsing (decoding / deserializing)
  * Iterating over objects and arrays
  * JSON stringifying (encoding / serializing)
