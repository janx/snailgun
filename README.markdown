Snailgun
========

This is a highly experimental, proof-of-concept piece of code which preloads
libraries into a Ruby process, and then forks that process whenever a new
Ruby interpreter is required. The idea is to reduce the slooooow startup of
Ruby apps which require a large number of libraries.

Example 1: standalone
---------------------

    $ time ruby -rubygems -e 'require "active_support"' -e 'puts "".blank?'
    true

    real	0m2.345s
    user	0m1.356s
    sys	0m0.216s
    
    $ bin/snailgun -rubygems -ractive_support
    Snailgun starting on /tmp/snailgun21574
    $ time bin/fruby -e 'puts "".blank?'
    true
    
    real	0m0.121s
    user	0m0.032s
    sys	0m0.004s
    $ exit
    Snailgun ended
    $ 

Example 2: inside a rails app
-----------------------------

    $ rails testapp
    $ cd testapp
    $ ~/git/snailgun/bin/snailgun --rails --rake
    Snailgun starting on /tmp/snailgun21717
    $ time script/runner 'puts "1+2"'
    1+2

    real	0m6.654s
    user	0m4.652s
    sys	0m0.640s
    
    $ time ~/git/snailgun/bin/fruby script/runner 'puts "1+2"'
    1+2

    real	0m2.710s
    user	0m0.036s
    sys	0m0.012s

    $ time ~/git/snailgun/bin/frake -T
    ...
    real	0m0.782s
    user	0m0.028s
    sys	0m0.012s
    $ exit
    Snailgun ended
    $ 

Bugs and limitations
--------------------
For some reason, `frake test:units` doesn't seem to run any tests :-(

`fruby script/console` stops the process when it reads from stdin. You need
to type `fg` to continue.

The ruby child process doesn't have a supervisor parent, which means if
it dies using 'exit' we don't get the status code

The environment is not currently passed across the socket to the ruby
process. This means it's not usable as a fast CGI replacement.

Not as much stuff is preloaded as I would like.

Licence
-------
This code is released under the same licence as Ruby itself.

Author
------
Brian Candler <B.Candler@pobox.com>