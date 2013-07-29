Git Branch Out
==============

## What does it do?

This script is intended to be placed on a server and run in a post push hook (like bitbucket's [POST Service](https://confluence.atlassian.com/display/BITBUCKET/POST+Service+Management)) to pull all changes from a remote repository and fan out the repository's branches into separate folders and allow them to be viewed separately.

This is something that could be useful for previewing different versions or revisions of a web project.

Ideally it would be set up with wildcard DNS and something like Apache VirtualDocumentRoot to allow any branch to be previewed at its own URL.

	<VirtualHost *:80>
		ServerAlias *.development.mydomain.com
	    VirtualDocumentRoot /path/to/repository/%1/
	</VirtualHost>

At the core of this script is the git-new-workdir script, from the contrib folder in the git repo.
https://github.com/git/git/blob/master/contrib/workdir/git-new-workdir

## Usage

Move the script somewhere in your PATH and make it executable with `chmod +x git-branch-out`

There are two different modes that this script can run in.

### Inital Setup Mode

	git-branch-out <remote repo url>

This will clone the repository and fan out the branches into separate folders in the current folder.

You can also pass a folder as the second parameter:

	git-branch-out <remote repo url> <folder>

This will clone the repo and fan the branches in the folder provided. If the folder doesn't exist, it will be created

If your project has a build script that needs to run you can pass that as the first paramter with a `-b` flag
	
	git-branch-out -b "buildscript" <remote repo url> <folder>


### Update Mode

Once the branches are fanned out, you could either run the script on a cron schedule or as a post push hook from your repository remote.

The usage is much the same as the inital setup except you don't need to pass the remote URL.

	git-branch-out <folder>

Where `<folder>` is the folder containing your fanned out branches.

If you are already in the folder, just run `git-branch-out`

In the same way as the inital setup you can pass a build script with the `-b` flag

	git-branch-out -b "buildscript"

#### The Queue

It is important that a second instance of the update command does not run if one is already running. So when the update script runs it creates a queue file and any further calls to the script will wait patiently in the queue file until it is their turn to run.

## Notes

It is very important that any files or folders that will be created or changed by the server are included in your .gitignore, this script will overwrite any local changes with whatever is coming from the remote.

This script makes several calls to the git remote, so it is probably best to use the SSH protocol so you aren't having to enter your password constantly.


