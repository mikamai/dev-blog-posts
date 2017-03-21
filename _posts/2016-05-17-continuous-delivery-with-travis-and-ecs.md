---
ID: 30
post_title: Continuous delivery with Travis and ECS
author: Nicola Racco
post_date: 2016-05-17 12:49:22
post_excerpt: ""
layout: post
permalink: >
  https://dev.mikamai.com/2016/05/17/continuous-delivery-with-travis-and-ecs/
published: true
tumblr_mikamayhem_permalink:
  - >
    http://dev.mikamai.com/post/144501132499/continuous-delivery-with-travis-and-ecs
tumblr_mikamayhem_id:
  - "144501132499"
---
ECS is a good product. Sadly it’s authored by the same UX designer that authored all other AWS products, so a lot of people couldn’t even succeed in starting a simple hello world container.

Some months ago `@fusillicode` wrote a two-part tutorial on how to dockerize and deploy on ECS a WordPress app (you can find them here: [part 1](https://dev.mikamai.com/2016/01/21/ecs-and-kiss-dockerization-of-wordpress/) and [part 2](https://dev.mikamai.com/2016/02/16/ecs-and-kiss-dockerization-of-wordpress-part-2/)). Of course, given we're talking of docker, the technology you're using is not so important.

What’s missing in those posts is how to do a painless deploy.
<!--more-->

Normally you would:

- push on your docker repo the new image
- open your task definition 
![ECS Task Definition](https://dev.mikamai.com/wp-content/uploads/2016/05/ecs_task_dash-640x376.png)
- update it without changing anything. This will create a new revision for it
- open the app service in the ECS cluster
![ECS Service](https://dev.mikamai.com/wp-content/uploads/2016/05/docker-private-registry-create-service.png)
- update the task definition revision

And ECS would take care of launching a new EC2 instance, starting a container with the updated image, and stopping the old container.

So, this is great news, ECS offers a zero-downtime deployment out of the box.

Nonetheless, a deploy can be tricky as you can see by the above checklist. A good way to avoid all of this is to have a continuous delivery system.

For this post I used a Rails app that is being tested on Travis CI. What I want to obtain is that when I push something in the master branch, if Travis is green, a deploy is automatically made in production.

I already have a running ECS Cluster, with a running service, and Travis is already running.

### Continuous docker building & pushing

First thing I need is to let Travis build and push docker images when specs are green. Add the following to your `.travis.yml`:

```yaml
sudo: required
services:
  - docker
after_success:
  - bin/docker_push.sh
```

We’re telling Travis that:

- we need a privileged container
- we need to have access to the docker exec
- it has to call the script `bin/docker_push` if tests are green

The content of `docker_push.sh` is the following:

```bash
#! /bin/bash
# Push only if it&#039;s not a pull request
if [ -z &quot;$TRAVIS_PULL_REQUEST&quot; ] || [ &quot;$TRAVIS_PULL_REQUEST&quot; == &quot;false&quot; ]; then
  # Push only if we&#039;re testing the master branch
  if [ &quot;$TRAVIS_BRANCH&quot; == &quot;master&quot; ]; then
  
    # This is needed to login on AWS and push the image on ECR
    # Change it accordingly to your docker repo
    pip install --user awscli
    export PATH=$PATH:$HOME/.local/bin
    eval $(aws ecr get-login --region $AWS_DEFAULT_REGION)
    
    # Build and push
    docker build -t $IMAGE_NAME .
    echo &quot;Pushing $IMAGE_NAME:latest&quot;
    docker tag $IMAGE_NAME:latest &quot;$REMOTE_IMAGE_URL:latest&quot;
    docker push &quot;$REMOTE_IMAGE_URL:latest&quot;
    echo &quot;Pushed $IMAGE_NAME:latest&quot;
  else
    echo &quot;Skipping deploy because branch is not &#039;master&#039;&quot;
  fi
else
  echo &quot;Skipping deploy because it&#039;s a pull request&quot;
fi
```

In order to have this script work as expected you need also to add the following ENV variables on Travis:

- `AWS_DEFAULT_REGION`: default region for your AWS account (e.g. eu-west-1)
- `AWS_ACCESS_KEY_ID`: Your AWS access key
- `AWS_SECRET_ACCESS_KEY`: Your AWS secret access key
- `IMAGE_NAME`: Docker image name for your app (e.g. foo)
- `REMOTE_IMAGE_URL`: Docker repo image url (e.g. your.repo.url/foo)

Do a push on your master branch, and if everything is ok you will see Travis pushing your image.

### Continuous deploy

In order to trigger the deploy on ECS we’ll use the [ecs-deploy](https://github.com/silinternational/ecs-deploy/blob/master/ecs-deploy) script.

I downloaded it in my bin folder. In addition, I added a new script, `bin/ecs_deploy.sh`:

```bash
#! /bin/bash
# Deploy only if it&#039;s not a pull request
if [ -z &quot;$TRAVIS_PULL_REQUEST&quot; ] || [ &quot;$TRAVIS_PULL_REQUEST&quot; == &quot;false&quot; ]; then
  # Deploy only if we&#039;re testing the master branch
  if [ &quot;$TRAVIS_BRANCH&quot; == &quot;master&quot; ]; then
    echo &quot;Deploying $TRAVIS_BRANCH on $TASK_DEFINITION&quot;
    ./bin/ecs-deploy -c $TASK_DEFINITION -n $SERVICE -i $REMOTE_IMAGE_URL:$TRAVIS_BRANCH
  else
    echo &quot;Skipping deploy because it&#039;s not an allowed branch&quot;
  fi
else
  echo &quot;Skipping deploy because it&#039;s a PR&quot;
fi
```

We need two more ENV variables on Travis:

- `TASK_DEFINITION`: the name of your ECS task definition
- `SERVICE`: the name of your ECS cluster service

Edit the `after_success` block of your `.travis.yml` to be like the following:

```yaml
after_success:
  - bin/docker_push.sh
  - bin/ecs_deploy.sh
```

Push it, and voilà! Deploy in progress.