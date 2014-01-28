---
layout: post
title: Git on the Server
tags: git git-server git-hooks git-shell tutorial
---

So lately I've pushed aside working on Ruby on Rails to get my servers 
up-to-par, which has included getting git all setup for some revision control. 
For those not aware it, put simply, [git](http://git-scm.com) is a system 
in which you are able to track revisions made to files.  This makes it ideal
for developers in that a developer can: make a new file, write changes to an 
exisiting file, and then save the file with a comment attached to it to 
describe what changes were done to the file.  Then, system administrators are
able to setup environment chains (such as development->testing->production) to 
receive this code from developers, try it out on the test systems (to ensure 
code works live and to test it with new architecture), and then to deploy it 
live!

The benefits of these revision control systems are **ENORMOUS**.

This means that a team of developers can all be working on the same set of 
files, and collaborate simultaneously on them! (The details of this can be 
found on the [git website](http://git-scm.com) and, of course, just by using 
it yourself!)  And at this point in time, pretty much every company or person 
who is serious about their code uses SCM (Software Configuration Management).

So let's cut to the chase: below is what you need to setup git on your own 
server.

Setting up git
==============
Though these directions are for [CentOS(6.5)](www.centos.org); if you're not 
on this flavor of GNU/Linux, you may need to make a few minor tweaks to the 
commands for them to suit your distribution.

---

Install git
-----------
`sudo yum install git`

---

The git user
-------------------
Create the user  
`sudo adduser git`

Switch to the user  
`sudo su git`

Secure the git user's home folder from outside access  
`chmod g-w,o-w ~`

Create the ssh directory skeleton, and set permissions.  
 `cd ~ && mkdir .ssh`
 `chmod 700 ~/.ssh`

Create the file where __ALL__ the developer's SSH keys will placed, and set 
it's permissions.  
 `touch ~/.ssh/authorized_keys`
 `chmod 600 ~/.ssh/authorized_keys`

Add in the the developer(s) public SSH keys, as they will be pushing code to 
the git user who holds the git repository.  
 `echo /PATH/TO/YOUR/.ssh/id_rsa.pub >> /home/git/.ssh/authorized_keys"`

---

The git repository
------------------
To avoid diving into a tutorial on setting up webserver/permissions/etc., 
we will just be setting up the repository in the git user's home directory. 
Assuming you are still logged in as the git user, and that this is a NEW 
repository we are setting up:

Make the git repo  
`mkdir ~/REPONAME.git`
`cd REPONAME.git`
`git --bare init`


At this point, our git repository is initialized and ready for use.  All 
that is needed is for us to setup the final events to occurr after receiving 
a new piece of code!

---

The post-receive hook
---------------------
Assuming you are STILL logged in as the git user:

Move to the __hooks__ folder in the repo  
`cd ~/REPONAME.git/hooks`  

Create a new __post-receive__ hook  
```
echo "#!/bin/bash git --work-tree=/PATH/TO/FINAL/DATA/LOCATION
--git-dir=~/REPONAME.git checkout -f
" > post-receive
```

And make it executable  
`chmod +x post-receive`

Basically, "PATH/TO/FINAL/DATA..." would be the location you ultimately want 
your data to be stored (be it /srv/, /var/www, etc.), so that your web-server 
programs (apache/nginx/etc...) have a set location they know where to find 
your code!  So in this script, we are _receiving post data_ and then moving it 
into the correct _working directory_.

---

Test it!
--------
At this point, assuming all firewall/etc. nuisances have been taken care of, 
you should now have a working git server!

Test it out by cloning it!  
`git clone git@YOURSERVERNAME:~/YOURREPONAME.git`

For me, to verify it was holding my code, I made a commit, deleted my local 
repo, then re-cloned the repo to see if the commit persisted.

**We are now LOCKING DOWN the git user by making the account INACCESSIBLE BY 
SHELL: that way we emphasize that this account is for handling data, and is 
NOT meant to be used regularly (can't be SSH'd into).**

---

Locking down the git user via git-shell
---------------------------------------
Under your normal user-account on the server (anyone BUT git):  
`sudo vim /etc/passwd`

Under here we will look for the line that is similar to:  
`git:x:1234:1234::/home/git:/bin/bash`

And change it to:  
`git:x:1234:1234::/home/git:/git-shell`

This switches the git user's default shell to a more locked down version 
implemented specifically by git for the purpose of being a "true" git user.

---

Now, sit back and enjoy the fact that you now have your own git repository, 
automatically munching code and moving it around to your whim.

