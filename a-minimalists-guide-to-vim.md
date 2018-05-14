A Minimalist's Guide to Vim
===========================
#### Author: Eric Mathison
#### Date: May 14, 2018
#### Last edited: May 14, 2018


People love to customize Vim. That's great to an extent. It's terrible when it means loosing some of the really great advantages of doing things "The Vim Way"®. By all means, try things out, mix it up, and see what works for you. This is the story my journey through customizing Vim, what I learned in the process, and why I've decided to go back to the basics.

Get in, Get out
---------------
Emacs users love to live in Emacs. This is not the Vim way. Vim is known for getting in and getting out. I've noticed a trend however of doing more and more from inside Vim. Examples include the ctrlp plugin (a fuzzy file finder), running tests from inside Vim, and even handling version control à la Fugitive (a plugin for git). This workflow is essentially treating Vim like Emacs. But if Emacs were a tank, Vim would be a dirtbike. As a Vim user, I prefer to let Vim do one thing and do it well—editing text.

The Journey
-----------
I remember watching someone use ctrlp for the first time. I was amazed at how fast they went from file to file—all with just a few keystrokes. I joined the bandwagon. After a while though it dawned on me that finding files wasn't really the domain of an editor. I thought, there must be a way to fuzzy match a file and simply pass it to Vim as in: `vi some/file.rb`. In fact there was! It’s called fzf (https://github.com/junegunn/fzf). At some point though, I realized if I wanted to go to the file I just edited I didn't really need any special plugins or fancy whizbangs. It's as simple as typing `vi<enter>` and then hitting `<ctrl-o>` twice. All my history is at the tip of my fingers. `<ctrl-o>` follows my steps backward and `<ctrl-i>` jumps instantly forward through anything I've been editing. Then again, with a shell like Fish that seems to know what file I want to edit before I do and gives file completion hints right out of the box, even this seems like too much typing sometimes.

Learn Useful Defaults, Customize When Critical
----------------------------------------------
Being able to use the same Vim setup just about anywhere is pretty incredible. Exiting Vim is one of those critical places where customization is key. I remap `<leader>q` and `<leader>w` to `ZQ` and `ZZ` respectively. So on my setup, to exit Vim without saving, I type `\q`. Two characters and I'm gone. I hate the shift key. Just think, `:q!` requires hitting shift twice! With another key awkwardly placed in between! `:wq` is a bit more reasonable so I often just stick with the default there but it really shouldn't take an eternity to open and close Vim. Get in, get the job done, and get out. That's the Vim way.
