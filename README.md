# reverse-gitignore
what does that even mean? pretty much what you might imagine. traditionally a
[gitignore](https://git-scm.com/docs/gitignore) file is used to, well ignore
files. the dmenugitignore script assists with doing the opposite of that.
## made to be used with a bare git repo
if you manage your dotfiles via a bare git repo, the best way to manage
dotfiles, then you've potentially got a ton of files that may end up on github
if you aren't careful. sensitive info, embarassing info, you name it. if it
lives in your $HOME then you can track it on github along with your dotfiles,
for anyone to see... the same goes for any other bare git repo, but i think
management of dotfiles is really where reversing the behavior the `.gitignore`
file makes the most sense.
## reversal
reversing behaviour of the `.gitignore` file is as simple as creating a
`.gitignore` file that contians nothing more than this in your $HOME directory
(or wherever the root of your git repo is)
```bash
*
```
that's it. now __EVERY__ file in your repo is ignored recursively. you cannot
add any files to be tracked unless you include the `-f` flag when using
```bash
git --git-dir=$HOME/.git --working-tree=$HOME add -f file
```
or, you can explicitly specify which files to add like so
```bash
# ignores everything
*
# enables the file or folder to be tracked, does not work recursively on folders
!.config
!.local
!.gitignore
!.xinitrc
```
"boy that sounds like a shitload of work, i'll just run this command instead"
```bash
git --git-dir=$HOME/.git --working-tree=$HOME config --local status.showUntrackedFiles no
```
well sure you could, and that's a fine option, possibly a better one that doing
what i am suggesting, but i still think it's risky. thankfully there is a
solution to *most* of the tedium of what is required
## dmenugitignore
as i acknowledged above this way of managing your dotfiles can be very time
consuming. **every single file you care about has to be entered into a**
**`.gitignore` file prefixed with an exclamation point.** to make that
proccess easier i've made this script, dmenugitignore. here is how it works.
### running for the first time
if you do not have a .gitignore file in the working directory where you are
running this script then one will be created for you. you will then be
prompted, via dmenu, to either track, or ignore each file and subdirectory
found in the working directory where the script was run. this does not work
recursively. only files that would show up with `ls -Al` will be options to
either track or ignore. once you've chosen what to do with each file you'll end
up with a .gitignore file that looks like this
```bash
### Updated on Thu Dec  9 09:42:20 PM PST 2021 ###
*
!.gitignore
#autoload
!init.vim
#plugged
```
**the hashes in front of files and directories does nothing, this is simply to
show, at a glance, which files are being ignored.** rinse and repeat for every
other directory where you want to track some files, again folders are not
tracked recursively, *but you need to track them to be able to track what is
inside.*
### running in a directory that already contains a .gitignore file
only .gitignore files that are being used in a reverse manner will work. **if
your .gitignore file is a traditional one please rename it and see above for
operating procedure.** if the .gitignore file looks like the freshly created one
listed above then this script will only ask you what to do with new files. any
previously tracked or ignored files will carry over to new `.gitignore` file.
if a file was previously listed but has since been deleted it will be noted in
the new `.gitignore` file. you will only be prompted to track or ignore new
files, or files not listed in the old `.gitignore` file. after you've responded
to each prompt you'll end up with two or three files, `.gitignore`
`.gitignore-old` and `.gitignore-very-old`. the third will only be present if
you've run this script three times in the same directory. that's basically it,
a simple while loop and some if and case statements and this process becomes a
lot easier.
### tested and recommended ways to run this script
**this script is not meant to be run in your home directory if you are using
this for your dotfiles.** the `.gitignore` file in your home directory should
be pretty small. something like this
```bash
*
!.config
!.local
!.gitignore
!.gitmodules
!.xprofile
!.zprofile
!README.md
```
**an asterisk (*) must be in the `.gitignore` file at the root of your repo in
order to reverse the behaviour of subsequent `.gitignore` files.** but if you
want to run it in your home directory, go for it.
#### running straight from dmenu
a very easy way to use this script is through dmenu. all subdirectory of your
home directory will be listed as an option for you to choose from. select the
directory where you want to track some files. the script will then work as if
you were to cd into that directory and run the script there. a `.gitignore`
file will be created in the chosen directory and you will only be prompted to
track the files in that directory. if this script has been run in that
directory previously then it will only ask what to do with new files.
#### use with [lf: terminal file manager](https://github.com/gokcehan/lf)
this is my perfered method. simply add this to your lfrc
```bash
map .gi %dmenugitignore
```
you can map it to whatever you like. this will run the the script in the
directory you are currently in. not much else to say. i like this way because i
mainly use lf for file browsing but specificly in this case i get a nice view
of what files are there and can figure out what i want to do with each of them
before i run the script.
#### cli
```bash
cd /dir/with/files/you/want/to/track
dmenugitignore
```
not much else to say. use `ls -Al` to get a preivew of what files you'll have
the option of tracking.
## dmenugitignore-auto
dmenugitignore-auto makes maintaining your reversed `.gitignore` files that
much easier by finding all the `.gitignore` files in the given repo and running
dmenugitignore in those directories to update all of those files at once.
instead of going and manually searching for and updating each `.gitignore` file
this script finds those files and updates them for you.
### not just for one repo
if you use a reverse `.gitignore` system for multiple repos then you will
appreciate the fact that this script will find those files for a given repo.
#### how it finds the files
if you run
```bash
/usr/bin/git --git-dir=$HOME/.git --work-tree=$HOME ls-tree -r master --name-only
```
every single file tracked by that repo will be printed onto your terminal. and
if you are tracking your reversed `.gitignore` files **which you should be
doing** then those will be printed here as well. if we pipe that output to grep
and search for ".gitignore" then we will have all the directories that are
being tracked. simply chop off the "/.gitignore" and you've got $HOME/foo/bar.
the script then cd's into that directory and runs dmenugitignore there for you.
by taking all those directories and feeding them through a while loop each and
every `.gitignore` file that is being track will be automatically updated. if
there are new files dmenugitignore will work as it normally does and ask you
what you want to do with them. and that's really all there is to it. this takes
much of the tedium out of updating these files and is especially helpful for
maintaining something like your dotfiles. but it is not just for you dotfiles
#### flags
##### -d
the -d flag wants the path to, effectively, a .git folder. above you can see
`--git-dir=$HOME/.git` -d wants whatever you would put after the equal sign.
**when using the -d flag it is neccessary to use the -t flag.**
##### -t
the -t flag is like -d but it want's the path to the working tree
`--work-tree=$HOME` it is up to you to know the working tree that goes with the
git dir. by default the git dir and working tree are set to $HOME/.cfg and $HOME
respectively. that is because i use this primarily to maintain my dotfiles which
i set up following [this](https://www.atlassian.com/git/tutorials/dotfiles) guide.
if you have a different default configuration then please change lines 51 and 52
to the proper paths for the repo you intend to use this with most frequently.
##### -b
-b is the branch for your repo that you want to update. if you have seprate
branches for your dotfiles, say for multiple machines, or whatever, then make
sure you are updating the proper branch. change line 53 if your most used
branch is something other than maste *cringe*
## installation
**the script has only been tested on linux but the principle can be applied to**
**any git repo...just without the convenience this script offers**
```bash
curl -O https://raw.githubusercontent.com/johndovern/reverse-gitignore/master/dmenugitignore
curl -O https://raw.githubusercontent.com/johndovern/reverse-gitignore/master/dmenugitignore-auto
chmod +x dmenugitignore
chmod +x dmenugitignore-auto
echo $PATH
```
place somewhere in your $PATH and feel free to run it as described above.
**dmenu must be installed for use with this script** please either install it
from you packagemanger or build from source
```bash
curl -O https://dl.suckless.org/tools/dmenu-5.0.tar.gz
tar -xvzf dmenu-5.0.tar.gz
cd dmenu-5.0
make && sudo make clean install
```
you may need some devel packages, read the errors to find what you're missing.
## disclaimer
**only tracked directories can track files.** if you want to track a file each
parent directory of that file must also be explicitly tracked in the
`.gitignore` file of it's parent directory. if any of this is confusing, or,
more likely, if i have failed to explain it properly, then please see the
directories and files in this repo to get an idea of how this all works.
reversing the way the .gitignore file works may seem strange, pointless, or as
a way to simply make one waste their time. however, i think that this really is
a great way to add a layer of security to you dotfiles repo. accidently adding
an ssh key or private picture does not sound fun to me. if you still have a
hard time understanding how this all looks in action, checkout my dotfiles.
each .gitignore file is tracked (this makes installation and maintence on new
machines a breeze) and should give you an idea of how this looks in action.
there isn't a single file there that i did not explicity add through a
.gitignore file. i run git add . whenever i've got a new file to track, no more
long cli commands or painfully typing in the exact path to the file i want as
git does not allow for autocompletion of file paths for some unknown reason.
basically it works for me, and if it works for you and helps you feel a bit
more safe, then that's great.
