# What are daemontools?
daemontools are tools that monitor daemons, and are tools by qmail author D.J.B. The advantage is that if you monitor it with daemontools, it will restart automatically. The notes are as follows.
* You cannot manage background daemons.
* For this reason, do not add & to the background of processes running from run.

For details, refer to the official website. https://cr.yp.to/daemontools.html

In this repository, I fixed it so that daemontools works on Termux (Android).

# Install

In this installation example, create a bin in your home directory, extract it there, and install it.

```$ cd
$ mkdir bin
$ cd bin

$ git clone https://github.com/take-i/termux-daemontools.git
$ cd termux-daemontools/admin/daemontools-0.76/
$ package/install
```

# Where are the commands?

The command is under the `$PREFIX/bin` directory under termux, which is a symbolic link.

```
$ which svscan
/data/data/com.termux/files/usr/bin/svscan
```

Also, this symbolic link will be `$PREFIX/command/`

```
$ ls -l /data/data/com.termux/files/usr/bin/svscan
lrwxrwxrwx 1 u0_a257 u0_a257 46 Oct 21 15:10 /data/data/com.termux/files/usr/bin/svscan -> /data/data/com.termux/files/usr/command/svscan
```
In other words, the body of the command is under the built directory. Be careful when deleting the built directory.

```
$ tree $PREFIX/command/
├── envdir -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/envdir
├── envuidgid -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/envuidgid
├── fghack -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/fghack
├── multilog -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/multilog
├── pgrphack -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/pgrphack
├── readproctitle -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/readproctitle
├── setlock -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/setlock
├── setuidgid -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/setuidgid
├── softlimit -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/softlimit
├── supervise -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/supervise
├── svc -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svc
├── svok -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svok
├── svscan -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svscan
├── svscanboot -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svscanboot
├── svstat -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/svstat
├── tai64n -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/tai64n
└── tai64nlocal -> /data/data/com.termux/files/home/bin/termux-daemontools/admin/daemontools/command/tai64nlocal
```
# Start daemontools

It is not essential, but if you want to explicitly describe the PATH, add it as follows.

~/.bashrc
```
::
#djb tool daemontools
export PATH=$PATH:$PREFIX/command
```
You can start Daemontools as root if you are rooting your smartphone. In that case,

`$ nohup sudo svscan $PREFIX/service/ &`

When starting as a general user

`$ nohup svscan $PREFIX/service/ &`


# Example setting

For example, here is a sample to start crond with daemontools.
Create a boot directory in your home directory. Later, I will start it by putting a Symbolic link on `$PREFIX/service/`
The actual directory can be anywhere you like, such as `$PREFIX/var/boot/`

```
$ cd
$ mkdir -p boot/crond
$ cd boot/crond

$ vi run
```

```
#!/bin/sh

export PATH=$PATH:$HOME/bin
export PATH=$PATH:$PREFIX/command
export PREFIX=/data/data/com.termux/files/usr

exec 2>&1
exec > /dev/null

# start up script example.
# need pkg install cronie
exec crond -n
```
I started crond in the foreground with the `-n` option to prevent the daemon from starting.

```
$ cd ../
$ chmod +t crond
```

# Start-up

It starts when you make a symbolic link.

`$ ln -s $HOME/boot/crond $PREFIX/service/`

# Confirmation of startup

`$ svstat $PREFIX/service/crond`

```
$ ps axu | grep crond
u0_a257  20259  0.2  0.0  33936  2692 ?        S<    1970   0:14 supervise crond
u0_a257  32339  0.1  0.0  34852  2984 ?        S<    1970   0:00 crond -n
```
# Temporary suspension of service

`$ svc -d $PREFIX/service/crond`

# Resume from suspension

`$ svc -u $PREFIX/service/crond`

# Persistent service removal

Remove the symbolic link from the watch directory.

```
$ cd $PREFIX/service/
$ unlink crond
```

# Notes

I don't think it will work for general users (users who have installed termux) such as setuidgid. Also, not all commands have been tested, so there is no guarantee of operation.

|    | Command       | Overview                  | Tested | Works as root | Works as user | Memo             |
|----|---------------|------------------------------------|--------|---------------|---------------|------------------|
| 1  | envdir        | Start the command after reading the environment variables in the specified directory                 | No     |               |               |                                     |
| 2  | envuidgid     | Start command with uid and gid of specified account                 | No     |               |               |                                     |
| 3  | fghack        | Prevent moving to the background                              | No     |               |               |                                     |
| 4  | multilog      | Create tai64n format log                             | No     |               |               |                                     |
| 5  | pgrphack      | Run the program in another process group                         | No     |               |               |                                     |
| 6  | readproctitle | I get the error message readproctitle in ps              | No     |               |               |                                     |
| 7  | setlock       | Lock the file and launch another program                        | No     |               |               | Can be used to prevent multiple boot                      |
| 8  | setuidgid     | Launch another program with the uid and gid of the specified account            | Yes    | O             | X             | Specifications that can only be used by root                      |
| 9  | softlimit     | Start another program with resource restrictions                         | No     |               |               |                                     |
| 10 | supervise     | A program that starts and monitors services. Start by svscan          | Yes    | O             | O             |                                     |
| 11 | svc           | Control services monitored by supervise                 | Yes    | O             | O             |                                     |
| 12 | svok          | Check if supervise is running                      | Yes    | O             | O             | `svok $PREFIX/service/test ; echo $?` |
| 13 | svscan        | Start and monitor service gatherings                          | Yes    | O             | O             |                                     |
| 14 | svscanboot    | Start svscan in the / service directory and pipe to readproctitle | X      |               |               |                                     |
| 15 | svstat        | Output the status of services monitored by supervise              | Yes    | O             | O             |                                     |
| 16 | tai64n        | Accurate time stamp in TAI64N format                    | No     |               |               |                                     |
| 17 | tai64nlocal   | Convert TAI64N format timestamps to human readable format               | No     |               |               |                                     |