# Linux File Management

## The Linux file system

Let's review some basic commands that you can use to navigate and control the file system:

- `cd` - change directory.
- `mkdir [options] <Directory>` - create directory.
- `rmdir [options] <Directory>` - remove directory.
- `touch [options] <filename>` - create a file.
- `cp [options] <source> <destination>` - copy file to another location.
- `mv [options] <source> <destination>` - move file to another location.
- `rm [options] <file>` - remove file from the fs.
- `cat <filename>` - print the content of a given file.

**Note:** there is no undo! The Linux cli does not have an undo feature. Perform destructive actions carefully.


## Hidden files

If the file or directory name begins with a `.` (full stop) then it is considered to be hidden.

```console
myuser@hostname:~$ cd ~
myuser@hostname:~$ ls -l
drwxrwxr-x   4 myuser myuser       4096 Jul 25  2022  aa
-rw-rw-r--   1 myuser myuser     191005 Feb  7  2022  aaa.pdf
-rw-rw-r--   1 myuser myuser   60085406 Oct 23 11:05  adobe.deb
...
myuser@hostname:~$ ls -la
drwxrwxr-x   4 myuser myuser       4096 Jul 25  2022  aa
-rw-rw-r--   1 myuser myuser     191005 Feb  7  2022  aaa.pdf
drwx------   3 myuser myuser       4096 Oct 23 11:21  .adobe
-rw-rw-r--   1 myuser myuser   60085406 Oct 23 11:05  adobe.deb
...
```

The above example uses the `ls` command with the `-a` flag to include hidden files in the list.
Note the directory `.adobe`, which is hidden. 
By default, `ls` doesn't print hidden files.

**Tip!** You can combine multiple options on the same flag: `ls -l -a` is equivalent to `ls -la`.


## File permissions

Permissions specify what a particular user may or may not do to a given file or directory.
On a Linux system, every file is owned by a **user**, a **group** and “**others**”.

```console
myuser@hostname:~$ ls -l
-rw-rw-r--  1   myuser  users 5 Jan 15 12:39 To_Do
-rwxr-xr-x  1   root    root 45948 Aug 9 15:01 /bin/ls*
```

In the above example, the first file is a regular file (first dash).
Users with username `myuser` or users belonging to the group `users` can read and write (change/move/delete) the file, but they can't execute it (second and third dash). All other users are only allowed to read this file, but they can't write or execute it (fourth and fifth dash).

The second example is an executable file, the difference: everybody can run this program, but you need to be `root` to change it.

On a Linux system there are only 2 people usually who may change the permissions of a file or directory. 
The owner of the file or directory and the root user. The root user is a superuser who is allowed to do anything and everything on the system.

Linux permissions dictate 3 things you may do with a file, read (`r`), write (`w`) and execute (`x`). They are referred to in Linux by a single letter each.

For every file we define 3 sets of people for whom we may specify permissions.

- user (`u`) - a single person who owns the file. (typically the person who created the file but ownership may be granted to someone else by certain users)
- group (`g`) - every file belongs to a single group.
- others (`o`) - everyone else who is not in the group or the owner.

Use the `chmod` command to change file's permissions:

```console
myuser@hostname:~$ ls -l
-rw-rw-r--  1   myuser  users 5 Jan 15 12:39 hello
myuser@hostname:~$ chmod u+x hello
myuser@hostname:~$ ls -l
-rwxrw-r--  1   myuser  users 5 Jan 15 12:39 hello
```

The logic behind the command:

1. Who are we changing the permission for? `[ugoa]` - user (or owner), group, others, all
2. Are we granting or revoking the permission - indicated with either a plus (`+`) or minus (`-`)
3. Which permission are we setting? - read (`r`), write (`w`) or execute (`x`).

 
## The root user and the `sudo` command

Sometimes, you need to execute a command as another user. For example:

```console
myuser@hostname:~$ ls -l
-rw-------  1   john  johnsfriends 5 Jan 23 12:39 phone
myuser@hostname:~$ cat phone
cat: phone: Permission denied
```

In the above example, your personal user is `myuser` (according to the name in the prompt), but the file `phone` is owned by the user `john`. Not only it is owned by `john`, according to the file's permissions, only john (or users under the group `johnsfriends`) can read/write to the file.

The `sudo` command is short for “**switch user and do**”. In a single command, you can switch to a specific user, perform a command on behalf of that user, and “return” to your user. Here is an example:

```console
myuser@hostname:~$ sudo -u john cat phone
+91524869328
```

In the above example, we under the hood switched to the user `john`, executed `cat phone` on his behalf, then back to our user, `myuser`. In a single command, quite useful.

According to `sudo`'s help page, if you don't specify a user (using the `-u` flag), the default user that you are switching to is `root`.

### The root user

The `root` user is the administrative user in a Linux-based operating system, including Ubuntu. The root user has complete control over the system and can perform any operation or command, including modifying system files and processes.

In Ubuntu, the `root` user is disabled by default for security reasons. Instead, administrative tasks are typically performed using the `sudo` command.  How does it work?

Every user that is a member of a special group called `wheel`, can use the `sudo` command to execute commands on behalf of the root user, without actually login to this strong user. Your default linux user is a member of the group sudo.

It's important to exercise caution when using the root user account, as any command or operation executed with root privileges has the potential to cause significant damage to the system if performed incorrectly. It's recommended to use the root user account only when necessary and to carefully consider the potential consequences of each action before executing it.

## IO Redirect

### The `>` and `>>` operators

Sometimes you will want to put the output of a command in a file, instead of printing it to the screen. You can do so with the `>` operator:

