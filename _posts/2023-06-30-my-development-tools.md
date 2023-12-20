---
title: "My development tools"
excerpt: "I'll go through some of the tools I use regularly as a developer"
date: 2023-06-30T08:00:30-04:00
categories:
  - blog
tags:
  - Tooling
---
Hello! I thought it would be good to start by explaining what I do at work. I think it's pretty much the same thing as you readers are doing if you're developers, but I'm usually starting the day with a team meeting to sync all members of the team what needs to be done during the day, discuss sprint scope and if there any issues that need to be dealt with.

After daily sync there might be some other meetings that go more into specific topics, but other than that it's mostly monitoring of the applications, doing some regular maintenance, perform code reviews and writing code that makes my day.

I have divided some of my tools into a couple of sections below:

- [Coding tools](#coding-tools)
- [Cloud tools](#cloud-tools)

## Coding tools

### Version control system/VCS + GitHub

Version control is a must for every team. **Git** is by far the most popular one [stack overflow survey](https://stackoverflow.blog/2023/01/09/beyond-git-the-other-version-control-systems-developers-use/) and the one I prefer. I have previously also used Subversion, which is a centralized VCS (it requires a central server). Git is in my opinion a lot easier to work with and I think it may be hard to find developers that knows svn today. Git is also a distributed VCS (there is no central server, so any developer can work freely locally).

GitHub is the platform where the code for this blog is hosted. It has a copy of this git-repository, but since it's decentralized I could easily move it to another solution if I wanted to do that. Some really nice features of having a web user interface for git is the ability to create Pull requests (PR). These makes it really clear what type of changes have been made in a branch and really helps when doing code reviews (this was not the case when using svn, merge issues where alot more common). A PR is basically a desription of the changes your wish to merge along with a `git diff` presented in a clear way.

#### Windows Terminal

The first thing I start after booting up my computer. This is where everything starts for me, Windows Terminal has options to customize the appearance and also to spin up different shells inside the same window. Usually for me this will be powershell and bash (wsl)

![My Windows Terminal][windows-terminal]

##### Powershell

I don't really do any heavy powershell-scripting a lot, but sometimes I do some light scripting, mostly used for automation purposes. Powershell is just the default shell for me to use on Windows, but I do have some tips & tricks here.

Firstly there's a place where functions and aliases can be stored. In powershell, type `echo $profile` and fire up a text editor to edit this file. If it doesn't exist you can create it by typing `New-Item -Path $profile -Type File -Force` and then add some aliases/functions. An example of an alias is a shortened version for `docker-compose` to `dc`, this will also pass the arguments as well, allowing me to type for example `dc up -d`:

```
# As alias
Set-Alias dc docker-compose

# Same thing as a function
function dc {
    docker-compose $args
}
```

To be able to use this, your powershell execution policy needs to be set to `RemoteSigned`, verify with `Get-ExecutionPolicy` and update the policy if needed with `Set-ExecutionPolicy RemoteSigned`.

Remember to restart your shell after editing this file. 

### WSL 2

When working on a Windows computer there's sometimes a need to run Linux software for me. Mostly this is Linux docker containers, that can be run without a WSL distribution, but using WSL2 as its backend is the preferred way to do this today. Docker Desktop will create its own WSL2 distribution automatically when using the WSL integration (type `wsl -l -v` to see current WSL instances, there will be a couple docker instances there if used with Docker Desktop).

But this is not the only use case for me, as sometimes I need (want) to run Windows-containers simultaneously with Linux-containers, this can be achieved by running Docker Desktop in Windows container mode while also installing a separate docker or podman installation into a dedicated WSL distribution. There's also the possibility to start Linux GUI applications directly from WSL2, it's not something I use regularly but it happens, for example if I need to verify browser behavior on Linux or just feel like using GIMP for Linux :)

And another reason that I sometimes store the source code inside of a WSL distro is that there are performance gains by doing this. This is especially obvious when there are watchers involved that re-compiles/hot-reloads on file changes. I believe this is due to file change notifications not working fully when code is mounted over `/mnt/c/..`, so it's better stored fully inside the distribution.



### Docker/containers/Docker desktop

Used on daily basis for several different reasons. Sometimes I need to run something on Linux and I would then prefer to either start up a predefined image with the software installed on it, or start from a base distribution image and install the tools myself into the container instead of installing it into my host.

Another purpose is to streamline the developer experience and provide an image that has all the necessary tools ready to go, instead of having the developer install and configure everything by themselves (dev containers can be used for this). This also removes a lot of the "Works on my machine" issues.

#### Visual studio code

My default choice of text editor. Previously some years back I used Sublime Text a lot. Some general and handy extensions I can recommend are:

- Dev Containers
- ESLint
- Prettier
- GitLens
- PartialDiff - diff clipboard, files, selections. Super handy
- REST Client - easily construct requests and set headers, payload etc in raw text files. It's like a light-postman.

#### Visual studio

When working in older .NET Framework projects this is a must

#### dotPeek

This is an excellent application that has helped me several times, what it does is to de-compile an existing library to have an idea of what's really happening. This can be very important if there is some issue in a closed library that is not open sourced. 

dotPeek supports many features for navigating the code such as searching, finding usages of methods, going to implementation and more. It has also helped me make decisions when I've had multiple libraries to choose from but I wasn't sure how they would handle large amount of load.

## Cloud tools

### k9s

> K9s provides a terminal UI to interact with your Kubernetes clusters

This is a CLI which speeds up navigating in Kubernetes instead of using `kubectl`. Nothing needs to be installed into the cluster, this is just a client side tool that saves lots of time and is very quick to work with.

Check their git repository at https://github.com/derailed/k9s

### Azure portal

The Azure portal is a place where I monitor resources on daily basis, there are really good tools available to spot failing requests and traffic between services. Combining this with alarms and ability to traverse into traces and exceptions often makes it easy to identify faulty behaviors. Azure services are overall great, but the portal itself has some annoying cumulative layout shifts (CLS) and the user experience is not that great. Hopefully Microsoft will invest some more into the UI parts of the portal :)

### terraform

This is a tool that helps when managing infrastructure in the cloud. For Azure, terraform provides a cleaner syntax than the regular ARM-templates that are otherwise sent to Azure Resource Manager. terraform transforms the cleaner syntax into ARM templates under the hood and will call the necessary API's in Azure, abstracting away this layer for the user. It also keeps track of the state of the resources to make decisions on wether something needs to be recreated, has dependencies or should be removed for example. It does this by creating a plan first, and showing an output what it plans to do. The next step would be to apply the plan.

terraform can be run locally on developer machines or in an automatic pipeline for deployments, like in GitHub Actions.

[windows-terminal]: /assets/images/windows-terminal.jpg
