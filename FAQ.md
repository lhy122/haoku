**Why does my program with colored output not have color under Ninja?**

Because Ninja runs commands in parallel, it buffers their output to prevent it from interleaving.  Commands will detect that their output is not directed to a terminal, assume they're going to a log file, and turn off their colored output.  To fix this, pass flags to the subcommand (like `-fcolor-diagnostics` to clang) to force color on. Ninja itself will detect if it's writing to a terminal color escape codes from commands if it isn't.

Ninja could use pseudo ttys to work around this further, but different systems have different limits on the number of ptys available.  I also worry that commands like `svn up` would assume they can interact with the user.

**Does Ninja support Python 3?**

Using Ninja for your project is language-agnostic.  Internally Ninja uses Python 2 to build itself.  It is my belief that systems where `/usr/bin/python` is not Python 2 are broken (given that we can't go back in time and add a `/usr/bin/python2` on existing systems; see also [this discussion](https://mailman.archlinux.org/pipermail/arch-general/2011-December/023344.html)).  However, I've grudgingly accepted patches to make the code run under Python 3 when they're not horrifically ugly.

**Why doesn't Ninja go memory-resident between builds to improve speed / why doesn't Ninja use disk-access monitoring to track when files change?**

The original design plan of Ninja included this.  But ideally Ninja should also be fast to start the first time, before it's had a chance to listen for incremental updates.  Upon implementing this first start, I found it was fast enough to just run every time.

Over time Chrome (and other projects) have grown larger and it might make sense to revisit this decision.  I am still wary of it due to additional complexity: a client-server protocol, propagating incremental changes across the dependency graph, policies about when to quit.  It's not impossibly hard but Ninja excluding tests and comments is only around 5k lines of code, and it's nice to stay simple.