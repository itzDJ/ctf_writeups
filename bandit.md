# OverTheWire: Bandit
- [game link](https://overthewire.org/wargames/bandit/)
- ssh command: `ssh bandit0@bandit.labs.overthewire.org -p 2220`
  - NOTE: change number in username to level number

## level 0 → level 1
### problem
The password for the next level is stored in a file called readme located in the home directory. Use this password to log into bandit1 using SSH. Whenever you find a password for a level, use SSH (on port 2220) to log into that level and continue the game.
### solution
- ssh into machine with password bandit0 (this password is given on website. no more passwords will be logged in this writeup)
- run `ls -la`
- password is in readme (view with cat or vim)

## level 1 → level 2
### problem
The password for the next level is stored in a file called - located in the home directory
### solution
- run `ls -la`
- run `vim .` and navigate to the file named "-" where the password is stored

## level 2 → level 3
### problem
The password for the next level is stored in a file called --spaces in this filename-- located in the home directory
### solution
- run `ls -la`
- run vim on cwd then select file or merely run `cat ./--spaces\ in\ this\ filename--`

## level 3 → level 4
### problem
The password for the next level is stored in a hidden file in the inhere directory.
### solution
- cd into visible directory
- cat hidden file

## level 4 → level 5
### problem
The password for the next level is stored in the only human-readable file in the inhere directory. Tip: if your terminal is messed up, try the “reset” command.
### solution
- cd into visible directory
- run `file ./*` to see the file type of all the files
- cat the file with human readable type (ASCII text)

## level 5 → level 6
### problem
The password for the next level is stored in a file somewhere under the inhere directory and has all of the following properties:
- human-readable
- 1033 bytes in size
- not executable
### solution
- `cd inhere`
- Check for non executable files of 1033 bytes: `find . -type f -size 1033c ! -executable`
- Cat only returned file

## level 6 → level 7
### problem
The password for the next level is stored somewhere on the server and has all of the following properties:
- owned by user bandit7
- owned by group bandit6
- 33 bytes in size
### solution
- need to search entire file system for file with multiple attributes (specific user, group, and size)
- run `find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null`
- NOTE: command returns many files with errors like "Permission denied" so it's best to redirect standard error to /dev/null (which disgards them)
- Cat the only revealed file

## level 7 → level 8
### problem
The password for the next level is stored in the file data.txt next to the word millionth
### solution
- password is next to word millionth
- run `cat data.txt | grep millionth`

## level 8 → level 9
### problem
The password for the next level is stored in the file data.txt and is the only line of text that occurs only once
### solution
- uniq command compares adjacent lines and the -u flag checks that each line is followed by a unique line
- because subsequent lines are checked, the data must be sorted first
- run `sort data.txt | uniq -u`

## level 9 → level 10
### problem
The password for the next level is stored in the file data.txt in one of the few human-readable strings, preceded by several ‘=’ characters.
### solution
- strings command can find human readable strings in binary files
- run `strings data.txt | grep =`
  - NOTE: if several equals are used instead of one as per the instructions, the password is even more apparent
- password is one of relatively few returned lines (preceded by a bunch of equal signs)

## level 10 → level 11
### problem
The password for the next level is stored in the file data.txt, which contains base64 encoded data
### solution
- check help page for base64
- peep -d flag to decode file
- run `base64 -d data.txt`

## level 11 → level 12
### problem
The password for the next level is stored in the file data.txt, where all lowercase (a-z) and uppercase (A-Z) letters have been rotated by 13 positions
### solution
- found solution in bash as well as python
- rot13 in bash: `cat data.txt | tr "A-Za-z" "N-ZA-Mn-za-m"`
- rot13 in python3 REPL
```python
import codecs
with open("data.txt") as f:
    codecs.decode(f.read(), "rot_13")
```

## level 12 → level 13
### problem
The password for the next level is stored in the file data.txt, which is a hexdump of a file that has been repeatedly compressed. For this level it may be useful to create a directory under /tmp in which you can work. Use mkdir with a hard to guess directory name. Or better, use the command “mktemp -d”. Then copy the datafile using cp, and rename it using mv (read the manpages!)
### solution
- based on rules of the game and permissions on the filesystem, files cannot be created and removed just anywhere on the device
- run `mktemp -d` to create a temporary directory and change into it
- run `xxd -r ~/data.txt data` to reverse the hex dump located in ~/data.txt and create a binary file from it called data in our temp dir
- run the file command on data and decompress based on file type
  - if gzip, use mv to rename with .gz then run `gzip -d`
  - if bzip2, run `bzip2 -d`
  - if tar, run `tar -xf`
- continue that process until the file type is ASCII text then cat the file
- cleanup any temp files

## level 13 → level 14
### problem
The password for the next level is stored in /etc/bandit_pass/bandit14 and can only be read by user bandit14. For this level, you don’t get the next password, but you get a private SSH key that can be used to log into the next level. Look at the commands that logged you into previous bandit levels, and find out how to use the key for this level.
### solution
- copy contents of ssh key located in home directory of bandit13 to somewhere on our host machine (maybe to file named `ssh_key`)
- increase permission strictness on the private key: `chmod 600 ssh_key`
- ssh into next level with the key: `ssh -i ssh_key bandit14@bandit.labs.overthewire.org -p 2220`

## level 14 → level 15
### problem
The password for the next level can be retrieved by submitting the password of the current level to port 30000 on localhost.
### solution
- After signed into level from ssh private key, `cat /etc/bandit_pass/bandit14` as mentioned in previous level
- Send that password to port 30000 on localhost using netcat: `cat /etc/bandit_pass/bandit14 | nc localhost 30000`
- Password for next level is sent in return

## level 15 → level 16
### problem
The password for the next level can be retrieved by submitting the password of the current level to port 30001 on localhost using SSL/TLS encryption.
### solution
- use nmap's ncat instead of nc for easier ssl connection
- run: `cat /etc/bandit_pass/bandit15 | ncat --ssl localhost 30001`

## level 16 → level 17
### problem
The credentials for the next level can be retrieved by submitting the password of the current level to a port on localhost in the range 31000 to 32000. First find out which of these ports have a server listening on them. Then find out which of those speak SSL/TLS and which don’t. There is only 1 server that will give the next credentials, the others will simply send back to you whatever you send to it.
### solution
- find open ports from 31000 to 32000: `nmap -p 31000-32000 localhost`
- connect to the returned ports one by one: `openssl s_client -connect localhost:<port> -quiet`
- if a connection is made, paste the password for current level and click enter
- repeat until an RSA private key is returned
- save that key to host machine and modify permissions to `chmod 600` just like level 13

## level 17 → level 18
### problem
There are 2 files in the homedirectory: passwords.old and passwords.new. The password for the next level is in passwords.new and is the only line that has been changed between passwords.old and passwords.new
### solution
- run: `diff passwords.old passwords.new`
- output shows the files only differ on one line
- first string is from old file; second string is from new file (second is password for next level)

## level 18 → level 19
### problem
The password for the next level is stored in a file readme in the homedirectory. Unfortunately, someone has modified .bashrc to log you out when you log in with SSH.
### solution
- need to get contents of readme file before .bashrc logs out
- execute a single command over ssh (since it's non-interactive, .bashrc never loads)
- `ssh bandit18@bandit.labs.overthewire.org -p 2220 'cat readme'`

## level 19 → level 20
### problem
To gain access to the next level, you should use the setuid binary in the homedirectory. Execute it without arguments to find out how to use it. The password for this level can be found in the usual place (/etc/bandit_pass), after you have used the setuid binary.
### solution
- binary in home directory allows running commands as the user bandit20
- run `./bandit20-do cat /etc/bandit_pass/bandit20`

## level 20 → level 21
### problem
There is a setuid binary in the homedirectory that does the following: it makes a connection to localhost on the port you specify as a commandline argument. It then reads a line of text from the connection and compares it to the password in the previous level (bandit20). If the password is correct, it will transmit the password for the next level (bandit21).
### solution
- cat password from level 20 to netcat which sends it to whoever connects to port 1337
- `cat /etc/bandit_pass/bandit20 | nc -l -p 1337&`
- run the given command in home directory to connect to the port netcat is on and the password for next level will be returned
- `./suconnect 1337`

## level 21 → level 22
### problem
A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.
### solution
- look in /etc/cron.d/ for config related to level
- run `ls /etc/cron.d/`
- run `cat /etc/cron.d/cronjob_bandit22`
- cronjob shows shell script is running at `/usr/bin/cronjob_bandit22.sh`
- run `cat /usr/bin/cronjob_bandit22.sh`
- the script shows the bandit22 password is copied to a tmp dir. cat the tmp dir for the next password

## level 22 → level 23
### problem
A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.
### solution
- similar start to previous problem
- look in /etc/cron.d/ for config related to level
- run `ls /etc/cron.d/`
- run `cat /etc/cron.d/cronjob_bandit23`
- cronjob shows shell script is running at `/usr/bin/cronjob_bandit23.sh`
- run `cat /usr/bin/cronjob_bandit23.sh`
- shell script shows location to next password (so next two commands are from that shell script)
- run a modified line (bandit23 instead of bandit22) in the shell script to find the location: `mytarget=$(echo I am user bandit23 | md5sum | cut -d ' ' -f 1)`
- cat to find next password: `cat /tmp/$mytarget`

## level 23 → level 24
### problem
A program is running automatically at regular intervals from cron, the time-based job scheduler. Look in /etc/cron.d/ for the configuration and see what command is being executed.

NOTE: This level requires you to create your own first shell-script. This is a very big step and you should be proud of yourself when you beat this level!

NOTE 2: Keep in mind that your shell script is removed once executed, so you may want to keep a copy around…
### solution
- similar start to previous problems
- look in /etc/cron.d/ for config related to level
- run `ls /etc/cron.d/`
- run `cat /etc/cron.d/cronjob_bandit24`
- cronjob shows shell script is running at `/usr/bin/cronjob_bandit24.sh`
- run `cat /usr/bin/cronjob_bandit24.sh`
- shell script shows all scripts are run then deleted in /var/spool/$myname/foo
- create temp folder to create script and save result: `mktemp -d`
- cd into that temp directory
- run `vim exploit.sh`
```bash
#!/bin/bash

cat /etc/bandit_pass/bandit24 > /tmp/<TEMP_DIR>/password
```
- fix permissions: `chmod +x exploit.sh`
- run `chmod 777 /tmp/<TEMP_DIR>`
- copy script from tmp to cron executed location: `cp exploit.sh /var/spool/bandit24/foo/`
- in a minute when cron executes, a password file will create in /tmp/<TEMP_DIR> with the password to next level

## level 24 → level 25
### problem
A daemon is listening on port 30002 and will give you the password for bandit25 if given the password for bandit24 and a secret numeric 4-digit pincode. There is no way to retrieve the pincode except by going through all of the 10000 combinations, called brute-forcing.
You do not need to create new connections each time
### solution
- create and cd into temp dir with `mktemp -d`
- create exploit shell script to send the password from current level and every four digit code filtering out responses that contain "Wrong"
```bash
#!/bin/bash

password=$(cat /etc/bandit_pass/bandit24)
for pin in {0000..9999}; do
    echo "$password $pin"
done | nc localhost 30002 | grep -v "Wrong"
```

## level 25 → level 26
### problem
Logging in to bandit26 from bandit25 should be fairly easy… The shell for user bandit26 is not /bin/bash, but something else. Find out what it is, how it works and how to break out of it.
### solution
- in bandit25, checek the shell for bandit26: `cat /etc/passwd | grep bandit26`
- shell is /usr/bin/showtext
- cat that file which we see runs more on login then exits
- copy bandit26.sshkey in $HOME to host computer
- to prevent instant exit, shrink terminal until more cannot output whole file at once
- ssh into bandit26 on the shrunk terminal: `ssh -i bandit26.sshkey bandit26@bandit.labs.overthewire.org -p 2220`
- assuming the terminal was shrunk correct and more was put in interative mode, click `v` to open vim
- run following commands in vim to open shell
```
:set shell=/bin/bash
:shell
```
- now that a shell is open, cat /etc/bandit_pass/bandit26

## level 26 → level 27
### problem
Good job getting a shell! Now hurry and grab the password for bandit27!
### solution
- while in the shell from the previous level, run `./bandit27-do cat /etc/bandit_pass/bandit27`

## level 27 → level 28
### problem
There is a git repository at ssh://bandit27-git@bandit.labs.overthewire.org/home/bandit27-git/repo via the port 2220. The password for the user bandit27-git is the same as for the user bandit27.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.
### solution
- run `git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo`
- cat the README in the repo

## level 28 → level 29
### problem
There is a git repository at ssh://bandit28-git@bandit.labs.overthewire.org/home/bandit28-git/repo via the port 2220. The password for the user bandit28-git is the same as for the user bandit28.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.
### solution
- run `git clone ssh://bandit28-git@bandit.labs.overthewire.org:2220/home/bandit28-git/repo`
- cat the README.md in the repo
- password is redacted so check `git log` for previous commits
- there was a commit with message "add missing data"
- run `git show <COMMIT_ID>` on that commit and copy password

## level 29 → level 30
### problem
There is a git repository at ssh://bandit29-git@bandit.labs.overthewire.org/home/bandit29-git/repo via the port 2220. The password for the user bandit29-git is the same as for the user bandit29.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.
### solution
- run `git clone ssh://bandit29-git@bandit.labs.overthewire.org:2220/home/bandit29-git/repo`
- cat the README.md in the repo
- nothing there and nothing in `git log`
- check for additional branches with `git branch -a`
- switch into dev branch with `git checkout dev`
- password is in README.md

## level 30 → 31
### problem
There is a git repository at ssh://bandit30-git@bandit.labs.overthewire.org/home/bandit30-git/repo via the port 2220. The password for the user bandit30-git is the same as for the user bandit30.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.
### solution
- run `git clone ssh://bandit30-git@bandit.labs.overthewire.org:2220/home/bandit30-git/repo`
- nothing in README.md, log, or branches
- check for tags (reference that points to specific commits): `git tag`
- uncovers tag named 'secret'
- run `git tag secret` to reveal password

## level 31 → 32
### problem
There is a git repository at ssh://bandit31-git@bandit.labs.overthewire.org/home/bandit31-git/repo via the port 2220. The password for the user bandit31-git is the same as for the user bandit31.

From your local machine (not the OverTheWire machine!), clone the repository and find the password for the next level. This needs git installed locally on your machine.
### solution
- run `git clone ssh://bandit31-git@bandit.labs.overthewire.org:2220/home/bandit31-git/repo`
- README.md says to create and push a text file to the remote repo
- .gitignore exists and ignores all text files
- After creating the text file with the text as instructed in README.md, force add the file to be tracked: `git add -f key.txt`
- run `git commit -m "any message here"`
- run `git push`

## level 32 → 33
### problem
After all this git stuff, it’s time for another escape. Good luck!
### solution
- ssh into problem
- prompted with "WELCOME TO THE UPPERCASE SHELL"
- put in shell with all commands forced uppercase making it so commands like `ls` don't work because the shell reads in `LS`
- run `$0` which is the environment variable for name of the shell
- this puts us in a new shell that's not forced uppercase
- by running `bash`, we can see we're logged in as user bandit33 (or we can just run `whoami`)
- cat password for bandit33: `cat /etc/bandit_pass/bandit33`

## level 33 → 34
### problem
At this moment, level 34 does not exist yet.
