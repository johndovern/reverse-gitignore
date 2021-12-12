# reverse-gitignore
what does that even mean? pretty much what you might imagine. traditionally a
gitignore file is used to, well ignore files. this script assists with doing
the opposite of that.
## made to be used with a bare git repo
if you manage your dotfiles via a bare git repo, the best way to manage
dotfiles, then you've potentially got a ton of files that may end up on github
if you aren't careful. sensitive info, embarassing info, you name it. if it
lives in your $HOME then you can track it on github along with your dotfiles,
for anyone to see... the same goes for any other bare git repo, but i think
management of dotfiles is really where reversing the behavior the `.gitignore`
file makes the most sense.
## reversal
reversing the gitignore file is as simple as creating a `.gitignore` file that
contians nothing more than this in your $HOME directory (or wherever the root
of your git repo is)
```bash
*
```
that's it. now __EVERY__ file in your repo is ignored recursively. you cannot
add any files to be tracked unless you include the `-f` flag when using
```bash
git --git-dir=$HOME/.git --working-tree=$HOME add -f file
```
or, you can explicitly specify which files to add like so
```
# ignores everything
*
# adds the file or folder to be tracked, does not work recursively on folders
!.config
!.local
!.gitignore
!.xinitrc
```
boy that sounds like a shitload of work, i'll just run this command instead
```bash
git --git-dir=$HOME/.git --working-tree=$HOME config --local status.showUntrackedFiles no
```
well sure you could, and that's a fine option, possibly a better one that doing
what i am suggesting, but i still think it's risky. thankfully there is a
solution to *most* of the tedium of what is required
## dmenugitignore
as i acknowledged above this way of managing your dotfiles can be very time
consuming. every single file you care about has to be entered into a
`.gitignore` file prefixed with an exclamation point. well to make that
proccess easier i've made this script, dmenugitignore. here is how it works.
### running for the first time
if you do not have a .gitignore file in the directory where you are running
this script then one will be created for you. you will then be prompted to
either track, or ignore each file and directory found in the directory where
the script was run. this does not work recursively. only files that would show
up with `ls -Al` will be options to either track or ignore. once you've chosen
what to do with each file you'll end up with a .gitignore file that looks like
this
```bash
### Updated on Thu Dec  9 09:42:20 PM PST 2021 ###
*
!.gitignore
#autoload
!init.vim
#plugged
```
rinse and repeat for every other directory where you want to track some files,
again folders are not tracked recursively, you need to track them to be able to
track what is inside.
### running in a directory that already contains a .gitignore file
only .gitignore files that are being used in a reverse manner will work. **if
your .gitignore file is a traditional one please rename it and see above for
operating procedure** if the .gitignore file looks like the freshly created one
listed above then this script will only ask you what to do with new files. any
previously tracked or ignored files will carry over to new `.gitignore` file.
if a file was previously listed but has since been deleted it will be noted in
the new `.gitignore` file. you will only be prompted to track or ingore new
files, or files not listed in the old `.gitignore` file. after you've responded
to each prompt you'll end up with two or three files, `.gitignore`
`.gitignore-old` and `.gitignore-very-old`. the third will only be present if
you've run this script three times in the same directory. that's basically it,
a simple while loop and some if and case statements and this process becomes a
lot easier.
### tested an recommended ways to run this script
**this script is not meant to be run in your home directory.** the `.gitignore`
file in your home directory should be pretty small. something like this
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
but if you want to run it in your home directory, go for it.
#### running straight from dmenu
a very easy way to use this script through dmenu. every subdirectory of your
home directory will be listed as an option for you to choose from. select the
directory where you want to track some files. the script will then work as if
you were to cd into that directory and run the script there. a .gitignore file
will be created in the chosen directory and you will only be prompted to track
the files in that directory. if this script has been run in that directory
previously then it will only ask what to do with new files.
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
## installation
**the script has only been tested on linux but the principle can be applied to**
**any git repo...just without the convenience this script offers**
```bash
curl -O https://raw.githubusercontent.com/johndovern/reverse-gitignore/master/dmenugitignore
chmod +x dmenugitignore
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
**i cannot stress enough the fact that a tracked folder does not allow for file**
**tracking recursively. adding a directory to a `.gitignore` file will only allow**
**you to track files in that directory and enable tracking in subdirectories. you**
**will need to run this script in each directory which contains files, or**
**subdirectories with files that you wish to track.** you can give an absolute
path to directories and files, hell you could have only one `.gitignore` file
in you home that has absolute paths to each file you want to track. but at that
point the file becomes almost unreadable. if any of this is confusing, or more
likely i have failed to explain it properly, then please see the directories
and files in this repo to get an idea of how this all works. reversing the way
the .gitignore file works may seem strange, pointless, or as a way to simply
make one waste their time. however, i think that this is really is a great way
to add a layer of security to you dotfiles repo. accidently adding an ssh key
or private picture does not sound fun to me. if you still have a hard time
understanding how this all looks in action, checkout my dotfiles. each
.gitignore file is tracked (this makes installing on new machines a breeze) and
should give you an idea of how this looks in action. there isn't a single file
there that i did not explicity add through a .gitignore file. i run git add .
whenever i've got a new file to track, no more long cli commands or painfully
typing in the exact path to the file i want as git does not allow for
autocompletion of file paths for some unknown reason. basically it works for
me, and if it works for you and helps you feel a bit more safe, then that's
great.
