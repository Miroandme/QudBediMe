# Git Process

1. [Commit Messages](readme.md#commit-messages)
2. [Branching](readme.md#branching)
3. [Pull Requests](readme.md#pull-requests)

## Commit Messages



* Needs to contain space name and ticket number: `AET-1532`
* Use the present tense ("Add feature" not "Added feature")
* Use the imperative mood ("Move cursor to..." not "Moves cursor to...")
* Limit the first line to *50 characters* or less
* Reference issues and pull requests liberally

Consider starting the commit message with an applicable emoji:
* :art: `:art:`when improving the format/structure of the code
* :racehorse: `:racehorse:` when improving performance
* :memo: `:memo:` when writing docs
* :bug: `:bug:` when fixing a bug
* :fire: `:fire:` when removing code or files
* :green_heart: `:green_heart:` when fixing/improving the CI build
* :white_check_mark: `:white_check_mark:` when adding tests
* :lock: `:lock:` when dealing with security
* :arrow_up: `:arrow_up:` when upgrading dependencies or branches
* :arrow_down: `:arrow_down:` when downgrading dependencies or branches

So it should respect the following formatting: `[:icon:] {verb}{description} [TEAM-NUMBER]`

**Examples:**

* :fire: Remove dead code related to T&C AET-1532
* :memo: Add unit-tests templateCache usage details AET-1533
* :white_check_mark: Add unit-tests for directive referralFooter AET-1534

## Branching

### Naming

`{feature|test|fix}/[ticketId]-[shortDescription]`

* Start with type of branch, feature, test or fix.
* Then write your team name following by the ticket id
* The description should stay short and in camel case

**Examples:**

* `feature/AET-1708-refactorReferralCenter`
* `fix/AET-1709-filterMathRound`

**Benefits**

* The pull request will then show-up in Jira
* The QA team can easily access the code/PR comments directly from within the related ticket
* Gives better visiblity to project managers

![Jira](https://s3.amazonaws.com/uploads.hipchat.com/16534/2237395/oyvRiYi8HbqHt4P/Screenshot%20from%202015-09-23%2015%3A53%3A55.png)

### Model
Sungevity uses the [nvie branching
model](http://nvie.com/posts/a-successful-git-branching-model/) in all major
project repositories.  The key concepts are:

* ```master``` branch is always in a clean, deployable state 
* ```develop``` branch contains all on-going work by developers 
* Feature branches made off of ```develop``` are used to isolate individual pieces of on-going work 
* Feature branches are merged back into ```develop``` when the work is complete
* Releases get their own branches 
* Hotfix branches are made off of ```master``` and merged back into ```master``` and ```develop```

![nvie branching model infographic](http://nvie.com/img/git-model@2x.png "Nvie
Branching Model")

Detailed documentation can be found by reviewing the [article that started it
all](http://nvie.com/posts/a-successful-git-branching-model/)

## Pull Requests

We encourage pull requests to be used whenever a feature is completed and needs
to be merged back into the ```develop``` branch. **A complete feature mean that
it integrates unit-tests**, if you think the feature has good reasons to be
merged without unit-tests please open a discussion with `@Mattias` on the
comments to get a written confirmation.

Make sure to write a description on your pull request before you ask others to
review it. A good description include a link to the related JIRA ticket as well
as a summary key elements that have been changed and why. Please consider
[rebasing the branch](rebase.md) to clean-up the git history

The easiest way to create a pull request is to go to the relevant repository,
click on "Pull Requests" and then "New pull request".  A detailed guide to
using pull requests on github can be found
[here](https://help.github.com/articles/using-pull-requests)

### When is it not appropriate for a pull request?

* Code formatting or comment clean up Most 1 line fixes (use your best judgement)

If you commit directly to `develop` you should still inform the team on Hipchat

### Things to watch for before merging

Be careful not to [break the build](/ci/breaking-the-build.md) when merging
branches that are not subject to CI, especially for compilation errors and unit
test failures. Make sure that:

* Code compiles 
* Unit tests run successfully, Assignee(s) have run and tested the
branch locally 
* You Never force push (git push -f or git push --force) as this is a
  potentially destructive action that will replace remote history with local
 history

Consider rebasing the branch to **clean-up the commit history** and message name.

### Cleanup branches

**Always** be sure to delete the remote branch after QA approving / merging the
pull request. When you don't, the repository becomes cluttered with stale,
inactive branches. 

If someone has more to contribute to the branch and doesn't want it deleted,
tell them that the remote branch will be re-created on their next push.  They
can create another pull request when their future work is completed.

### How to properly merge?

#### Feature branch to develop

Please use the `--no-ff` flag to causes the merge to always create a new commit
object, even if the merge could be performed with a fast-forward. This avoids
losing information about the historical existence of a feature branch and groups
together all commits that together added the feature. Compare:

![no-ff_details](http://nvie.com/img/merge-without-ff@2x.png)

In the latter case, it is impossible to see from the Git history which of the commit
objects together have implemented a feature—you would have to manually read all the
log messages. Reverting a whole feature (i.e. a group of commits), is a true headache
in the latter situation, whereas it is easily done if the --no-ff flag was used.

#### Develop to feature branch

When you want to keep your branch up to date with develop please consider doing
a rebase when you pull the new version: `git pull --rebase develop`. That way
you will not create an entry on the git commit history. 
