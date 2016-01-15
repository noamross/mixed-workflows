Peace between git and Dropbox with git-worktree
===============================================

This is the first of what may be a series of short bits about ways of harmonizing workflows between people who work in low-level, programmatic ways with goals of reproducibility and automation, and those work with high-level graphical tools with less support for those things. I won't go too far into the *why* here, suffice to say that it's problem many people face, and

-   I think specialization is good; some people should learn to code, others have other things to focus on
-   High-level, popular GUI tools work for a lot of people for a lot of things
-   People and organizations' practices change slowly. Baby steps, etc...

With that, our first installment is about git/Github and Dropbox. Both are exceptionally useful tools, but for the most part they don't work that well together when directories are being shared with multiple people via Dropbox. Conflicts in the `.git` folder can wreak havoc, dropbox gets bogged down with many small files in the repo, and in general they just are designed under fundamentally different approaches to collaboration. However, in a team it's likely that Dropbox is the collaboration tool of choice for non-programmers.

I've found a useful solution recently with [`git-worktree`](https://git-scm.com/docs/git-worktree). This is a git command that allows one to use different directories to represent different branches of a project. It's good for many things, such as using different directory for the `gh-pages` branch of a project. With `git worktree`, you have one or more "linked working trees" (linked directories) connected to the "main working tree" (main directory). Importantly, *linked directories do not have `.git` folders*. Instead, they have `.git` *files* which point them to the main directory. This means they are much less trouble to sync with Dropbox.

To setup a project this way, I start with my main project directory outside of Dropbox. Then I run the following

    git worktree add -b dropbox ~/Dropbox/project-repo-dropbox

This creates a new branch called `dropbox` and gives me a directory structure like this:

    home
    ├── project-repo
    ├── Dropbox
        ├── project-repo-dropbox

If you `cd` into `project-repo-dropbox`, you'll find you're in a git directory on the `dropbox` branch, but if you `cat .git`, you'll get

    gitdir: /home/Dropbox/project-repo/.git/worktrees/project-repo-dropbox

The contents of this `.git` file are how git knows where the actual repository lies.

Now you can share `project-repo-dropbox` with team members via Dropbox and it won't clobber your git repository. Dropbox-using team members can edit files, and you can commit those changes as needed. I typically push changes from `dropbox` to `origin master` prior to any time I might pull from `origin master` to `master`. I pull from `origin master` to `dropbox` right after I push from `master` to `origin master`. The `origin` remote doesn't have a `dropbox` branch at all, though I'm sure there might be reason to in some workflows. It gets a bit tricky if more than one person needs to use git and Dropbox. I haven't had this problem yet, but it's likely to come up if multiple git users need to pull in changes coming from Dropbox users. First, `gitdir:` stores an absolute path, so all git users will have to tell Dropbox not to sync the `.git` file [using this trick](http://superuser.com/a/757498). Second, users after the first will likely have to create a separate linked directory, copy the `.git` file into the shared dropbox directory, then manually edit both the `.git` file and it's equivalent in the main directory, which is

    home/project-repo/.git/worktrees/project-repo-dropbox/gitdir

If anyone tries this final step, or any of this let us know how it works!

Anyone have suggestions for improvements or alternatives?
