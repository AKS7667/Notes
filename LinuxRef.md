


# Header 1
## Header 2
### Header 3
**Bold**
*Italics*
Normal text
```powershell
Code Block
```
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Row1 A   | Row1 B   | Row1 C   |
| Row2 A   | Row2 B   | Row2 C   |

<br> // for Line breaks

==highlight==

> Blockquote
>> dfsd
>>> dffdfd

~~strikethrough~~ 

To highlight ==these words==  

subscript - H~2~0  

superscript - 4^2^ 


**To view me press ```ctrl + k then v```**

<br><br>


# Linux Basics


## Helper - Shortcuts, Tricks & Tips, General Knowledge

### Redirection

0(stdin) - The standard input, keyboard. 
1(stdout) - The results of the command ( the good output ).
2(stderr) - The errors the command returns

|Operator|	Meaning |	What it does |
|----------|---------|----------|
| > |	Overwrite |	Sends output to a file, deleting the file's old contents. |
| >> |	Append |	Sends output to a file, adding it to the end of the existing text. |
| < |	Input |	Takes the contents of a file and "feeds" it into a command. |
| \| |	Pipe |	Connects the output of one command to the input of the next command. |
| 2> |	Error | Only	Sends only the error messages to a specific file. |
| &> |	Both |	A modern shortcut for ```2&1```. Redirects both stdout and stderr. |

<br><br>

### Meta characters
| Character | Meaning | Use case |
|----------|----------|----------|
| * | 0 or more chars | ``` find . -name "*.md" --> all .md files``` |
| ? | Exactly one character | ``` find . -name "file?.txt" --> 'fileA.txt and 'file1.txt```  |
| [abc] | Any one character in set | ``` find . -name "file[12].txt" --> 'file1.txt' , 'file7.txt' ``` |
| [a-z] | Any 1 character in range | ``` find . -name "file[a-c].txt" --> 'filea.txt' , 'fileb.txt' ``` |
| {1,2,3} | Brace Exapnsion | ``` touch file{1,2,3}.txt --> Creates three files 'file1.txt' , 'file2.txt', file3.txt' ``` |
| . | Literal (shell) and any single char (regex/grep command) | ``` grep "a.b" --> matches 'a_b', 'a1b' ```|
| $ | Line/ string ending with | ``` grep "txt$" --> lines ending with txt``` |
| ^ | Line/ string starting with | ` ls -l \| grep ^d ` Returns all lines starting with a d , i.e. all directories|

#### Examples 
``` <command> > logs.log 2>&1 ``` Redirects BOTH the output and the errors to the logs file. It is important that you put hte ```2&>1``` AFTER the first - mian output - direction is done. Think of it as saying , first we define where the output goes by doing ``` <command> > output.txt ``` and then we tell the errors to follow that pipe by saying ``` 2>&1```.
``` < command > > logs.log 2> errors.txt ``` Seperates the errors from the useful output.
``` <command> > /dev/null &>``` IF you don't want to see the output or the errors.
``` <command> >> append.txt ``` Appends to the file instead of overwriting it like ```>``` does.  

<br><br>
Linux OS filesystem when downloading libraries. 
|Location | what goes there |
|----------|----------|
| `/lib` | Core OS libraries| 
|`/usr/lib` | System package libraries (`apt-get`)|
|`/usr/local/lib`| Manually compiled/installed libraries (`make install`)|
|`usr/local/lib/SomeName/` | Non-standard — package chose its own subdirectory |

For python specific - 
| Install method | Goes to |
|----------|----------|
|`apt-get install python3-numpy` | `/usr/lib/python3/dist-packages/`
|`pip install numpy` (system) |`/usr/local/lib/python3/dist-packages/`|
|`/opt/venv/bin/pip install numpy` | `/opt/venv/lib/python3.10/site-packages/`|
|`conda install numpy` | `~/miniforge3/envs/myenv/lib/python3.x/site-packages/ `|

To look for libraries (cpp)- 
```bash
find / -name "libX*.a" 2>/dev/null    # static
find / -name "libX*.so" 2>/dev/null   # shared
```
<br><br>
### Actions
**Tags:** #Terminal Control
| Action | Shortcuts | Utility |
|----------|----------|----------|
| Interuppt process|```ctrl + Z``` | Pauses a running command in the terminal (like a long download) and gives you back the terminal
|Suspend Process | ```ctrl + C ```| Complete termination of the command/ program.|
| Exit Shell | ```ctrl + D```|  Logs out/ closes Shell |
| Clear Screen | ```ctrl + L``` | | 
| Force logout SSH | `~.` | | 
| Move cursor to beginning/end | ``` ctrl + A / ctrl + E ```| | 

**Tags:** #Command Line Editing
| Action | Shortcuts | Utility |
|----------|----------|----------|
| Move back one word | ```Alt + B``` | | 
| Move forward one word | ```Alt + F```| |
| Delete word before cursor | ``` ctrl + W ``` | |
| Delete all from cursor to end | ```ctrl + K``` | Chop tail |
| Delete all from cursor to start | ``` ctrl + U```| Chop head |
| Paste last deleted text | ```ctrl + Y``` | Works to unfo ```ctrl + W``` |

