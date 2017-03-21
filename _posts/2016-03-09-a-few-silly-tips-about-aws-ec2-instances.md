---
ID: 45
post_title: 'A few (silly) tips about AWS EC2 instances &#8220;backup/migration&#8221;'
author: Gianluca
post_date: 2016-03-09 14:38:30
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/03/09/a-few-silly-tips-about-aws-ec2-instances/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/140744619489/a-few-silly-tips-about-aws-ec2-instances
tumblr_mikamayhem_id:
  - "140744619489"
---
EC2 instances are often considered long lasting resources in charge to deliver the Web applications deployed on them.
This is actually a wrong belief as their nature is instrinsically ephemeral and they should be considered reproducible assets in the context of a much larger infrastructure.

<!--more-->

The infrastructure should be, in general, able to handle the creation of these assets automatically without any human intervention.
Considering the overmentioned wrong belief, it may happens however that you need to manually reproduce one of these assets. In this case these three silly tips may help you to accomplish your goal:

<strong>1-</strong> If you want to move (or clone) an EC2 instance from one availability zone to another you can “simply” create an AMI of the original instance and launch a new EC2 in the desired availability zone from the created AMI.

<strong>2-</strong> If you create an AMI from an EBS backed EC2 instance, i.e. an instance with its root (<code>/</code>) mounted on a persistent EBS volume, the instances created from that AMI will be pretty much identical to the original. In particular it will also be EBS backed with a new dedicated EBS volume.

<strong>3-</strong> Unfortunately right now it seems impossible to perform an <code>rsync</code> between two EC2 instances allocated in two different availability zones.

These tips are actually pretty obvious to the ones having already some experience with AWS but unfortunately no one is born knowing everything! ;)

Cheers!