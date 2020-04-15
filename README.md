# labbox
labbox: shell script for Mac to update a remote host's files.

Variant of "poor-man's dropbox", which originated as a script by Sahir.
I evolved it to be better for MacOS. But it is not a full 2-way-sync. It
runs on your laptop and detects changed files within a parent directory
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

Requires "fswatch", which is available via https://brew.sh/
```
brew install fswatch
```
