---
title: Increase File Descriptors For Supervisor Run Processes
category: How-To
layout: post
author: Chris Zeeb
excerpt: Increase available file descriptors for Supervisor managed processes running as non-root user.
tags:
  - supervisor
  - ubuntu
  - linux
  - pam
  - vagrant
---

A pair of colleagues have been working on a website in their spare time.  Recently it has received some additional attention and traffic has increased.  Without getting into details, they have some processes being managed by [Supervisor] which were running out of file descriptors.  The processes run by [Supervisor] are run using a system account.  

It was the first time either of them have had to deal with this, so I was asked to help out.  Their site runs on [Ubuntu] 14.04, so the instructions are specific to that.  The instructions will work fine for other distros such as CentOS. Just be aware that filenames and locations may be different.

If at any step along the way you break the box complete, no problem!  Log out of the vagrant environment and simply run `vagrant destroy -f ; vagrant up` to create a brand new instance.

For those completely unfamiliar with PAM, you may want to read this article on Digital Ocean as a primer: [How to Use PAM to Configure Authentication on a Ubuntu 12.04 VPS](https://www.digitalocean.com/community/tutorials/how-to-use-pam-to-configure-authentication-on-an-ubuntu-12-04-vps)

For those confident enough in their abilities to not test first, you can skip down to [Increasing nofile Limits](#increasing-nofile-limits).

## Instructions
{:.no_toc}

* TOC
{:toc}

## Vagrant time!

When working out the exact steps for a problem like this, I always turn to [Vagrant].  It gives me a nice pristine environment to test in that I can throw away when done.  Usually I'll create them in `~/tmp`.  It doesn't get deleted on reboot which makes cleaning up the VMs later much easier.

1. `mkdir -p ~/tmp/1404`
2. `cd ~/tmp/1404`
3. `vagrant init ubuntu/trusty64`
4. `vagrant ssh`

The last command will log you in to the vagrant environment as the vagrant user.  The vagrant user has access to sudo without a password prompt.

## Create Role Account

It's best for testing that you don't run the [Supervisor] controlled programs as the vagrant user.  It can give a false positive that you modified the ulimits properly.  It is likely that you are using a system user to run you programs rather than root anyway, so we should test with that same setup.

Add the user with `sudo adduser --system testuser`.

## Install supervisor

To install simply run `sudo apt-get install supervisor`.

## Increasing nofile Limits

I use 32000 for the limits for no particular reason.  The purpose was to test setting limits and it seemed like a good round number at the time.  Add the following to `/etc/security/limits.d/custom.conf`:

```
root hard nofile 32000
root soft nofile 32000
```

I prefer to change limits in a `/etc/security/limits.d` file rather than manage it all in `/etc/security/limits.conf`.  It helps separate why limits are set and makes managing them with configuration management a lot easier!  It's much easier to manage individual files for a specific purpose than manage settings like this in a single file.  Don't forget to add a comment to the file to document the reason for setting the limits.

## Update PAM

I really recommend that any time you alter anything with PAM that you log in to another terminal and make sure you have root.  That way if things go horribly awry and you lock yourself out, you can undo the damage.  It's not that big a deal with a vagrant environment.  You'll thank yourself the first time you fat finger a PAM config on a server not so easily repaired.

Add to /etc/pam.d/common-session-noninteractive: `session required pam_limits.so`

## Log Out and Back In

Logging out and back in is only necessary if you are logged in as root.  If you are using sudo to run commands it is not necessary.

## Create Supervisor Program Config

##### /etc/supervisor/conf.d/cat.conf
{:.no_toc}

{% highlight ini %}
[program:cat]
command=cat
user=testuser
{% endhighlight %}

## Restart Supervisor

This will also restart any programs it is managing and grab the new limits.
`sudo service supervisor restart`

## Double Check Limits

Find the pid of a command that supervisor is maintaining.  It should now have the updated file limits

```bash
vagrant@vagrant-ubuntu-trusty-64:~# cat /proc/`pidof cat`/limits
Limit                     Soft Limit           Hard Limit           Units     
Max cpu time              unlimited            unlimited            seconds   
Max file size             unlimited            unlimited            bytes     
Max data size             unlimited            unlimited            bytes     
Max stack size            8388608              unlimited            bytes     
Max core file size        0                    unlimited            bytes     
Max resident set          unlimited            unlimited            bytes     
Max processes             3842                 3842                 processes 
Max open files            32000                32000                files     
Max locked memory         65536                65536                bytes     
Max address space         unlimited            unlimited            bytes     
Max file locks            unlimited            unlimited            locks     
Max pending signals       3842                 3842                 signals   
Max msgqueue size         819200               819200               bytes     
Max nice priority         0                    0                    
Max realtime priority     0                    0                    
Max realtime timeout      unlimited            unlimited            us        
```

[supervisor]: http://supervisord.org/
[ubuntu]: http://www.ubuntu.com/
[vagrant]: https://www.vagrantup.com/