```console
myuser@hostname:~$ echo Hi
Hi
myuser@hostname:~$ echo Hi > myfile
myuser@hostname:~$ cat myfile
Hi
```

The `>` operator overwrites the file if it already contains some content. If you want to append to the end of the file, do:

```console
myuser@hostname:~$ echo Hi again >> myfile
myuser@hostname:~$ cat myfile
Hi
Hi again
myuser@hostname:~$ date >> myfile
myuser@hostname:~$ cat myfile
Hi
Hi again
IST 10:06:02 2020 Jan 01
```

### The `|` operator

Sometimes you may want to issue another command on the output of one command. You can do so with the `|` (pipe) operator:

```console
myuser@hostname:~$ cat myfile | grep again
Hi again
```

In the above command, the output of the `cat` command is the input to the `grep` command.
Both `>`, `>>`, and `|` are called IO redirection operators. There are [more operators](https://tldp.org/LDP/abs/html/io-redirection.html)... but the above 3 are the most common.

### Linux `grep` command and Regular Expressions

#### Regex

Regular expressions (Regex) allow us to create and match a pattern in a given string. Regex are used to replace text in a string, validate string format, extract a substring from a string based on a pattern match, and much more!

Regex is out of this course' scope, but you are highly encouraged to learn regex yourself. There are so many systems and configurations that you'll be required for regex skills!

Learn Regex: https://regexone.com/


#### `grep` - Global Regular Expression Print

A simple but powerful program, `grep` is used for filtering files content or input lines, and prints certain regex patterns.

Let's print the content of `/var/log/auth.log`. This file is a log file that records information about system authentication events. The below commands create some authentication event (by creating the `/test` directory and file in it), then printing the content of `auth.log`.

```console
myuser@hostname:~$ sudo mkdir /test
myuser@hostname:~$ sudo touch /test/aaa
myuser@hostname:~$ cd /var/log
myuser@hostname:/var/log$ cat auth.log
...
Mar  7 19:17:01 hostname CRON[2076]: pam_unix(cron:session): session closed for user root
Mar  7 19:33:10 hostname sudo:   myuser : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/usr/bin/mkdir /test
```

The above output shows information regarding the executed `sudo` command. `myuser` is the user who performed the `sudo` command, the working dir is `/var/log` and the command coming after sudo was `mkdir /test`.
Now let's demonstrate the power of the `grep` command.

Print all lines contain the word "sudo":

```console
myuser@hostname:/var/log$ grep sudo auth.log
Mar  7 19:17:01 hostname CRON[2076]: pam_unix(cron:session): session closed for user root
Mar  7 19:33:10 hostname sudo:   myuser : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/usr/bin/mkdir /test
...
```

But what if we want to print all "sudo" events while the command coming after sudo is `mkdir`? We can do it using regular expressions:

```console
myuser@hostname:/var/log$ grep -E "sudo: .*COMMAND=.*mkdir" auth.log
Mar  7 19:33:10 hostname sudo:   myuser : TTY=pts/0 ; PWD=/var/log ; USER=root ; COMMAND=/usr/bin/mkdir /test
```

The above example uses the `.*` match pattern to catch what we want. `.` means "any single character except a line break", `*` means "0 or more repetitions of the preceding symbol".

Another example - list all users that executed commands using `sudo`:

```console
# first (bad) try - the second output line is unwanted
myuser@hostname:/var/log$ grep -Eo "sudo:.*:" auth.log
sudo:   myuser :
sudo: pam_unix(sudo:session):
...

# second (success) try
myuser@hostname:/var/log$ grep -Eo "sudo:\s+\w+\s+:" auth.log
sudo:   myuser :
```

# Exercises

### :pencil2: Playing with permissions


1. Create a new linux user by `useradd -m -s /bin/bash <username>` (change `<username>` to your username. `-m` creates a home directory, `-s /bin/bash` sets Bash as the default shell). 
2. Set a new password for your user: `passwd <username>`
3. Login as the user you've created: `su -l <username>`. Make sure the prompt is looking like `<username>@hostname...`. 
4. Under `/etc/passwd` and `/etc/group` you'll find a list of all your users and groups on your system, respectively. Use `cat` and `grep` to verify there are user and group exist for your user.
5. Try to create the `/testdir`. What happened? Why? Are you able with `sudo`?
6. Add your user to the `wheel` group. Try to create again with `sudo`.


1. Use the `ls -l` command to list the files in your home dir along with their permissions.
2. Create a **file** and **directory** in your user's home directory.
3. Why are all your directories having an execute (`x`) permission? Use the `chmod` command to remove the `x` permission of some directory, what happened?
4. Use the `chmod` command to remove **ANY** permissions (`rwx`) of the file.
5. Use `sudo` to print the content of the file even though no one has permissions to do it.

### :pencil2: IO redirection basics

1. Create a file called `fruits.txt` with the contents "apple banana cherry"
2. Use `>` to write the contents of `fruits.txt` to a new file called `output.txt`.
3. Use `>>` to append the contents of `fruits.txt` again to `output.txt`.
4. Use `|` to pipe the output of cat `output.txt` to grep `banana`. How many times does `banana` appear?
5. Use `grep` to search for `APPLE` (upper cases) in `output.txt`. Did the search succeed?
6. Use `grep` to display all lines in `output.txt` that don't contain banana.

### :pencil2: `grep` on file

Create the file `~/bashusers.txt`, which contains lines from the `/etc/passwd` file which contain the text “/bin/bash”.


[linux_links]: https://exit-zero-academy.github.io/DevOpsTheHardWayAssets/img/linux_links.png
