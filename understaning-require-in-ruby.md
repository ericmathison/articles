Understanding `require` and Friends in Ruby
===========================================
####Author: Eric Mathison
####Date: Sat Sep 14 17:06:03 PDT 2013
####Last edited: Fri Sep 27 22:56:01 PDT 2013

Ruby's `require` method is a tool for referencing and executing code that is
not actually contained in the current file. My initial attempts at using
`require` left me a bit confused. Not until later did I realize that Ruby's
`require` method sits atop something more organized and elegant than I had
realized. It turns out that `require`'s main use case is generally not one off,
relative references to other random Ruby files as I had expected. Instead,
`require` is generally used to look in certain pre-determined directory
locations for Ruby libraries. I had previously sort of assumed that a require
in Ruby would be something like the relative references to an image in CSS or a
reference to a JavaScript file in HTML. Not so. In fact, Ruby's `require` seems
to have drawn several concepts from the UNIX environment.

So How Does Ruby's `require` Work Anyway?
---------------------------------------
The best way to think of `require` is in relation to the UNIX `$PATH` variable.
Just by way of a refresher, the `$PATH` variable in UNIX is a list of
directories where executables can be found. So when you type the name of a
program on any UNIX terminal, your computer is looking through the executable
files in the directories specified in your `$PATH` variable. `require` does
something very similar. When, for example, you write `require 'set'` at the top
of your Ruby file, you are telling Ruby to look through a bunch of directories
for a library called set.rb (Ruby's set library).

So where does Ruby look for set.rb? Well, once again, Ruby has something very
similar to UNIX's `$PATH` variable. It is the global variable `$LOAD_PATH` also
sometimes known by it's ugly and undescriptive alias `$:` (which I don't suggest
using by the way--short though it may be). It is an array of directory names
where Ruby looks when it comes across a `require`.

Here's where the analogy falls short however. The UNIX `$PATH` generally
contains a list of bin directories while the Ruby `$LOAD_PATH` contains an array
of lib directories. Well, what on earth are lib and bin directories and what
difference does it make? The lib and bin directories are another Ruby concept
which originally comes from UNIX. A bin directory is a place for standalone
executables. The name bin is a reference to binary files (as opposed to plain
text files) since before the days of ubiquitous scripting languages, executables
needed to be compiled from source into a binary file. Ruby, of course, is an
interpreted language and doesn't need to be compiled but the concept of
standalone executables still applies. A lib directory on the other hand is where
library code goes. This is the code you reference in other code. Referencing
library code is the whole reason we need `require`s in the first place.

When you require a library you are telling that code to be executed any time the
file containing the `require` is executed. The required file is executed only
once. Even if the require is repeated in the same file (why anyone would want to
do this I don't know) it will be executed only once\*. This is because Ruby
checks whether the `$LOADED_FEATURES` variable a.k.a `$"` (not to be confused
with `$LOAD_PATH`) already contains the name of the required file before
executing

The really nice thing about the way `require` works in combination with the load
path is that as long as a library is in the load path, there is no need to
specify the full path or even to know where exactly it is located. You don't
even need to put the '.rb' on the end since that is implicit. All you need to
put is the extensionless name of the file. Personally, I think this makes for
some very clean looking code.

It is important to also realize that in your application, there is generally no
need to add directories to the load path manually as this is the environment's
job. I mentioned above that the `$LOAD_PATH` contains an array of lib directory
paths. RubyGems will, for example, automatically add the lib directory of gems
installed on your computer to the load path. Well, that's actually only mostly
true. If you *actually* run a Ruby script that just outputs `$LOAD_PATH` without
doing anything else, you won't see every single installed gem's load path
listed. This is because RubyGems lazily loads Gem's lib directories. So if you
want to see your gem's lib directory listed in the load path you have to
actually require it first. But things work essentially the same as if every
single gem's lib directory was on the load path.

Slightly unrelated to the topic of this article but perhaps also interesting to
note is that RubyGems will also add a gem's bin directory to the UNIX `$PATH`
variable so that a gem's executables can be run from the terminal.

Understanding `load`
--------------------
The `load` method is very similar to `require`. The main differences is that it
will run the referenced code as many times as `load` calls it. As I mentioned
above, `require` only runs the first time.

`load` must be given the '.rb' file extension because it won't be inferred like
it was with `require`.

All in all, I would say that `load` has a fairly small amount of usefulness.
Probably it's main use case is re-running a file from a Ruby REPL (like pry or
irb) after making some edits.  It's also very helpful when you need to load ruby
files that do not have a .rb extension since `require` will automatically look
for files ending in .rb.  This situation can occur when trying to load ruby
script files or executables which tend not to have an extension.

