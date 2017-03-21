---
ID: 4
post_title: >
  Elastic IP Address (EIP) and ECS (EC2
  Container Service) cluster, a naive
  solution
author: Gianluca
post_date: 2016-12-07 07:47:00
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/12/07/elastic-ip-address-eip-and-ecs-ec2-container/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/154154623149/elastic-ip-address-eip-and-ecs-ec2-container
tumblr_mikamayhem_id:
  - "154154623149"
---
Recently I had the opportunity to set up another ECS cluster for a Ruby on Rails application that exposes a few API endpoints and a backend to manage some contents, i.e. images, videos and so on.

Considering our previous experience we decided to automate the provisioning of the infrastructure by using Ansible and after a bit we ended up with a few playbooks that allowed us to bring up everything we needed, from the DB to the instances, ELB, task definitions and services.

Everything was working quite well until we were asked to provide a static IP that could be used to access the aforementioned API endpoints.

<!--more-->

While this request may sound legit and easy to satisfy it highlighted a possible flaw in our stack.

The “entry point” of our infrastructure was (and still is!) the ELB which can be referenced with an A record but not with an IP. Moreover it is also not possible to give it an EIP.

After looking around for a solution and trying to solve the problem with an ad-hoc network interface we ended up with a really naive (and temporary) solution: add another “custom layer” of load balancing after the ELB and associate the EIP to this layer.

Considering how we structured our playbooks and how they work we find particularly easy to implement this custom layer as another EC2 instance of our cluster.
The new EC2 instance, just another t2.micro as all other instances, would be responsible to route every request it gets to the other instances in charge of delivering the responses.

Following this rationale we have built a basic HAProxy container and have added a proper task and service to integrate the new layer inside our ECS cluster.

Below you can find, in order, the <code>Dockerfile</code>, the HAProxy configuration (i.e. <code>haproxy.cfg</code>), the Ansible definition of the HAProxy task and its service.
<pre><code>FROM haproxy
MAINTAINER fusillicode &lt;fusillicode@mikamai.com&gt;

ENV BALANCED_ENDPOINT=you-elb-name-180542309.eu-west-1.elb.amazonaws.com:80

RUN mkdir /dev/log

COPY haproxy.cfg /usr/local/etc/haproxy/haproxy.cfg
</code></pre>
<pre><code>global
  log /dev/log    local0 info alert
  log /dev/log    local1 notice alert
  daemon

defaults
  log     global
  mode    http
  option  httplog
  option  dontlognull
  timeout connect 5000
  timeout client  50000
  timeout server  50000

frontend localnodes
  bind *:80
  mode http
  default_backend node

backend node
  mode http
  balance roundrobin
  option forwardfor
  http-request set-header X-Forwarded-Port %[dst_port]
  server web0 ${BALANCED_ENDPOINT}
</code></pre>
<pre><code>- name: Provision Cluster Task Definition (HAProxy)
  aws_ecs_taskdefinition:
    containers:
    - name: proxy
      cpu: 10
      essential: true
      image: "{{ repository }}/your-app-haproxy:latest"
      memory: 500
      portMappings:
      - containerPort: 80
        hostPort: 80
    state: update
    family: "{{ project_id }}-haproxy-{{ env }}"
  register: taskdefinition_haproxy
</code></pre>
<pre><code>- name: Provision Cluster Service HAProxy
  ecs_service:
    name: haproxy
    state: present
    desired_count: 1
    task_definition: "{{ taskdefinition_haproxy.taskdefinition.family }}"
    cluster: "{{ cluster.cluster.clusterName }}"
</code></pre>
By following this naive solution, the next logic thing that we would have to do was to implement the automatic assignment of the EIP to the desired instance.

However I think that this kind of “service discovery” would be quite complex and error prone so I strongly recommend to investigate other possible ways to associate an EIP to an ECS cluster. Moreover the custom layer we introduced represents a single point of failure regarding the exposed API endpoints.

Despite the fact that we have a proper ECS service that handles the HAProxy functionality it does not handles the automatic rebind of the EIP. If, for any reason, the HAProxy container or the EC2 instance in which it runs becomes unavailable the ECS service would provide another instance without knowing nothing about the previous EIP.

Considering these problems, a possible alternative could have been, for example, to change our ELB to an Application Load Balancer and try to handle the new custom redirection layer in a more resilient way. Another one would have been to create a proper network interface and integrate it inside your VPC. And if you don’t have a VPC…well, you should have one ;)

Beside the naiveness of the approach described in this article I hope that it can be useful to the ones facing the same situation I had.

Cheers!