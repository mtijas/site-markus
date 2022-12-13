---
Title: Quick'n'dirty Jenkins Build Executor in local network
Description: Quick'n'dirty Jenkins Build Executor with SSH connection in local network
Template: withsubmenu
---

# Quick'n'dirty Jenkins Build Executor in local network

The need for me is to have a single Jenkins Build Executor (aka worker) running in my home network to mostly learn something Jenkins, but also to have some separation between the controller and the actual worker node(s). Please keep in mind that this guide is written by someone with very little experience in Jenkins, so there probably are better ways to do many things. That's also part of the reason I do not consider [security aspects](https://www.jenkins.io/doc/book/security/) much (it's an informed decision just to get something running in properly firewalled network quickly).

Let's assume we already have a Jenkins Controller running somewhere in our network. If you don't already have a controller running please take a look at their excellent [documentation book on installing Jenkins on Linux](https://www.jenkins.io/doc/book/installing/linux/).

The end result should be something like:
- *controller*: Jenkins Controller running in a Debian Bullseye VM at 192.168.20.21 (jenkins.local.lan:8080) with no build executors locally.
- *jenkinsagent1*: Jenkins Build Executor running in a Debian Bullseye VM at 192.168.20.55 (jenkinsagent1.local.lan).

## The controller

### Prequisities on the controller

The next thing to do is to generate a SSH key for the controller to connect to the jenkinsagent1 worker eventually. See' I'm going towards a system where the controller directly connects to agents (instead of the agent connecting to the controller through [JNLP](https://www.jenkins.io/doc/book/security/services/)).

Ok, let's get started already!

### Generating RSA key pair

First log in to your controller VM with SSH. I'm assuming you are running Jenkins as a specific user `jenkins` there, but log in to the SSH server with your personal account. I'm also assuming you to have some kind of superpowers available (for that I'm using sudo).

Let's first switch to the jenkins user, go to jenkins' home directory and actually check that we are in a correct place:

```bash
markus@jenkins:~$ sudo su jenkins
[sudo] password for markus:
jenkins@jenkins:/home/markus$ cd
jenkins@jenkins:~$ pwd
/var/lib/jenkins
```

We've got `jenkins@<host>` and `/var/lib/jenkins`. That seems to be correct. Let's generate a new RSA key pair that we're going to be using to log in to the agent eventually. Let's just use the default file location and name, and enter a passphrase for the key.

```bash
jenkins@jenkins:~$ ssh-keygen -t rsa -b 4096 -C "jenkins@controller.local.lan"
Generating public/private rsa key pair.
Enter file in which to save the key (/var/lib/jenkins/.ssh/id_rsa):
Created directory '/var/lib/jenkins/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /var/lib/jenkins/.ssh/id_rsa
Your public key has been saved in /var/lib/jenkins/.ssh/id_rsa.pub
```

Seems to have worked as expected. We now have a key pair for use.

Let's then ensure the correct access rights for our newly created `.ssh` directory (if that did not exist already):

```bash
jenkins@jenkins:~$ ls -la
```

That should result in a bunch of rows with .ssh somewhere in between. The correct access rights are `drwx------` (aka 700) for that folder:

```bash
drwx------  2 jenkins jenkins  4096 Nov 13 11:28 .ssh
```

If that's not the case run `$ chmod 700 .ssh`.

For things to work on Jenkins side we also need to add a `.ssh/known_hosts` file and assign correct permissions to it:

```bash
jenkins@jenkins:~$ touch .ssh/known_hosts
jenkins@jenkins:~$ chmod 600 .ssh/known_hosts
```

### Adding the private key to Jenkins

The next thing is to add the private key to Jenkins controller. First view the private key to the terminal with `$ cat .ssh/id_rsa` and copy it to your local clipboard. **Remember that this should be kept secret**.

After that log in to the controller's web user interface and navigate to `Manage Jenkins -> Manage Credentials`. From `Domains (global)` dropdown select `Add credentials`.

To the form that opens add following details:
- **Kind**: SSH Username with private key
- **Scope**: System (Jenkins and nodes only)
- **ID**: *leave blank*
- **Description**: enter "JenkinsController RSA key" (or anything you like)
- **Username**: jenkins
- **Private key**: Enter directly -> add -> paste the private key to the Key input field
- **Passphrase**: enter the passphrase for that key if you provided it during the creation of the key.

Click OK to save the key. The key should now appear to the Global credentials list.

### Getting the public key

The last thing to do in the controller, through the SSH connection, is to view the public key (`$ cat .ssh/id_rsa.pub`) and copy it somewhere to be eventually pasted into the agent. After that you may exit out of the SSH session.

### Configuring Node (agent) in Jenkins

On the web user interface navigate to `Manage Jenkins -> Manage Nodes and Clouds` and select `New Node`.

For name enter something of your choice. I'm going with `JenkinsAgent1`. For Type select `Permanent Agent` and click `Create`. That opens a configuration screen for the node/agent. I'm only going through the fields that need input:

- **Remote root directory**: `/home/jenkins/jenkins`
- **Usage**: *Use this node as much as possible*
- **Launch method**: *Launch agents via SSH*
- **Host**: The IP address of the agent. Mine will be *192.168.20.55*
- **Credentials**: Select the RSA key we just added: *JenkinsController RSA key*
- **Host Key Verification Strategy**: *Known hosts file Verification Strategy*
- **Availability**: *Keep this agent online as much as possible*

That should be enough. Now click `Save`.

The agent will remain in failed state until we finish configuring it, quite obviously.

### Preventing builds from running in controller's built-in node

That's an easy configuration change at `Manage Jenkins -> Manage Nodes and Clouds`. Select `Built-In Node -> Configure`. On the configuration screen set `Number of executors` to zero (0) and click `Save`.

## The agent

Configuring the agent is not much harder than configuring the controller. It's in fact quite a simple process.

I'm assuming you already have a VM running with IP address, SSH server and updates configured, and some kind of firewall running with port 22 opened (both for you and for Jenkins controller).

### Installing required packages

There is actually only one package required: `default-jre`. For my agent I'll also install `docker.io` and `docker-compose` to be able to run some things inside Docker containers with Docker-compose.

```bash
markus@jenkinsagent1:~$ sudo apt update
markus@jenkinsagent1:~$ sudo apt install -y default-jre docker.io docker-compose
```

### Configuration

Let's add a new user for Jenkins:

```bash
markus@jenkinsagent1:~$ sudo adduser jenkins
```

We need to also add the public key for the controller user `jenkins`. So let's switch users and change directory:

```bash
markus@jenkinsagent1:~$ sudo su jenkins
[sudo] password for markus:
jenkins@jenkinsagent1:/home/markus$ cd
jenkins@jenkinsagent1:~$
```

Create necessary files with correct permissions and open the `authorized_keys` file with our preferred text editor:

```bash
jenkins@jenkinsagent1:~$ mkdir .ssh
jenkins@jenkinsagent1:~$ touch .ssh/authorized_keys
jenkins@jenkinsagent1:~$ chmod 700 .ssh
jenkins@jenkinsagent1:~$ chmod 600 .ssh/authorized_keys
jenkins@jenkinsagent1:~$ vim .ssh/authorized_keys
```

And of course add the public key (the one we copied on part "Getting the public key") to a new line there. Save the file and `exit` back to your normal user.

If we are to run Dockerized things here also add user `jenkins` to group `docker`:

```bash
markus@jenkinsagent1:~$ sudo usermod -aG docker jenkins
```

And perhaps reboot the VM (`sudo reboot now`). Remember also to wait for the reboot to complete before continuing.

That should be all.

## Testing things

### Adding jenkinsagent1 to known_hosts on controller

A single vital step still remains: adding jenkinsagent1 to the known_hosts file on the controller. For that let's connect to the controller through SSH, switch users and change to correct directory:

```bash
markus@jenkins:~$ sudo su jenkins
[sudo] password for markus:
jenkins@jenkins:/home/markus$ cd
jenkins@jenkins:~$
```

We are now ready to test the SSH connection from our controller to the agent. Connecting to the agent will prompt for accepting the key from jenkinsagent1. Say yes to that, and it gets automatically added to the `known_hosts` file, and the connection should be made. You might also need to give the passphrase given during the creation of the key.

```bash
jenkins@jenkins:~$ ssh 192.168.20.55
```

If we got connection just `exit` out of it, and close the SSH connection to Controller also.

### Launch the agent!

Let's go back to the Jenkins web user interface. Select the `JenkinsAgent1` from the `Build Executor Status` list, and click `Launch agent`.

If everything went well the agent should now be connected and ready to accept jobs. Jenkins will also tell you that: `Agent successfully connected and online`.

All you need to do now is try to run some jobs. Go to a pipeline and try to `Build now`.
