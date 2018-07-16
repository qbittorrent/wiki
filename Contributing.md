# Simple Git workflow for contributing to qBittorrent. #

If you want to contribute code to the project, you are going to need your own copy of the qBittorrent Git repository. Create (if you don't have it yet) a GitHub account and fork qBittorrent repository into your account. Then clone the fork locally.

## Creating a pull request ## 

To make it easier for you, create a git branch for your new code from the very beginning (if you didn't, create a branch right now and cherry-pick your commits into the new branch).

Then make your changes, commit them to your feature branch (let's call it `my-feature`). You may split your changes on several commits if that makes sense. When you are done, push your changes into your fork (this remote repository will be named `origin` in your local repository by default). So, push your changes via `git push`.
Now go to [qBittorrent GitHub page](https://github.com/qbittorrent/qBittorrent) and create your pull request.

## Updating your pull request ##

During life time of your PR you might be asked to make changes to it or qBittorrent code base might be updated and your changes will be in conflict with its code base and you will be asked to "rebase your PR".

### Making changes to your pull request ###
If you are asked to make corrections to your changes, you may either amend your commits (works best with small pull requests) or add additional fixup commits to `my-feature` branch. When your PR is large, it will be simpler for reviewers to get new changes in new commits. For that, create new commits using [`--fixup=` or `--squash=` git parameters](https://robots.thoughtbot.com/autosquashing-git-commits). When your pull request is accepted, you will need to squash all these commits and `--fixup` & `--squash` parameters will make it much simpler for you. Note, you can create fixup commits to fixup commits, and that will allow git to arrange all of them properly when squashing.

### Rebasing your pull request ###

First of all, add new remote to your local repository, pointing to qBittorrent repository: `git remote add upstream  https://github.com/qbittorrent/qBittorrent.git`

1. Checkout your master branch: `git checkout master`.
1. Fetch changes from upstream: `git fetch upstream`.
1. Merge upstream master branch into your local master branch: `git merge upstream/master`. This has to be a fast-forward merge.
1. Checkout your feature branch: `git checkout my-feature`.
1. Rebase your changes on top of your master branch: `git rebase master`. If you do this because of the conflicts, you will be solving them right now correcting source files and amending your commits.
1. Push your new commits to your fork: `git push --force`. You need `--force` because you want to rewrite history of the remote branch with new commits.