**Tags:** #History
| Action | Shortcuts | Utility |
|----------|----------|----------|
|🔥 Search History |```ctrl + R```|  Makes command history searchable ; Like ```ctrl + F``` for command history |
| Repeat Last command | ``` !!```<br>```  sudo !! ```<br>``` !-2 ```<br>``` !ssh``` | Re-run instantly<br> Repeat last command as sudo <br> run command two steps back (one before last) <br> Run the most recent command starting with 'ssh'| 
| Repeat last argument | ``` !$ ```| Uses the last argument as the current argument | 

<br>
<br>

### Snippets - Composite/Custom/Useful Commands

1. ``` cd - ``` takes you to the previoius directory you were in.
2. ```bash -x script.sh``` : Lets you see execution of a file line-by-line by printing it out, with updated values for the variables. 
3. ```sudo -u user printenv```: To trouble shoot the variables of a specific user.
4. ``` echo 'export PATH= "$PATH:MY/NEW/PATH/"'>> ~/.zshrc```where A new directory path is added to the $PATH of the User's enviroment, so that when running a new command , it's location is available in the path.
5. ` cd "$(find . -maxdepth 2 -name 'EVERYTHING' -type d -print -quit)"` to input a command as an argument to another. -print prints the output and goes to the next argument, so basically only the first one is printed. 
6. ` module avail | less` : To see all available modules. The less option is what makes the output scrollable and also SEARCHEABLE by using the `/` key. 
`
Where:
   L:        Module is loaded
   D:        Default Module
   Aliases exist: foo/1.2.3 (1.2) means that
             "module load foo/1.2" will load foo/1.2.3
`
7. 

<br>
<br>

### Specifics
1. **ACL** :  Doesn't work with mounted files, only files on the system. 
2.  **Stacking parameters** : Instead of typing ls -l -a -h you can just type ls -lah.
3.  **tab** : When using with cd command, shows all options that the command can end with. For example - 
   cd * press Tab * - shows all options for the cd command to end in (based on the current directory)
4. **DEBUG** : Setting ```DEBUG=True``` Lets you see what is going on under the hood .
5. ```' ' & " " & $```: To tell Linux that 'there is a variable you need to look for' we add a '\$' sign before it - the variable pointer;  Single quotes interpret the quoted value as  a literal (no interpretation of any variables); The double quotes CAN interpret variables. Example: 
 ``` echo "My name is $NAME"```, Note how we used double quotes, same thing with single quotes would have printed ```My name is $NAME```.
6. ```<command> &**``` : Runs the command in the background. 
<br><br>


### Definitons 
1. ```$PATH``` : all the files that are the default 'search directories' to look for the command file in when you run the command. 
2. ```.bashrc/.zshrc``` : The default files that run when you start a new session/ terminal, note that we use zsh shell, there is bash and the original sh. Makes sure all your aliases, custom PATHs are added and updated.
3. ```.profile/.zprofile``` : Runs once when you login into the system. THe difference between this and the bashrc file is that the bashrc runs for every terminal (so local information liek aliases, terminal color information, etc, should be saved here) all the global varaibles are inhertied from the parent - the .profile file - into the children - every new terminal - allowing them to read 'export' commands, PATH, and other variables. 
4. ```.<file>``` : The ```.``` hides the file. 
<br><br>

## Commands

### Command: <command_name>

**Tags:**

#### Purpose
Short explanation of what this command does in plain English.

#### Syntax
```powershell
command [options] [arguments]
```
| Column 1 | Column 2 | Column 3 |
|----------|----------|----------|
| Row1 A | Row1 B | Row1 C |
| Row2 A | Row2 B | Row2 C |

-------

### Command: Basic Commands


**Tags:** #File Operations

| Command | Meaning | Use cases | Comments|
|----------|----------|----------|----------|
| ls | list out elements of the current directory   | ```ls -lah``` <br> `ls -l oldfile.txt` <br> `ls -lahd *D ` | -l - long format <br> -a all files (including hidden files) <br> -h human readable (for file sizes) <br> *D is the pattern and -d lists only the directory, not the contents within them |
| uname | System name | uname | |
| cmp | Compares two files byte by byte and shows the first instance they are different| ```cmp file1.txt file2.txt ```|
| compress| Compresses the file to a zip file and returns a .Z| ``` compress file.txt ```|
| cpio | Creates a bakup of all the files - usually piped from an output , like ls into ONE single database of these files, .cpio | ``` ls \|cpio -o > backup.cpio``` | ```cpio -it < backup.cpio``` -i for input reading , -t to return the table of files ; ```cpio -id < backup cpio``` -d direcotries,  Returns as individual files on disk |
| file | identifies the type of file | ``` file a.txt ```|
| fold | wraps text to specified width for easier reading | ``` fold -w60 gfgf.txt ```|
| head | shows the first 10 lines of a file by default | ``` head a.txt ```|
| stat | Details about the file, permissions, last modified, owner, file type | ```stat -c %a myfile``` permissions<br> %U gives owner name<br> %Y gives last modified<br> %F gives file type<br> %n gives file name<br> ``` stat -c "perms: %a \| Owner: %U \| edited : %Y \| Type : %F"``` | |
| touch | quickly create files | ```touch file.txt abs.py``` <br>  `touch -c oldfile.txt` | | -c create file if it doesn't exist| 
| ln | lets you create links to file | ```ln targetfile.txt link.txt``` <br> ```ln targetfile1.txt trgtfile2.py /destination``` | instead of adding a link file name, can also add a destination directory | 
| cat| display contents of a file | ```cat abc.py``` |-i case insensitive; ```-l "match_find" *``` All files with match; -w whoole word match; -``` grep -e "Agarwal" -e "Ushin -e "Krish"``` | 
| ps | Current processes | ```ps``` | | 
| grep | look for a specific term in the input (could be piped from another command or a file); like ctrl+f but in linux | ```grep "python" notes.txt ``` | -r recursivwly looks for it (even in subdirectoreis) <br> -i makes the search term case insensitive; -n shows the number of the line as well <br> -w matches only the whole word (no sub words) <br> -c counts the number of matches as well|
| wget | To get a website through a url| ```wget https://www.geeksforgeeks.org/linux-unix/basic-linux-commands/``` | ``` -o ./filename ``` saves into specific file; ```-w 10 https://www.geeksforgeeks.org/linux-unix/basic-linux-commands/``` waits for a specified time before retreiving|  

