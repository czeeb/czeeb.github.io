---
title: Faster Test Kitchen Using vagrant-cachier Vagrant Plugin
category: How-To
layout: post
author: Chris Zeeb
excerpt: Reduce Test Kitchen converge with the vagrant-cachier vagrant plugin by caching Chef Omnibus and system packages.
tags:
  - chef
  - vagrant
  - vagrant-cachier
  - test-kitchen
  - virtualbox
date: 2015-12-13 11:36 EST

---

![Converge with already downloaded omnibus](/assets/vagrant-cachier-with-test-kitchen/console_output.png)

[Test Kitchen] is a fantastic aid for writing anything between high quality [Chef] cookbooks to playing with [Chef] for the very first time. When using the [Vagrant] driver some operations can be sped up significantly by utilizing the [vagrant-cachier](https://github.com/fgrehm/vagrant-cachier) plugin.  The vagrant-cachier plugin does an excellent job of caching operation system packages, ruby gems, composer modules, etc. The list if quite long.  You can find the list of package managers supported by going to the [vagrant-cachier documentation](http://fgrehm.viewdocs.io/vagrant-cachier/) and looking under 'Available Buckets'.  Caching these packages means you do not need to download them after they are cached.

* TOC
{:toc}

## Requirements

* [ChefDK](https://downloads.chef.io/chef-dk/)
* [Vagrant](https://www.vagrantup.com/)
* [VirtualBox](https://www.virtualbox.org/)
* A Chef cookbook configured to use [Test Kitchen].  If you don't have one, you can clone the [Neo4j](https://github.com/czeeb/neo4j-cookbook) cookbook.

## Installing vagrant-cachier

Installing the vagrant-cachier plugin is done by simply running `vagrant plugin install vagrant-cachier` from the command line.

## Global Vagrantfile Configuration

Rather configuring [Test Kitchen] to modify the generated Vagrantfile, I set the use of vagrant-cachier globally.  The vagrant-cachier plugin helps increase up times for all vagrant instances sharing the same box this way.  The `~/.vagrant.d/Vagrantfile` file is where you place settings you want global to all [Vagrant] instances you start on that host as your user.

###### ~/.vagrant.d/Vagrantfile
{:.no_toc}

{% highlight ruby %}
Vagrant.configure('2') do |config|
  if Vagrant.has_plugin?('vagrant-cachier')
    config.cache.scope = :box
  end
end
{% endhighlight %}

It is helpful to read up on the precedent order for [Vagrantfile](https://docs.vagrantup.com/v2/vagrantfile/) and the various locations [Vagrant] pulls them from when running `vagrant up` for an instance.

## Telling Test Kitchen to Cache Chef Omnibus

Every time a new kitchen instance is converged the [Chef] omnibus is downloaded.  Rather than try and force the usage of vagrant-cachier on a project we can simply add the necessary settings to the `.kitchen.local.yml` file in the same directory with `.kitchen.yml`.  Just be aware that `.kitchen.local.yml` is usually added to `.gitignore`.  The chef_omnibus_install_options value tells the omnibus installer to save the omnibus download to `/tmp/vagrant-cache/vagrant-omnibus` and check to see if the file already exists and matches it against a checksum.  

###### .kitchen.local.yml
{:.no_toc}

{% highlight yaml %}
---
provisioner:
  chef_omnibus_install_options: '-d /tmp/vagrant-cache/vagrant_omnibus'
{% endhighlight %}

The directory `/tmp/vagrant-cache` is a [synced folder](https://docs.vagrantup.com/v2/synced-folders/) with [Vagrant] in the kitchen instance. Downloading an omnibus package in one kitchen instance makes it available to the rest for future `kitchen create` runs using that same vagrant box!  This is fantastic because the chef omnibus clocks in between 40MB and 45MB, depending on the distro.  If you do a lot of `kitchen destroy ; kitchen converge` or `kitchen test` the time waiting to download that package can really add up.  Especially when you add packages from other package manager on top of that which are now also cached.

[Test Kitchen]:http://kitchen.ci/
[Vagrant]: https://www.vagrantup.com/
[Chef]: https://www.chef.io/
