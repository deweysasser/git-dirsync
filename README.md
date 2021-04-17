# git-dirsync -- synchronize directories across multiple computers

This project is designed for an individuals working files (e.g. ~/Documents) to synchronize across
multiple computers on a reasonable time scale.

It is *not* designed for team drives (i.e. simultaneous editors) but may work for that. YMMV.

It should be run on multiple computers via a cron job or other scheduling mechanism.

It will generally:

* pull down any change pushed to the remote system
* maintain a "work in progress" commit on the main branch which is amended and force pushed
* deal with a remote push on top of the "work in progress" commit without losing any state.
* turn a "work in progress" commit in to a checkpoint on demand

If there are conflicts at any point it will wait politely for you to resolve them without losing
state on either side.

Generally, with one person switching back and forth between multiple computers, the files should
merge without conflicts. Problems might occur if an editor has a file open (and particularly if it's
locked).

## Implementation details

### Checkpoints

On every checkpoint, first we pull any changes from the remote repo and (if they do not conflict),
merge them in.

Then we create a checkpoint commit on branch `checkpoint/$mainBranch/$(hostname)` with all updates
and push it, then we find newly changed files and either create or amend a checkpoint commit and
force push it.

If the checkpoint commit is not the current HEAD, we create a new checkpoint commit and push that.

### Flush

On flush, we take the latest checkpoint commit, amend it to a daily commit, and force push it,
overwriting the checkpoint commit.

We also delete the checkpoint branch.

### Conflicts

Oh boy. For now, just refuse to pull in changes. Keep checkpointing to the checkpoint branches but
don't muck with the main branch.  

## TODO:

* Specify a regular expression for files in a repo setting `dirsync.file-regexp`