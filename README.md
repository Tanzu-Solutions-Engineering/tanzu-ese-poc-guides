# VMware Tanzu Emerging Solutions Engineering - PoC Guides

[![Twitter
Follow](https://img.shields.io/twitter/follow/vmw_rguske?style=social)](https://twitter.com/vmw_rguske)
[![Twitter
Follow](https://img.shields.io/twitter/follow/Alec1823?style=social)](https://twitter.com/Alec1823)

## Table of Contents

- [VMware Tanzu Emerging Solutions Engineering - PoC Guides](#vmware-tanzu-emerging-solutions-engineering---poc-guides)
  - [Table of Contents](#table-of-contents)
  - [The Intention](#the-intention)
  - [Tanzu Solutions](#tanzu-solutions)
    - [Tanzu Kubernetes Runtimes](#tanzu-kubernetes-runtimes)
    - [Tanzu Packages](#tanzu-packages)
    - [Tanzu Kubeapps](#tanzu-kubeapps)
  - [Additional](#additional)
    - [CSI SMB Provisioner](#csi-smb-provisioner)
  - [Contribution Guidelines](#contribution-guidelines)
    - [Preperations](#preperations)
    - [Contributions](#contributions)

## The Intention

We are seeing a huge increase in demand from clients who are transforming their software delivery capability and building cloud-native applications to drive their digital transformation.

VMware Tanzu represents our growing portfolio of solutions to help you build, run and manage modern apps.

The **VMware Tanzu Emerging Solutions Engineering** Team supports both, customers and VMware internal teams, in successfully performing proof-of-technologies (or proof-of-concepts) on VMware Tanzu solutions.

This repository supports PoT(C)'s by providing helpful resources, hints, tips and tricks. It contains several subsections for a variety of the Tanzu solutions.

## Tanzu Solutions

### Tanzu Kubernetes Runtimes

* [Tanzu Kubernetes Grid Service (TKGs)](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/tkgs.md)
* [Tanzu Kubernetes Grid Multi-Cloud (TKGm)](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/tkgm.md)
* [Tanzu Kubernetes Grid Integrated Edition (TKGi)](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/tkgi.md)

### Tanzu Packages

* [Tanzu Packages](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/packages.md)

### Tanzu Kubeapps

* [Tanzu Kubeapps](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/kubeapps.md)

## Additional

### CSI SMB Provisioner

* [Kubernetes CSI SMB Provisioner](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/blob/main/csi-smb-provisioner.md)

## Contribution Guidelines

1st, **thanks a lot for your willingness to contribute** :thumbsup:

In order to have multiple actors working on this helpful resource, please follow our contribution guidelines.

### Preperations

* Don't commit changes directly to `origin`! 
* [Fork](https://docs.github.com/en/get-started/quickstart/fork-a-repo) the repository to your Github account and go ahead from there:

  * `clone` the fork locally: `git clone git@github.com:rguske/tanzu-ese-poc-guides.git`
    * Change into the cloned dir `cd tanzu-ese-poc-guides`
  * Rename your fork from e.g. `origin` to `fork`: `git remote rename origin fork`
  * Add the `origin`al repository as well, to e.g. `fetch` and `pull` changes from `origin` (changes caused by e.g. contributions from others): `git remote add origin https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides.git`
  * Execute `git remote -vvv` to see both added repos

You should see a similar output like this:

```bash
git remote -vvv
fork    git@github.com:rguske/tanzu-ese-poc-guides.git (fetch)
fork    git@github.com:rguske/tanzu-ese-poc-guides.git (push)
origin  https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides.git (fetch)
origin  https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides.git (push)
```

### Contributions

* Please always use Github Issues to document your work. Example: [Issue-2](https://github.com/Tanzu-Solutions-Engineering/tanzu-ese-poc-guides/issues/2)
  * Assign contributers to the issue
  * Use lables like e.g. `wip` or `documentation`
* Name branches after the appropriate issue: `git checkout -b issue-2`
  * Start working within the branch (don't forget to checkout if you are working on multiple branches!)

* If you finished working on an issue and you'd like to push it to your fork, to ultimately open a Pull Request at `origin` follow the example:
  * Stage all changes which you'd like to `commit` and `push`
* Follow Git commit [best practices](https://cbea.ms/git-commit/)
* A commit message can address an issue directly. Make use of it! `git commit -s - m "your commit message" -m "Closes: #2"`
  * By appending `-m "Closes: #2"` to your commit, it'll automatically close it finally

> Sign your commits! The e-mail address used to sign must match the e-mail address of the Git author. If you set your `user.name` and `user.email` git config values, you can sign your commit automatically with `git commit -s`.

I highly recommend reading this awesome [BLOG POST on Git](https://www.mgasch.com/2021/05/git-basics/) by [Michael Gasch](https://twitter.com/embano1) when you are in trouble.

Finally, merge your branch if you've finished working on it. `git merge issue-2`
