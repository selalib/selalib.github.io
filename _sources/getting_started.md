# Getting Started

To download the developer version you need to use GIT.
The latest development version of SeLaLib is tracked in the 'main' branch.

Git is tracking and controlling changes in the software. Git branches
allow frictionless context switching. Everything is local so we have
multiple backups on all developers' computers. It's impossible to get
anything out of Git other than the exact bits you put in. There is a
staging area, an intermediate index between working directory and
repository. Some tools you can install on your desktop:

- Install bash-completion and source git-prompt.sh [^1].
- use GitHub gui tool[^2][^3].

[^1]: <https://github.com/bobthecow/git-flow-completion/wiki/Install-Bash-git-completion>.
[^2]: <https://mac.github.com>.
[^3]: <https://windows.github.com>.

## First time set up

Set up your developer info:

```bash
git config --global user.name "Your NAME"
git config --global user.email "you@somewhere"
git config --global core.editor emacs (or other)
git config --global merge.tool opendiff (or other)
```

Check configuration with:
```bash
git config --list
```

For help:
```bash
git help <command-you-want-help-about>
```

Clone the repository
```bash
git clone git@github.com:selalib/selalib.git
cd selalib/
git checkout -b your-branch
```

To get all new updates of development (the main branch)
```bash
git checkout main        (change to main)
git fetch origin         (download changes from repository)
git merge origin/main    (update local branch main)
git checkout your-branch (back to your branch)
git merge main           (update your branch with modifications on develop)
```

If you have conflict, no problem just do :

```bash
git mergetool
```

It will open a nice editor and help her to choose the right version.
Close the editor and commit.

```bash
git commit -m 'Update from main and fixed conflicts'
```
Merge modifications from develop as much as possible


Full documentation for Git is available `here <http://git-scm.com/>`_.