Understanding `require_relative`
--------------------------------
`require_relative` is essentially what I had originally thought `require` would
be used for. If you want to reference a file relative to another file, not
treat it like a library, and bypass the load path, this is the tool for you.

In order to understand the need for `require_relative`, we must understand a bit
more about `require`. So far, the only use I've mentioned for `require` is
executing code on the load path (code that would be considered a library).
That's for good reason. That's what it is good for and you probably shouldn't be
using `require` for anything else.

In the past (Ruby 1.8 and earlier), if you wanted to do a one off `require`
without managing the load path, the best way would have been to generate the
full path to that file and pass it as an argument to `require`. It would look
something like this:

    require File.expand_path('my_file', File.dirname(__FILE__))

`File.expand_path` gives the full path to the file specified in the first
argument. Specifying the second argument essentially says, look for my\_file.rb
relative to such and such directory. `File.dirname(__FILE__)` returns the
directory path for the current file (that this line was written in). So put
together, `require` is getting passed the full path to my\_file.rb which is the
file sitting in the same directory in which this line was written. Another way
of writing the same thing looks like this:

    require File.expand_path('../my_file', __FILE__)

The difference here is that the "directory" we are passing in the second
parameter isn't actually a directory. It is just the current file. Fortunately,
`File.expand_path` doesn't care since theoretically, we could make a directory
called `'current_file.rb'` strange as that would be. Assuming I'm running this
line from my home directory, without the `'../'` before the file name in the
first parameter, the string we are passing to `require` would look something
like this: `'/home/eric/current_file.rb/myfile.rb'`. From `File.expand_path`'s
perspective, adding the `'../'` brings us up a directory but from our
perspective, it simply causes current\_file.rb to be removed from the path. So,
with `'../'` back in place, the path would expand to `'/home/eric/my_file.rb'`.

Let me back up and explain why passing `require` the full path to a file would
even be necessary. When I write something like `require '../my_file.rb'`, that
path is relative to the current working directory of the process executing this
line NOT necessarily the file it was written in. That's important and it means
that depending on what directory you are in when you run `ruby current_file.rb`,
your code may or may not be able to find your relative reference to my\_file.rb.
Yikes!

Enter `require_relative` stage left. `require_relative` does essentially what
the above two examples using `require` do except with the added benefit of a
clean syntax. So instead of needing that massive monstrosity, all that
`require_relative` needs to do is `require_relative 'myfile'` and my file will
be reference relative the file containing that line (current\_file.rb in our
example).

Also note that even if you are running current\_file.rb from it's containing
directory (so that the current working directory of the process is the same as
the location of current\_file.rb), it is not possible to do something like
`require 'myfile'` although `require './myfile'` would work. This is because
since Ruby 1.9.2, the current working directory is no longer part of the load
path. In the UNIX world, this is considered a good security practice since it
avoids the issue of accidentally running an unexpected script by simply
executing a program from a directory that just happens to have a file with the
same name. Since `require_relative` is requiring relative to the current file
instead of the current process, it doesn't have this security issue. That's one
more reason to use `require_relative` for relative requires.

One last note about `require_relative`: Have you ever noticed that using
`require_relative` doesn't work on a Ruby REPL like pry? Think about it. If
`require_relative` is requiring relative to a file, which file would that be in
a REPL? Well, there isn't really a file that fits that bill. Your alternatives
in this situation are to use `load` or `require`. If you use `require` though,
make sure to remember the leading `'./'` or `'../'` and don't do this in
production code!. And if you ever want to determine what the current working
directory of your Ruby process is, you can always output `Dir.pwd`.

Conclusion
----------
So, we have seen that `require`, `require_relative` and `load` all fulfil
different use cases when used as intended. `require` is generally for
referencing libraries, `require_relative` is for making one off local references
within an application (typically deployed applications, not within libraries)
and about all `load` is good for is re-loading a script in a REPL. Well, that's
how I see it anyway :).


\* Prior to version 1.9, Ruby would only do a string comparison between required
files. So if you require a file using the full path (this is typically what
happens when requiring a file from the load path) and then require it again with
a relative path (`require './myfile'` for example), it could inadvertently be
run twice. Since Ruby 1.9 however the path to a required file is expanded to the
full path before attempting to add it to the `$LOADED_FEATURES` variable. At
least, that's what it appeared like from my experimentation.
