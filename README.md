# labbox - shell script for Linux and Mac to update a remote host's files.

Project home:
https://github.com/fordsfords/labbox

License:
CC0 (equiv to public domain). Any use, no attribution required.

Labbox is a variant of "poor-man's dropbox",
which originated as a script by Sahir Hoda.
I evolved it to be better for Linux and MacOS.

Unlike DropBox, this is not a 2-way-sync.
It runs on your laptop and detects changed files within a parent directory
and all child directories under it (using fswatch) and copies the changed
content to the lab machine (using rsync).

Any changes you make on the lab machine will *not* show up on your pc,
and are subject to being overwritten the next time rsync runs. I.e. if
you have files X and Y, and you edit Y on the lab machine, and X on your
laptop, updating X will trigger rsync to run, and rsync will see that both
X and Y differ, and will overwrite both files. So your Y changes are lost.
(However, if you have file Z on the lab machine that does not exist on your
laptop, rsync will *not* delete Z.)


# DEPENDENCIES

Requires "fswatch", which is available for Mac via
[brew install fswatch](https://brew.sh/)
and available for Ubuntu Linux via "apt install fswatch".

# USAGE

````
labbox remote_path [additional_options]
````
Where:
* "remote_path" - directory path on remote host where the
files should be copied to.
* "additional_options" - extra "rsync" options.

All files in the current directory tree will be copied (subject to
exclusions; see below).

The script will continue to run until killed (typically with "ctrl-C").
After the initial sync, it will wait for changes, and re-copy changed files.


# REMOTE HOST

I put the name of my remote host right inside of "labbox".
But I also allow it to be overridden with the environment
variable "LABBOX_HOST".


# RSYNC OPTIONS

Labbox normally runs rsync with the options "-azq".
````
 -a: archive is a meta option that turns on "-rlptgoD"
   -r: recursive; -l: copy symlinks as symlinks; -p: permissions
   -t: preserve time; -goD: preserve group, owner, and device files
 -z: compress during transfer
 -q: quiet
````

If there are additional options desired, you can add them after the
remote_path on the command line.
These are added to the default options.

Alternatively, you can define the "LABBOX_OPTS" environment variable to
override the default options with a completely different set.


# EXCLUSIONS

Frequently one does not wish to copy various types of files.
For example, if I build and test on my laptop, I generally don't want ".o" objects
and ".log" output files copied to the remote host.
Both rsync and fswatch support the "--exclude" option to prevent files
from being processed.

Unfortunately, rsync and fswatch use different matching algorithms for their
"--exclude" options:

* rsync: uses [Unix-like glob patterns](https://en.wikipedia.org/wiki/Glob_(programming)#Unix-like)
* fswatch: uses [regular expressions](https://en.wikipedia.org/wiki/Regular_expression)

The labbox tool in this package defaults some standard exclusions that I use a lot,
and also supports overriding those defaults using the files:
* labbox.exclude - if present, one glob pattern per line.
* labbox.fsexclude - if present, one regular expression per line.

One note about fswatch exclusions: let's say I have ".*\.log$" as an exclusion,
and I run a test that creates a ".log" file.
The fswatch detects it!
But look closely at the file it detected - it's the enclosing directory.
Which makes sense - creating a file that you don't care about does modify
the directory.

On the other hand, if you exclude an entire directory, modifications to that
directory do not trigger fswatch.
For example, in some projects I have a set of platform-specific directories:
"mac", "lin", and "win".
There is no need to distribute the contents of "mac" to the remote host,
so I include this in "labbox.exclude" (used by rsync):
````
mac/
````
and this in "labbox.fsexclude" (used by fswatch):
````
.*/mac/.*
.*/mac$
````
Now I can create files inside of "mac" and it will not trigger fswatch.

To turn off exclusions, create empty versions of those files.


# DESIGN NOTES

Random thoughts about why I did what I did.

## Bash vs Sh

Note that it starts with "#!/bin/bash".
This is because I wanted to use constructs like:
````
echo -n -e "\033]0;Init\007"
````
which updates the window title and lets me know when an update is complete.

## Eval

The option processing for rsync was a bit tricky.
I ended up using "eval" to run the rsync command.
See http://blog.geeky-boy.com/2020/11/sometimes-you-need-eval.html