<br>
<br>

**Tags:** #Directory Operations
| Command | Meaning | Use Cases | Comments |
|----------|----------|----------|----------|
| dirname  | Gives the directory path without the file name in it | ```dirname --> /mnt/c/Users/aksom - no file ``` |
| lsblk | Will present the mount points and storage information of different directories |   
| du | Shows disk usage of the file | ``` du "./Dir/folder" ```|
| dirs | Shows previously used directories in hte directory stack | ```dirs```|
| tree | Shows the subdirectories in a tree like structure to be able to view them easily |``` tree -L -I 'node_modules\|Ignore_me\|Ignore_this\|*git' ``` <br> `tree -L 2`| -d directories only; -P pattern ```tree -P '*.sh' ```; -D displas the date as well; -h --du humnan readable sizes; -pug shows permissions, users and groups like -ls -l; -- sort=size sorts by size; <br> -L tells tree how deep it should go into each directory| 

<br>
<br>

---------

### Permissions
**Tags:** #File Permissions

When you use the *stat* command or *ls -l* command, you get something like this as the output - 
```powershell
-rwxrwxrwx 1 aksom aksom       20 Dec 20  2022 Untitled-1.py
```

- The "-" represents the holder for file type, where *d* means directory, *-* means file and *l* means link.
- The "rwxrwxrwx" is three parts for permissions of three groups. The first three characters represent the permission for the User, the next three represent the permissions for the group and the last three represent the permissions for Others. Where *r* represents Read, *w* represents Write and *x* represents Execute permissions respectively; *-* represents that no permission is set
- Then you have the number of links pointing to this file (1 is default), the Owner of the file (aksom) and the Group the owner is a part of (aksom) , file size, modification time, and the file name. Note only the primary group of the Owner has the permissions above, to find the grp use - ``` id -gn <username>``` Leave username empty if trying to find active user's Primary group. 
- Note Read(r) = 4, write(w) = 2, Execute (x) = 1 and No permission (-) = 0
  
Sample table on how to read numbered permission files - 

| Number| Permission |
|----------|----------|
| 0 | --- |
| 1 | --x |
| 2 | -w- |
| 3 | -wx |
| 4 | r-- |
| 5 | r-x |
| 6 | rw- |
| 7 | rwx |


<br>

### Command: chmod

**Tags:** #Permissions

#### Purpose
Used to edit permissions to a file;  

#### Syntax
```powershell
chmod [permissions] <files>
```

#### Examples

```powershell
chmod 764 agatha.txt
```
>

> 7 (r+w+x) for User, 6 (r + w) for group and 4 (r) for others.

<br>

``` 
chmod u-r g+x o+w agatha.txt
 ``` 
 >

 > - symbolic way to edit permissions for other (o), user (u), group (g) or all(a) [u+g+o] , + for adding , - removing, = to overwrite current with new  

<br>

### Command: <chown >

**Tags:**

#### Purpose
To change the owner and the group of a file. 

<br>

#### Examples

```powershell
Ex1: 
sudo chown bob: his.txt

Ex2:
chown --reference permFile.txt newfile.txt 

Ex3:
chown $USER:$USER chng.txt
```  
>  - If you just to want to change the owner, don't use a colon Not specifying fgroup after the : defaults to the User's primary grp `$USER" Automatically fills in the name of the current user can use -R to recursively apply to subdirectories, but RISKY, for system files 

<br>  

### Command: chgrp

**Tags:** #Permissions

#### Purpose
Change group ==ownership== of a file. 

#### Examples
  
```powershell
sudo chgrp Newgroup file.txt
```  
<br> 

### Command: setfacl

**Tags:** #Permissions

#### Purpose
Change permissions of a group (g), user(u), other (o), to a file. Also type mask (m) to restrict giving certain permissions ; stands for **Access Control List**

#### Examples
  
```powershell
Ex1. 
setfacl -m u:$USER:rwx test.py`

Ex2.
setfacl -x g:interns:--x imp.py
```

