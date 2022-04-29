---
layout: post
title: "Use Git Hooks to Automate Your Development Workflow"
date: 2018-06-09
tags: git terminal ci
---
There's something that happens in my workflow more often than I'd like to admit. I'm focused on a coding task, fixing a bug or finishing up a feature. Certain that I'm done, I `git push` and move on to something else. Meanwhile, a CI pipeline is running, analyzing my latest changes and finding them lacking.

{% highlight bash %}
CI build failed!

> rubocop
Inspecting 491 files
...................................................C.......................................................................................................................................................................................................................................................................................................................................................................................................................................................

Offenses:

app/models/some_model.rb:19:1: C: Style/EmptyLinesAroundModuleBody: Extra empty line detected at module body beginning.

491 files inspected, 1 offense detected

{% endhighlight %}

And there goes my focus and momentum, all due to an extra empty line, and forgetting to run rubocop before pushing my code!

## Enter Git Hooks! ##

Hooks are extension points built in to git that allow us to add our own functionality in the workflow. Peering into the `.git/hooks` directory of a git repo will show us some example scripts for the available hooks:

{% highlight bash %}
> tree .git/hooks
hooks
├── applypatch-msg.sample
├── commit-msg.sample
├── post-update.sample
├── pre-applypatch.sample
├── pre-commit.sample
├── prepare-commit-msg.sample
├── pre-push.sample
├── pre-rebase.sample
└── update.sample
{% endhighlight %}

To prevent myself from pushing bad code up to my git/CI server, I'm going to create a pre-push script that will run my code analysis tool (rubocop) and abort the push on failure. This script will go in `.git/hooks/pre-push`:

{% highlight bash %}
#!/bin/bash
#
# Runs rubocop before pushing to remote. Aborts the push if rubocop fails.

rubocop -ESD

if [[ $? = 0 ]]
then
  exit 0
else
  echo >&2 "Rubocop failed, not pushing"
  exit 1
fi
{% endhighlight %}

Now set the script as executable:

{% highlight bash %}
> chmod +x .git/hooks/pre-push
{% endhighlight %}

Git will run the script before pushing changes. A non-zero exit code indicates failure and will abort the push.

Enjoy!
