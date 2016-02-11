---
title: Writing My First Ansible Module
category: General
layout: post
date: 2016-02-10 21:22 EST

author: Chris Zeeb
excerpt: First attempt at creating an Ansible module.  I chose to write it for providing facts from the AWS CloudDeploy API.
tags:
  - ansible
  - clouddeploy
  - module
---

![Header](/assets/my-first-ansible-module/header.png)

I use [Ansible] at work and have become curious to see how the community differs from [Chef].  Ansible is different from Chef in a lot of ways.  Those difference are not good or bad, they just are.  I won't get into what those differences are in this post as I do not want to turn this into a X versus Y entry.

I chose a module that would be small and easy to start with.  There's a recent shift to create *_fact modules.  CloudDeploy is missing a facts module so I started there.  I felt this was a good place to start as it was a read-only module.

An Ansible module can be written in any language. However, to be included in the Ansible distribution it must be written using python and follow some guidelines.

Steps For Creating Ansible Module:

* TOC
{:toc}

## Starting

To start I pulled up three links:

* [Ansible Module Development Documentation](http://docs.ansible.com/ansible/developing_modules.html)
* [AWS CodeDeploy Documentation](http://docs.aws.amazon.com/codedeploy/latest/APIReference/API_Operations.html)
* [Boto3](https://boto3.readthedocs.org/en/latest/)

I also read through several of the existing [Amazon Ansible modules](https://github.com/ansible/ansible-modules-extras/tree/devel/cloud/amazon) as the Ansible documentation suggested.  This gave me an excellent starting point.  There was no point in reinventing the wheel on how to write an Ansible module which queried the AWS API so that facts could be registered.

## Documentation

Documentation for an Ansible module is structured using YAML.  It felt odd and a bit gross to start but overall it's not too bad to work with.  It makes generating the documentation for docs.ansible.com a lot easier on their part, making for a much better overall user experience.

One area I struggled with was proofing my documentation.  Since it is YAML, it's not the easiest to just look at and understand how it is going to look once merged and on the docs.ansible site.  Building the documentation was not a good experience.

The documentation currently just says:

> Put your completed module file into the ‘library’ directory and then run the command: make webdocs. The new ‘modules.html’ file will be built and appear in the ‘docsite/’ directory.

What it means is you either want to copy or symlink your module's .py file not in a `library` file but in `lib/ansible/modules/extras` from the root of the Ansible clone or `lib/ansible/modules/core` if you are writing a core module.  From there you go back to the root of the repository and run `make webdocs`.  Then wait... and wait... and wait... until all the documentation is done building.   For me it takes just over 10 minutes to regenerate all the documentation.  Once done you can then view the documentation locally from `docsite/htmlout`.

![Travis](/assets/my-first-ansible-module/travis.png)

After submitting the pull request initially, Travis returned an error of `ERROR: No RETURN provided`.  It took me a bit of digging to find out that it wasn't complaining that I wasn't returning values somewhere, but that the documentation block for returned values was required and was missing.

## Testing

Ansible modules have no automated testing. Everything must be done by hand.  It is almost solely for this reason that I chose to write a read-only Ansible module as my first.  The idea of writing an open-source module that other people would rely upon without any sort of automated testing really concerned me.

I didn't want to just test against empty API calls, so I set up a quick t2.nano instance and followed the [Setting Up AWS CodeDeploy](http://docs.aws.amazon.com/codedeploy/latest/userguide/getting-started-setup.html) guide. From there I bought some bogus deployments.  It didn't matter if they ran properly or not.  I just wanted some entries for the AWS API to respond with beyond empty arrays.

I had already symlinked my module from my cloned fork of ansible-modules-extras into `lib/ansible/modules/extras`.  That simplified a few things for me.

A sample run of testing the `list_applications` query was run as follows from the ansible root:

{% highlight bash %}
$ ./hacking/test-module -m lib/ansible/modules/extras/cloud/amazon/codedeploy_facts.py -a "query=list_applications"
* including generated source, if any, saving to: /home/czeeb/.ansible_module_generated
* this may offset any line numbers in tracebacks/debuggers!
***********************************
RAW OUTPUT
{"applications": ["TestApplication"], "changed": false, "ResponseMetadata": {"HTTPStatusCode": 200, "RequestId": "b2f6c626-b0be-11e5-bedd-b904f7072eea"}}


***********************************
PARSED OUTPUT
{
    "ResponseMetadata": {
        "HTTPStatusCode": 200, 
        "RequestId": "b2f6c626-b0be-11e5-bedd-b904f7072eea"
    }, 
    "applications": [
        "TestApplication"
    ], 
    "changed": false
}
{% endhighlight %}

Perfect! That's exactly what I wanted to see.  From here it is a matter of testing everything by hand with each of the query types and required fields.  Pretty monotonous really and error prone.  Not to mention any time I want to update the module I need to test everything by hand once again to ensure it is working properly and I didn't break anything.

Multiple variables can be passed in as well, delimited by a space. For example: `-a "query=list_deployment_groups application_name=TestApplication"`.


## Approval Process and Availability

Two current module owners are asked to review the [module guidelines](http://docs.ansible.com/ansible/developing_modules.html#module-checklist).

Once approved (after any requested revisions are made) the module is marked as `shipit` and eventually merged into the master branch.  The Ansible release schedule is roughly every 2 months but adventurous users can use the latest and greatest modules immediately if they use Ansible from a github clone and keep pulling down new revisions.

## Conclusion
{:.no_toc}

I wasn't able to duplicate the output from `ansible-validate-modules` that I saw in my [pull request](https://github.com/ansible/ansible-modules-extras/pull/1445), nor could I find anywhere in the [contributing guidelines](https://github.com/ansible/ansible-modules-extras/blob/devel/CONTRIBUTING.md) that mentioned that the RETURN documentation block was required.  Furthermore, there seems to be some issues getting modules peer reviewed.  The pull request I reference was submitted almost 6 weeks before I published this blog post.  I was hoping that it would be reviewed and accepted long before then.  If you are going to write an ansible module, be prepared to show a lot of patience in getting the module accepted.

[ansible]: http://www.ansible.com
[chef]: https://www.chef.io