> -   -x removes access as defined; -R makes the permissions recursive, i.e. for all directories and subdirectories -d makes permissions valid for all new files added to the directory (doesn't apply to files currently in the directory);
> -  ` setfacl -m m::r-- test.txt` allows only read permssions to be set for the file, even if rwx is given to a user, it will automatically revert to r--, NOTE: doesn't apply to owner of the file;-n no mask calculation, use -n when assigning to a user and mask needs to enforced (otehrwise mask 'automatically fixes' to user needs); 
> - -b removes all acl's defaults to chmod settings; to give access to all current and future files in a dir use `setfacl -R -m d:u:username:rwX impodir`, note that the X gives executables to files only in the current directory, and NOT executable to files in sub-dirs, the d: is what sets the 'defualt' for the dir's acl ; 
> - -k to clear all defaults.  

  
<br> 

### Command: getfacl

**Tags:** #Permissions

#### Purpose
Gets the acl to the file/dir, shows details such as user permissions, mask, owner, default permissions, etc.**Access Control List**

#### Examples
  
```powershell
Ex1. 
getfacl impodir
```
<br>
<br>

---------

### Process Management
| Command | Meaning | Use Cases | Comments |
|----------|----------|----------|----------|
| ps | Shows all running processes |```ps aux``` | a - all users; u - usage (RAM/CPU); x - includes processes not attached to the terminal |
| top | The 'Task manager' of your linux system, shows all running tasks, mem usage etc| `top -p <PID>` <br> `top -M <k or m or g>`  <br> `top -b -n 1`| -p allows you to inspect a specific PID<<br>> -M allows you to set units for storage info<br> -b runs in batch mode (plain text rather than interactive), -n 1 tells it to run only once and then quit, useful when output to file. It also has interactive keys, pressing *k* (kills a command) --> asks PID of command , pressing *r* sets Niceness (-ve Niceness = high priority; +ve = lower priority) --> asks PID --> asks niceness level; q quits |
| htop | Interactive version of top, with mouse interactions and color coded info | ``` htop ```| |  
| kill | Kills the program, specified by PID | ``` kill %<PID>``` <br> ```kill -9 %<PID>```| -9 is the nuclear, forced way to kill, the first one is a cleaner way - save and close | 
| killall | Uses name intead of PID | ``` killall firefox ``` | Kills all programs related to firefox, no PID needed | 
| pkill | Similar to killall but more flexible | | | 
| pgrep | Get the PID from the command name | ``` pgrep -la snap ``` | -a gives you the whole command name | 
| jobs | lists all jobs active and suspended | ``` jobs ``` <br> `jobs -p`  | Returns jobs, each with a number - such as [1] ; -p gets the PID as well useful for `kill $(jobs -p)` kill all jobs|
| bg | Runs a suspended/ paused job (typically with ``` ctrl + Z``` ) in the background, handing you the terminal back | ``` bg %<Job Number>``` | Job number can be found from the output of the ```jobs``` command |
| fg | brings a job to the foreground | ``` fg %1 ``` | job number 1 runs in the terminal |

<br><br>

Top Table reference - 
| Name | Meaning |
|---------|---------|
| PID | shows task's unique process id|
| USER | User owner of task |
| PR | The processes priority; lower number = high priority |
| NI | Represents the niceness value; -ve number = high priority; +ve number = low priority | 
| RES | How much physical RAM the process is using in kb |
| SHR | Shared memmory used by task | 
| %CPU | CPU usage | 
| %MEM | Memory usage of task | 
| First line | <time> up <time since last boot> <No of Users> user, load average : <cpu wait time last 1 min>, <" " last 5 min>, <" " last 15 min> |
| MiB Mem | RAM usage |
| MiB Swap | Overflow memory/ emergency RAM usage | 

<br>
<br>

----

### Environemnt & Shell

- **Environemnt** : The environemnt of a shell is a set of varaibles that the shell uses. Such as ``` $PATH , $HOME, $USER, $LANG``` etc, the most important being ```$PATH``` , which is a list of all folders to look for when trying to run a command (without it you would have to type out the complete path of the command  you are trying to use - such as for ```ls``` and ```python```). Similarly, ```HOME``` points to your home directory, ```SHELL``` tells you which shell you are using (bash, zsh).  
<br><br>

| Command | Meaning | Use Cases | Comments |
|----------|----------|----------|---------|
| env or printenv | Prints all current<br> environment variables<br>env alsos used to run a program <br>in a new 'clean' env with variables privy to that env <br>which are gone after <br>the program is finished running | ```env PATH=/custom/path MY_VAR = hello python script.py ```<br>``` printenv PATH ```<br> ```printenv <VARIABLE> ``` | printenv path shows <br> you all the<br> directories currently in<br> the environments $PATH<br>, basically everywhere it <br>looks for a command folder <br> <variable> prints <br>the value of the variable| 
| export | When using ```VAR=2 script.sh``` the variable only exists <br>for that one <br>command and only <br>that one file's <br> execution, no other <br>file running after <br>that can see <br>the variable or<br> access it. Export makes <br>it available to<br> all files in <br> the CURRENT SESSION | ```export MY_VAR 2``` <br> ``` echo 'export PATH= "$PATH:MY/NEW/PATH/"'>> ~/.zshrc``` | Note that only <br> the current terminal can see<br> this variable another <br>terminal won't be able to see this, and it <br>gets erased from there <br>too as soon as <br>the session terminates. | 
| unset | Undoes the export command| ```export MY_VAR="hello" ```<br> ```unset MY_VAR``` | Unsets the global variable |
| source | Rereads the <br>.bashrc (or the .zshrc) file <br>or the .profile file,<br> without having to restart<br> the terminal to update the terminal<br> with the changes made <br>in the bsahrc file | ```source ~/.bashrc``` <br> ``` source ~/.profile``` | |
| set | Useful to set variables, but when passed with no args, returns all variables that have been set already; *All variables the user is allowed to set*. | `Add examples` | |

<br>

### Command : tee

#### Purpose 
To print AND to save the output to one (or multiple files)

### Syntax
```powershell
<command> | tee <file1> <file2> 
```
<br>

```powershell
<command> | tee -a <files>
```
> The -a argument appends instead of overwriting.

<br>

### Command : tmux
Terminal Multiplexer

#### Purpose
Lets you run multiple windows inside one terminal, which persist even when you disconnect, useful for remote work.

#### Syntax
```powershell
tmux # start new session
tmux new -s myproject # starts new session with a name
tmux ls # lists all current projects
tmux attach -t myproject # reattaches to the terminal
```
Within  ```tmux``` you have the following actions available to manage and navigate the terminal/s - 
The default prefix action is ``` ctrl + b```, followed - separately - by one of the following - 

| option | Meaning |
|----------|----------|
| c |  Create new window| 
| n | Next window | 
| p | previous window |
| % | Split pane vertically | 
| " | Split pane horizontally | 
| <arrrow keys> | Navigate between panes | 
| d | Detach - session keeps running! |
| x | Kill session |  
<br>

### Command : xargs

#### Purpose 
Takes - typically - a list as an argument and uses it as an input to another command, bridging the gap between output lists and commands that take arguments. 

#### Syntax 

```powershell
Ex1. 
find . -name "*.txt" | xargs rm
```


> remove all .txt files from the home directory

<br> 

```powershell
Ex2.
find .name "*.txt" | xargs -p rm 
```
> ask before inputting into next command - here , ask before removing.

<br>

```powershell
Ex3.
ls *.jpg | xargs -n1 -P8 convert convert resize 50%
```  
>- -n1 ensures it runs one file at a time (instead of using all the files as an input in  one command, use each file as an individual input, and run command for each one of them sequentially). 
> - -P8 means to run 8 tsuch commands simultaneously together (for it to go faster)

<br>

### Command: find

#### Purpose
To locate specific files by path, name, type.

#### Syntax
```powershell 
find [starting_point/s...] [expression]

find /mnt/Users/aksom -maxdepth 2 -mtime -24 -iname "linux_reference.md" 
```
| option | Meaning | Use case | Notes |
|----------|----------|----------|----------|
| path *pattern*   | lets you add the paths to directories where to search | `find . -path "./src*sc"` | Doesn't differentiate between characters and / , so this could reuturn ./src/misc| 
| type  | file is of type *type*   | `find -name 'abc.txt' -type f ` | |
| printf  | print full file name with options to modify   | `find -name 'abc.txt' -printf` | |
| name | Specify the name of the file, use *iname* to make the search char insensitive|` find -iname "Linux_reference.MD"` |  |
| maxdepth | Specify how many directories deep should the search go | `find . -maxdepth 2 -name "Linux.txt"`  | searches two directories deep; Similar param is *depth*, where it solves the contents of the directory first before solving for the directory itself.|
| mtime | Time it was last edited |` find . -mtime +2 -name "abc.txt"` | Similar command is atime, which looks for last accessed time|
| quit | quits the programme | `find . -name "*.txt"  -print -quit` | Find the first occurrence of 'PATTERN' , prints it and quits it (basically finds the first match) | 
| exec | Executes a command | `find . -type f -exec <command> '{}' \;` | the braces and the ; are  syntax|
| perm | Search by perm | ` find . -perm 664` <br> `find . -perm -664` <br> `find . -perm /222` | Exact 664 permissions. <br> Atleast 664, regardless of other bits (here - the execution bit). <br> Find files that have which has write permissions for atleast user, grpup or others /where someone has write permissions |
Note: for mtimme, and other operations that require a boolean codition, +n, -n, and n indicate greater than, less than or exactly equal to, where n is the integer.


  


<br><br>



<br> 


Add Python Environment stuff here. 

# GitHub 



A practical reference for setting up, using, and troubleshooting Git and GitHub.

---

## Table of Contents
1. [Core Concepts](#1-core-concepts)
2. [First-Time Setup](#2-first-time-setup)
3. [Creating a Repo From Scratch](#3-creating-a-repo-from-scratch)
4. [Cloning an Existing Repo](#4-cloning-an-existing-repo)
5. [The Daily Workflow](#5-the-daily-workflow)
6. [Understanding Branches](#6-understanding-branches)
7. [Understanding `git checkout`](#7-understanding-git-checkout)
8. [Remotes: origin, upstream, and renaming](#8-remotes)
9. [Working on Multiple Projects](#9-working-on-multiple-projects)
10. [Useful Commands Beyond the Basics](#10-useful-commands)
11. [Common Errors and Fixes](#11-common-errors-and-fixes)
12. [Rewriting Git History](#12-rewriting-git-history)

---

## 1. Core Concepts

**Git** is the version control tool that lives on your computer, tracks file versions locally. **GitHub** is a website that hosts your Git repositories remotely so you can access them from anywhere and share them.

- **Repository (repo):** A folder that Git is tracking.
- **Commit:** A saved snapshot of your changes, locally. 
- **Branch:** An independent line of work within a repo. `main` is the default.
- **Remote:** A version of your repo hosted elsewhere (e.g., on GitHub). Usually called `origin`.
- **Push:** Send your local commits up to GitHub.
- **Pull:** Bring GitHub's latest changes down to your local machine.
- **Clone:** Download a full copy of a GitHub repo to your machine.
- **Pull request:** Merge your branhc with the main branch, in remote. 

---

## 2. First-Time Setup
### Windows
Run these once after installing Git:

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global core.editor "code --wait"   # sets VS Code as default editor (optional)
```

**Authenticate with GitHub** (recommended method):

```bash
# Install GitHub CLI, then:
gh auth login
```

Or use a Personal Access Token (PAT) — generate one at GitHub → Settings → Developer Settings → Personal Access Tokens.

### Linux 
Get git on your system
```bash
sudo apt-get update
sudo apt-get install git
```

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

```bash
ssh-keygen -t ed25519 -C "your@email.com"
```
hit enter twice (or add optional passphrase)

Start ssh agent and add key
```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```
copy your key - 
```bash
cat ~/.ssh/id_ed25519.pub | clip.exe
```
Then on GitHub: Settings → SSH and GPG Keys → New SSH Key → paste → Save

Test the connection and authenticate github.com - 
```bash
ssh -T git@github.com
```
Note when cloning on Linux, you will have to use the ssh link (find it on thr right hand side in your repo, in a greenbox that says "<> code", and click ssh, and use that link to clone)
---

## 3. Creating a Repo From Scratch

### Option A: Start on GitHub, then clone (easiest)

1. Go to github.com → click **New Repository**
2. Give it a name, optionally add a README, click **Create**
3. Copy the repo URL (e.g. `https://github.com/yourname/my-project.git`)
4. On your machine:

```bash
git clone https://github.com/yourname/my-project.git
cd my-project
```

The remote (`origin`) is  linked.

---

### Option B: Start locally, then push to GitHub

```bash
# 1. Create and enter your project folder
mkdir my-project
cd my-project

# 2. Initialize Git
git init

# 3. Create your first file
echo "# My Project" > README.md

# 4. Stage and commit
git add .  # Staging means whatever is ready to be commited in the next commit. 
git commit -m "Initial commit"

# 5. Create a new repo on GitHub (without a README), then link it: (look at BONUS to understand what to do if created with a README)
git remote add origin https://github.com/yourname/my-project.git

# 6. Rename your default branch to 'main' (if needed)
git branch -M main

# 7. Push and set upstream tracking
git push -u origin main   # only need to do it once, to set upstream. 

# Bonus: If you need to start tracking a local folder remotely (alreay tracked with git init locally) : either 
# a. Assuming you have added remote already
git fetch origin
git checkout local-branch

git merge origin/main --allow-unrelated-histories # force merge the histories

# Note that git pull = git fetch + git merge. BUT in this case, we needed to use the extra flag for histories to FORCE it to merge. 
# The issue here is that we have made an inital commit on gitHub by adding a README. AND we have made an inital commit locally, and both of these 'mains' dont mathc. 

#b. 
git push -u origin main --force
# simply force the histoeies to merge. 
```


After the `-u` flag, future pushes just need `git push`.

---

## 4. Cloning an Existing Repo

```bash
git clone https://github.com/yourname/repo-name.git

# Clone into a specific folder name:
git clone https://github.com/yourname/repo-name.git my-folder-name
```

Cloning automatically sets `origin` to point back to that GitHub URL.

---

## 5. The Daily Workflow

```bash
git pull origin main         # Get latest changes from GitHub first time, then..
git pull                     # should work from next time onwards
# ... make your edits ...
git status                 # See what changed
git add filename.md        # Stage a specific file
git add .                  # Stage everything
git commit -m "Your message"
git push                   # Send to GitHub
```
### 
For Pull requests, after pushing, on github there should be a merge option, after merging your branch, **delete it on it hub** and delete it **locally**, using the same branch to push and PR again could be messsy.

  

All Commits need a message, will throw an error without a message. 


---

## 6. Understanding Branches

Branches let you work on something without touching `main`. Useful for experiments, features, or drafts.

```bash
git branch                        # List local branches (* = current)
git branch -a                     # List all branches including remote ones

git branch my-feature             # Create a new branch
git checkout my-feature           # Switch to it
# (or do both in one step:)
git checkout -b my-feature

git push -u origin my-feature     # Push branch to GitHub

git merge my-feature              # Merge branch into current branch (usually main)
git branch -d my-feature          # Delete branch locally after merging
git push origin --delete my-feature  # Delete branch on GitHub
```

---

## 7. Understanding `git checkout`

### Switch branches
```bash
git checkout main
git checkout my-branch
```

### Create and switch to a new branch
```bash
git checkout -b new-branch
```

### Discard changes to a file (restore it to last commit)
```bash
git checkout -- filename.md     # This permanently discards your unsaved changes. There's no undo.
```
⚠️ 

### Go back to a specific commit (read-only exploration)
```bash
git checkout abc1234    # use the first 7 chars of a commit hash
git checkout main       # return to normal
```

> **Note:** In newer versions of Git, `git switch` and `git restore` were introduced to split up `checkout`'s jobs — but `checkout` still works fine and is more widely documented.

---

## 8. Remotes: origin, upstream, and renaming

A **remote** is a named link to a GitHub (or other) URL.

```bash
git remote -v                    # See all remotes and their URLs
git remote add origin <url>      # Add a remote named 'origin'
git remote remove origin         # Remove a remote
git remote rename origin new-name  # Rename a remote
```

### Rename `origin` to something more descriptive
```bash
git remote rename origin notes-repo
git push notes-repo main
```

### `origin` vs `upstream`
- **`origin`** = your own fork/copy on GitHub
- **`upstream`** = the original repo you forked from

```bash
git remote add upstream https://github.com/someone-else/original-repo.git
git fetch upstream
git merge upstream/main          # Sync their changes into your local main
```

---

## 9. Working on Multiple Projects

### Know which repo you're in
```bash
pwd                              # Shows your current folder path
git remote -v                    # Shows which GitHub repo this folder is linked to
git status                       # Shows branch and repo state
```

### Know which branch you're on
```bash
git branch                       # * marks your current branch
git status                       # Also shows current branch at the top
```


### Each project is completely independent
Switching folders (`cd`) is all you need. Git state is per-folder. Being on `main` in one project has nothing to do with another.

```bash
cd ~/projects/project-a
git branch         # shows project-a's branches

cd ~/projects/project-b
git branch         # shows project-b's branches
```

---

## 10. Useful Commands Beyond the Basics

### Viewing history
```bash
git log                          # Full commit history
git log --oneline                # Compact one-line view
git log --oneline --graph        # Visual branch/merge graph
git diff                         # See unstaged changes
git diff --staged                # See staged changes (what's about to be committed)
```
> To exit `git log` press the `Q`


### Undoing things
```bash
git restore filename.md          # Discard changes to a file (modern way)
git restore --staged filename.md # Unstage a file (keep changes, just un-add it)

git commit --amend -m "New message"   # Fix your last commit message (before pushing)

git revert abc1234               # Create a new commit that undoes a previous one (safe)
git reset --soft HEAD~1          # Undo last commit, keep changes staged
git reset --hard HEAD~1          # Undo last commit AND discard all changes
```

### Stashing (temporarily set aside work)
```bash
git stash                        # Stash your current changes
git stash list                   # See all stashes
git stash pop                    # Restore the most recent stash
git stash drop                   # Delete the most recent stash without restoring
```

### Renaming `main`
```bash
# Rename local branch
git branch -m main new-name

# If already pushed, update remote too:
git push origin -u new-name
git push origin --delete main    # Delete old name on GitHub
```

### Ignoring files
Create a `.gitignore` file in your repo root:

```bash
# Example .gitignore
.DS_Store 
*.log             # all log files
node_modules/     # specific folders
.env              # all env files
__pycache__/
```

GitHub has template `.gitignore` files for most languages when you create a repo.

---

## 11. Common Errors and Fixes

### "Your local changes would be overwritten by merge"
Your uncommitted changes conflict with incoming changes.
```bash
git stash        # temporarily stash your changes
git pull         # pull the latest
git stash pop    # reapply your changes on top
```
Or if you don't want to keep your changes:
```bash
git checkout -- filename.md    # discard changes to that file
git pull
```

### "Failed to push — tip of your branch is behind its remote counterpart"
Someone else pushed to the same branch, or you pushed from another machine.
```bash
git pull --rebase    # Pull their changes and replay yours on top
git push
```

### Push refused — two remote branches differ only by capitalization (`Main` vs `main`)
Git treats branch names as case-sensitive, so `Main` and `main` can both exist on GitHub as two separate branches. If your local `main` ends up tracking both, Git doesn't know which one to push to and refuses. No new commands — just string together `fetch`, `merge`, `push`, and the remote-branch delete you already know:

```bash
# 1. Make sure both remote branches are fetched
git fetch origin

# 2. Merge the stray branch into the one you want to keep
git checkout main
git merge origin/Main
# If Git complains about unrelated histories, force it:
# git merge origin/Main --allow-unrelated-histories

# 3. Push the combined result
git push origin main

# 4. On GitHub: Settings → Branches → set the default branch to 'main'
#    (GitHub blocks deleting a branch while it is still the default)

# 5. Delete the stray 'Main' on GitHub
git push origin --delete Main
```
The order matters: you can't delete `Main` until `main` is the default on GitHub, so switch the default (step 4) before deleting (step 5).

### "The current branch main has multiple upstream branches, refusing to push"
Different from the case above — this one is a local **config** problem, not two branches on GitHub. Your local `main` has more than one push destination configured (often a leftover `refs/heads/Main` *and* `refs/heads/main`), so Git won't guess which to push to. Reset the upstream to a single branch:

```bash
# See what's configured (you'll usually spot a duplicate line)
git config --get-all branch.main.merge
git config --get-all remote.origin.push

# Clear the tangled tracking config, then re-set it to exactly one branch
git branch --unset-upstream
git push -u origin main          # re-establishes a single upstream

# If it STILL complains, a multi-value push refspec is the culprit:
git config --unset-all remote.origin.push
git push -u origin main
```
After this, plain `git push` works again.

### "fatal: not a git repository"
You're not inside a Git repo. Either `cd` into your project folder, or run `git init` to start tracking the current folder.

### "fatal: remote origin already exists"
```bash
git remote set-url origin https://github.com/yourname/repo.git
```

### Accidentally committed to the wrong branch
```bash
git log --oneline -3              # Find the commit hash
git checkout correct-branch
git cherry-pick abc1234           # Apply that commit here
git checkout wrong-branch
git reset --hard HEAD~1           # Remove it from wrong branch
```

---

## Quick Reference Card

| Command | What it does |
|---|---|
| `git init` | Start tracking a folder |
| `git clone <url>` | Download a remote repo |
| `git status` | See what's changed |
| `git add .` | Stage all changes |
| `git commit -m "msg"` | Save a snapshot |
| `git push` | Upload to GitHub |
| `git pull` | Download from GitHub |
| `git branch` | List branches |
| `git checkout -b name` | Create + switch branch |
| `git checkout main` | Switch to main |
| `git merge branch` | Merge branch into current |
| `git stash` | Temporarily shelve changes |
| `git log --oneline` | View history (compact) |
| `git remote -v` | See remote URLs |
| `git diff` | See unstaged changes |
|`echo "file-name" >> .gitignore`| Ignore (new/untracked) file, do not track it. Doesn't work with files that are already tracked.|
|`git rm --cached file-name.type` or `git rm -r --cached folder/`|Stop tracking a previously tracked file, then add to git ignore to not track it in the future.|

## 12. Rewriting Git History
Use `filter-repo`. It ius a tool to rewrite git history. It goes through **every** commit in your repo, and rewrites it.Download `filter-repo`
- On WSL : 
```bash
# On Ubuntu/WSL
sudo apt-get install git-filter-repo
```
- On powerShell:
```bash
# Python 3 pip
pip3 install git-filter-repo

# Or with python directly
python3 -m pip install git-filter-repo

# Or via conda if you have it
conda install -c conda-forge git-filter-repo
```

### Use Cases
* Remove All files of a type- 
```bash
git filter-repo --path-glob "*.sif" --invert-paths --force
git filter-repo --path-glob "*.csv" --invert-paths --force
```

* Remove an Entire Folder - 
```bash
git filter-repo --path data/ --invert-paths --force
```

* Keep onyl specific files - useful for splitting a repo
```bash
git filter-repo --path scripts/ --force
# Now history only contains scripts/ folder
```

* Rename a file across all of history
```bash
git filter-repo --path-rename old_name.py:new_name.py
```

* Remove sensitive data like passwords
```bash
git filter-repo --replace-text expressions.txt
# expressions.txt contains: password123==>REDACTED
```
Always follow `filter-repo` commands by `git push origin --force`. 

> WARNING:Anyone who has clones the repo, will have conflicts after. Always backup before running filter repo.
## Miscallaneous 
```powershell
git branch --show-current       # shows current branch you are
git rev-parse --show-toplevel   # shows complete path 
git remote get-url origin       # gets origin's url
```


<br><br> <br>



<br><br> <br><br><br><br><br><br><br><br>

<br><br> <br><br><br><br><br><br><br><br>




---

# 🧠 The Linux Efficiency Toolkit



---


These alone make you 3x faster.

---






---



## 🖥 6️⃣ Window / Desktop Shortcuts (Linux Desktop)

| Action           | Shortcut             |
| ---------------- | -------------------- |
| Switch apps      | `Alt + Tab`          |
| Close window     | `Alt + F4`           |
| Switch workspace | `Ctrl + Alt + Arrow` |
| Open terminal    | `Ctrl + Alt + T`     |
| Lock screen      | `Super + L`          |

(Super = Windows key)

---

## 🧵 7️⃣ tmux (HPC Superpower)

If you learn only ONE advanced tool — learn this.

| Action           | Command              |
| ---------------- | -------------------- |
| Start tmux       | `tmux`               |
| Detach session   | `Ctrl + B`, then `D` |
| List sessions    | `tmux ls`            |
| Reattach         | `tmux attach -t 0`   |
| Split vertical   | `Ctrl + B`, `%`      |
| Split horizontal | `Ctrl + B`, `"`      |

This lets you:

* Disconnect from SSH
* Come back later
* Job still running

Absolute HPC necessity.

---

## 🔍 8️⃣ Monitoring & Watching

| Action             | Command         |
| ------------------ | --------------- |
| Monitor repeatedly | `watch command` |
| Show open files    | `lsof`          |
| See memory         | `free -h`       |
| See CPU info       | `lscpu`         |
| Show disk usage    | `du -sh *`      |

---

## 🧠 9️⃣ Text Navigation in Terminal Output

| Action         | Key              |
| -------------- | ---------------- |
| Scroll up      | `Shift + PageUp` |
| Search in less | `/pattern`       |
| Next match     | `n`              |
| Quit less      | `q`              |

---

## 🏎 10️⃣ Efficiency Mindset Commands

These separate good from elite:

| Tool     | Why                 |
| -------- | ------------------- |
| `alias`  | Create shortcuts    |
| `xargs`  | Automate pipelines  |
| `awk`    | Data parsing        |
| `sed`    | Text editing        |
| `screen` | Alternative to tmux |
| `ulimit` | Resource debugging  |
| `time`   | Measure performance |

---

# 🚀 The Real “Super User” Behaviors

It’s not memorizing commands.

It’s:

* Using `Tab` instead of typing
* Using `Ctrl + R` instead of retyping
* Using `sudo !!`
* Using pipes instead of manual inspection
* Using `tmux` instead of keeping laptop open
* Reading `man` pages fast

---

# 🧠 If You Master Just These 10

1. Ctrl+C
2. Ctrl+R
3. Ctrl+A / Ctrl+E
4. Ctrl+U / Ctrl+K
5. Tab completion
6. sudo !!
7. cd -
8. grep -r
9. pipes ( | )
10. tmux

You’ll feel 5x faster instantly.

---

If you want, I can now:

* Turn this into a clean `.md` file you can paste into your notes
* Or create a printable “Linux Super User Cheat Sheet”
* Or make an HPC-specific version

What would you prefer?
