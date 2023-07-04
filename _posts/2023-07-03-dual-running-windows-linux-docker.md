---
title: "Running Windows and Linux containers simultaneously"
excerpt: "I'll describe the way I think is the best to run Windows and Linux containers simultaneously"
date: 2023-07-03T08:00:30+02:00
categories:
  - blog
tags:
  - Docker
  - containers
---

This guide is for you guys that have the need to run both Windows and Linux containers simultaneously. There are a couple of ways this can be done, and I'll show the way I prefer and explain why.

So to run Window containers it's mandatory to use a Windows-machine, but on a Windows it's also possible to run Linux-containers which you all know probably, which is the most common thing to do. So in case you have a need to run both of them simultaneously, let's say for example that you have a .NET Framework application that you have containerized. You could theoretically run this one on Mono which is cross platform .NET framework, but it's not the same as a Windows Docker image. This should not stop you from launching Linux-containers for other services where applicable, to keep up with modern technology. Below I'll describe a couple of ways to do it.

## Way 1: From Windows, using experimental --platform parameter
There is a way to pull and run Docker images for Linux while having Docker Desktop set to run Windows containers. To achieve this:
- Set Docker Desktop to run Windows containers
- Enable the experimental features in Docker Desktop and restart Docker daemon

![docker-experimental][docker-experimental]

After this is done, there is a `--platform` parameter added to some commands to specify the platform to use for the image to be pulled and run. So to run a Linux image at this point from your Windows shell you can simply type something like `docker run --platform=linux -it alpine sh` and it should fire up a Linux container. This is quick and convenient but it's still experimental and from my experience it doesn't work very well unfortunately.

## Way 2: Installing docker into WSL 2
By installing Docker into your WSL 2 distribution you can run Docker fully inside of your WSL 2 instead of relying on Docker Desktop to do the magic for you. From my point of view this  approach feels more stable.

#### Install docker
Start by installing docker into your WSL2 distribution, I'm using the Ubuntu distro and to install Docker on it, follow the guide provided for Ubuntu at the docker homepage on [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/). At the time of writing it consists of these steps:

```bash
# Remove any old packages
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

# Set up the repo
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

# Install docker
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Verify installation
sudo docker run hello-world
```

When installing on Ubuntu, you may need to start the service manually here, running `sudo service docker start`. This may also fail at this point because at least my version of Ubuntu  is using `iptables-nft` and Docker needs `iptables-legacy` to work. To change this, run:

```bash
sudo update-alternatives --config iptables
```

Then select **1** to pick `iptables-legacy` and then you can start the service with `sudo service docker start`.

![wsl2-iptables][wsl2-iptables]

#### Activate systemd
You could simply run `sudo service docker start` every time you need to start the service, but `systemd` is now available in WSL 2 and I think it's most convenient to leverage this functionality now.

`systemd` support is available in WSL 2 from version **0.67.6+**. Check your version with `wsl --version`. If your version is lower, or the command doesn't work, you can simply upgrade WSL from Windows Store.

Then in order to activate systemd, edit `/etc/wsl.conf` and add this section:
```bash
[boot]
systemd=true
```

To verify that this works, go back to your Windows shell and restart your distribution. If your distribution in named "Ubuntu" you can type `wsl -t Ubuntu` to terminate it. List your current distros with `wsl --list`. After terminating it, fire up a new WSL 2 shell and run `docker ps` to see the ordinary output.

[docker-experimental]: /assets/images/docker-windows-linux-wsl/wsl2-docker-experimental.png
[wsl2-iptables]: /assets/images/docker-windows-linux-wsl/wsl2-docker-1.png
