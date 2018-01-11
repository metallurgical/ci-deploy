# Introduction

Continuous integration & deployment using Gitlab CI which is hosted on Gitlab.com(self hosted gitlab should be fine). This example using Docker Executor as executor instead of shell executor.
 
## Requirements
1. Gitlab repository hosted on gitlab.com
2. Self hosted gitlab runner
3. Hosted application server(using same server with no 2) which is running on top Ubuntu 16.x LTS
4. Docker installed on server that hosted gitlab runner
5. Key-value paired SSH key to be stored on gitlab.com
6. .gitlab-ci.yml file

## Installation

These steps will go through one by one of each requirements stated above(common/easy step didn't cover).

Before going to start, let's assume we already have project host inside gitlab.

### Install gitlab-runner on server

Before we're going to install gitlab-runner, we need repository's token which is called `specific runner` or `shared runner`. Get the token for specific runner by navigate to **Settings -> CI/CD -> Runners Settings -> find the token on specific runner'section.**

Ssh to server using terminal and add gitlab's official respository:

```
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh | sudo bash
```

Install latest version of gitlab runner:

```
sudo apt-get install gitlab-runner
   or
# install specific version
apt-cache madison gitlab-runner
sudo apt-get install gitlab-runner=10.0.0
```

Once finished install, register a runner. Subsequents prompt from will appear after execute command below:

```
sudo gitlab-runner register
```

Enter URL of gitlab instance(this could be url of gitlab.com or self hosted gitlab):

```
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com )
https://gitlab.com
```

Enter Token(get from gitlab repository's setting):

```
Enter the token you obtained from gitlab to register the Runner:
<token-from-ealier-step>
```

Enter a description for the Runner, you can change this later in gitlab's site:

```
Please enter the gitlab-ci description for this runner
[hostame] my-first-runner
```

Enter the tags associated with the Runner, you can change this later in GitLab's UI:

```
Please enter the gitlab-ci tags for this runner (comma separated):
php, python, another-tag
```

Choose whether the Runner should pick up jobs that do not have tags, you can change this later in GitLab's UI (defaults to false):

```
Whether to run untagged jobs [true/false]:
[false]: true
```

Choose whether to lock the Runner to the current project, you can change this later in GitLab's UI. Useful when the Runner is specific (defaults to true):

```
Whether to lock Runner to current project [true/false]:
[true]: true
```

Choose runner executor to execute gitlab's job later(choose docker as we need docker):

```
Please enter the executor: ssh, docker+machine, docker-ssh+machine, kubernetes, docker, parallels, virtualbox, docker-ssh, shell:
docker
```

If you chose Docker as your executor, you'll be asked for the default image to be used for projects that do not define one in .gitlab-ci.yml:

```
Please enter the Docker image (eg. ruby:2.1):
alpine:latest
```

After completing the step above, the Runner should be started already being ready to be used by your projects! To ensure runner is installed, run :

```
sudo gitlab-runner list
```

If installed correctly, you should see `my-first-runner` on the list.


### Install docker

Docker is available in two editions: Community Edition (CE) and Enterprise Edition (EE). We will using the Community Edition.

To install Docker CE, you need the 64-bit version of one of these Ubuntu versions:

  - Artful 17.10 (Docker CE 17.11 Edge and higher only)
  - Zesty 17.04
  - Xenial 16.04 (LTS)
  - Trusty 14.04 (LTS)
  
Before install, uninstall older version first. Older versions of Docker were called docker or docker-engine. If these are installed, uninstall them:

```
sudo apt-get remove docker docker-engine docker.io
```

It’s OK if apt-get reports that none of these packages are installed.

The contents of `/var/lib/docker/`, including `images, containers, volumes, and networks, are preserved.` The Docker CE package is now called `docker-ce`.

Setup repository:

```
sudo apt-get update
```

Install packages to allow apt to use a repository over HTTPS:

```
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common
```

Add Docker’s official GPG key:

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

Verify that you now have the key with the fingerprint 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88, by searching for the last 8 characters of the fingerprint:

```
sudo apt-key fingerprint 0EBFCD88

pub   4096R/0EBFCD88 2017-02-22
      Key fingerprint = 9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid                  Docker Release (CE deb) <docker@docker.com>
sub   4096R/F273FCD8 2017-02-22
```

Use the following command to set up the stable repository. You always need the stable repository, even if you want to install builds from the edge or test repositories as well. To add the edge or test repository, add the word edge or test (or both) after the word stable in the commands below.

Note: The lsb_release -cs sub-command below returns the name of your Ubuntu distribution, such as xenial. Sometimes, in a distribution like Linux Mint, you might have to change $(lsb_release -cs) to your parent Ubuntu distribution. For example, if you are using Linux Mint Rafaela, you could use trusty.

```
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

Finally install `docker-ce`, Update the apt package index:

```
sudo apt-get update
```

Install the latest version of Docker CE, or go to the next step to install a specific version. Any existing installation of Docker is replaced. If you have multiple Docker repositories enabled, installing or updating without specifying a version in the apt-get install or apt-get update command will always install the highest possible version, which may not be appropriate for your stability needs.

```
sudo apt-get install docker-ce
```

On production systems, you should install a specific version of Docker CE instead of always using the latest. This output is truncated. List the available versions(OPTIONAL)

```
$ apt-cache madison docker-ce

  docker-ce | 17.12.0~ce-0~ubuntu | https://download.docker.com/linux/ubuntu xenial/stable amd64 Packages
  
  # The contents of the list depend upon which repositories are enabled. Choose a specific version to install. The second column is the version string. The third column is the repository name, which indicates which repository the package is from and by extension its stability level. To install a specific version, append the version string to the package name and separate them by an equals sign (=):
  
$ sudo apt-get install docker-ce=<VERSION>
```

Verify that Docker CE is installed correctly by running the hello-world image.

```
sudo docker run hello-world
```

This command downloads a test image and runs it in a container. When the container runs, it prints an informational message and exits.

Docker CE is installed and running. The docker group is created but no users are added to it. You need to use sudo to run Docker commands.


### Generate key value paired ssh

This key are useful for deployment after we test and build our application. Means that, the deployment occured on `gitlab-runner`'s host from gitlab-runner's container.

Open Terminal. And run ssh-keygen command to create key-valued pair :

```
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

This creates a new ssh key, using the provided email as a label.

```
Generating public/private rsa key pair.
```

After running above command, it will ask for renaming your public-private's file name. Otherwise it will created with `id_rsa.pub and id_rsa`'s name.

When you're prompted to "Enter a file in which to save the key," press Enter. This accepts the default file location:

```
Enter a file in which to save the key (/Users/you/.ssh/id_rsa): [Press enter]
```

At the prompt, type a secure passphrase or leave it blank to create without secure passphrase:

```
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type passphrase again]
```


 



