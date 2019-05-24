---
layout: post
title: "GitLab CI Variables for Coveralls"
date: 2019-05-24
categories: ci gitlab
---
Setting up a demo project on GitLab to use Coveralls was not as straightforward as I expected. Here are a couple issues I ran into.
## .gitlab-ci.yml
I used GitLab's Ruby on Rails template project and the CI wasn't working out of the box. Here's a working version that I trimmed down to just have an rspec step:

{% highlight yaml %}
# This file is a template, and might need editing before it works on your project. 
# Official language image. Look for the different tagged releases at: 
# https://hub.docker.com/r/library/ruby/tags/ 
image: "ruby:2.5.1" 
# Pick zero or more services to be used on all builds. 
# Only needed when using a docker container to run your tests in. 
# Check out: http://docs.gitlab.com/ce/ci/docker/using_docker_images.html#what-is-a-service 
services: 

variables: 

# Cache gems in between builds 
cache:
  paths:
    - vendor/ruby 

# This is a basic example for a gem or script which doesn't use 
# services such as redis or postgres 
before_script:
  - ruby -v # Print out ruby version for debugging
  # Uncomment next line if your rails app needs a JS runtime:
  # - apt-get update -q && apt-get install nodejs -yqq
  - bundle install -j $(nproc) --path vendor # Install dependencies into ./vendor/ruby

rspec:
  script:
    - bundle exec rspec
{% endhighlight %}

## GitLab CI project variables
Coveralls supports multiple CI's but there are not specific instructions for GitLab. [This page](https://docs.coveralls.io/supported-ci-services) (near the bottom) lists the environment variables required for Coveralls to send code coverage stats. The trick is in how you map those variables to their GitLab counterparts. 

Here's what I came up with. In your GitLab project, under Project Settings -> CI/CD Settings -> Variables:

| Key                  | Value               |
|----------------------|---------------------|
| CI_BRANCH            | $CI_COMMIT_REF_NAME |
| CI_BUILD_NUMBER      | $CI_JOB_ID          |
| CI_BUILD_URL         | $CI_JOB_URL         |
| CI_NAME              | gitlab              |
| COVERALLS_REPO_TOKEN | your token          |

## Rails config

Make sure you've added the Coveralls gem ([instructions](https://docs.coveralls.io/ruby-on-rails)) and added this to your spec_helper.rb or rails_helper.rb:

{% highlight ruby %}
require 'coveralls'
Coveralls.wear!('rails')
{% endhighlight %}

Now you should be good to go. Next time you push commits up to GitLab the GitLab CI should trigger a Coveralls stage after running the rspec task.

![pipeline](/assets/img/2019-05-24-coveralls-ruby-gitlab-ci/pipeline.png)

If you have troubles you can consult my [test project](https://gitlab.com/NeilsC/coveralls-ruby-test).
