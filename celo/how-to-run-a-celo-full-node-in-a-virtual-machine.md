# Introduction

In this tutorial, we will set up a Virtual Machines and install the necessary software to create a full node, then run a light client to monitor the network.  
Full nodes are a major part of the Celo blockchain. A full node is a constantly running program that will help to write, and validate new changes in the network. Multiple full nodes work together, validating the blockchain in a decentralized way.

Please watch the videos linked below and follow along with the written tutorial!

# Prerequisites

* Have a basic understanding of Virtual Machines \(VM\), and the Linux terminal.
* Install the Valora app on iOS or Android: [https://valoraapp.com/](https://valoraapp.com/)
* Download the most recent Debian 10.x 64-bit ISO: [https://www.debian.org/](https://www.debian.org/)
* Download VirtualBox, the Oracle VM host: [https://www.virtualbox.org/](https://www.virtualbox.org/)
* Download Signal for iOS or Android: [https://www.signal.org/](https://www.signal.org/)

# Creating the VM

**Video 1 -** Creating the Virtual Machine in VirtualBox

**Click the image to access the video :**

[![SC2 Video](https://img.youtube.com/vi/Cdqwzf-zfug/0.jpg)](http://www.youtube.com/watch?v=Cdqwzf-zfug)

* Click on "Create New" in VirtualBox
* Select a dynamically allocated virtual hard disk
* Start the Virtual Machine
* Create a new user 
* Select "Use entire disk", then write the changes to disk
* Select the checkboxes: Debian desktop environment, SSH server, Standard system utilities 
* Install the Grub bootloader onto the Virtual HARDDISK

The installation of Debian is complete and the VM is now ready.

**Video 2, Building the full node**

[![SC2 Video](https://img.youtube.com/vi/l8qAISLJZq8/0.jpg)](http://www.youtube.com/watch?v=l8qAISLJZq8)

### Gain sudo on the VM

On Linux, the command `sudo` stands for "substitute user do".   
Once a user account is added to the `sudo` group, it will be able to act with administrator or "root" privileges on the VM.

The following commands are entered into the VM terminal:

`su -`

* You will be required to enter the root password of the VM.

`usermod -aG sudo YOURUSERNAME`

* YOURUSERNAME needs to be replaced with the currently logged in users' account name. By default this should be displayed in the terminal prompt, but you can use the command `who` to make sure which account you are currently logged in with. 
* Restart the VM

`id`

* Find \(sudo\) in the group list:

![image](https://user-images.githubusercontent.com/80616939/120870817-2f38a480-c557-11eb-9fd0-5be9e710af8d.png)

## Install Docker & utilities

Curl will be used for inserting GitHub scripts into the terminal.

Docker is a service that will be running the light client.

Vim is a tool used for editing files within the terminal. [Click here](https://www.linux.com/training-tutorials/vim-101-beginners-guide-vim/) to read a beginners guide to Vim.

`sudo apt update && sudo apt upgrade`

`sudo apt install docker.io`

`sudo apt install curl`

`sudo apt install vim`

## Enable bi-directional copy/paste & and improve display resolution

Run an external program called VBoxLinuxAdditions, which enables copy/paste between the VM and the host machine.

Cursor over devices in the top left corner, and select: Insert Guest Additions CD Image 

![image](https://user-images.githubusercontent.com/80616939/118760448-0ebadb80-b830-11eb-88fa-51e672a41d83.png)

`cd /media/cdrom`

`sudo sh VBoxLinuxAdditions.run`

* Restart the VM 
* Select Bidirectional copy/paste under Devices &gt; Shared Clipboard

Linux headers provide many vital functions without installing unnecessary files, while VBoxLinuxAdditions provides the dynamic screen resolution.

``sudo apt install build-essential linux-headers-`uname -r``` 

* Restart the VM 
* Select "Adjust window size" under View on the menubar.

## Install nvm and the Celo CLI

This is the install script for [nvm](https://github.com/nvm-sh/nvm), the Node Version Manager. Remember to source files after adding them.

`curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.37.2/install.sh | bash`

`source ~/.bashrc`

Install/use node version 10, then install the Celo CLI onto this client.

`nvm install 10`

`nvm use 10`

`npm install -g @celo/celocli` : The `-g` flag here means we will [install the package globally](https://docs.npmjs.com/downloading-and-installing-packages-globally).

## Join the Docker group

Add your user account to the Docker group.

`sudo usermod -aG docker $YOURUSERNAME`

* Restart the VM

`id`

* Find \(docker\) in the list because 

Docker is required for running the light client.

* Close this terminal 

## Use curl to fetch the script

To learn about `.env` files check out this [guide](https://docs.figment.io/network-documentation/extra-guides/dotenv-and-.env).

We are using `curl` to fetch the information stored on [Gist](https://docs.github.com/en/github/writing-on-github/editing-and-sharing-content-with-gists/creating-gists), a free site that GitHub provides for sharing code and other documents.

This `.env` file is for storing and accessing environment variables. It will hold the public keys. 

```text
curl -o celo.env https://gist.githubusercontent.com/alchemydc/ce712f6f3caa7ec79f15f930ed5904ed/raw/385c65b1d3f760854258bfd6dd8cbd135710b78f/celo.env 

source celo.env
```

Here are the contents of this file, if you'd like to create `celo.env` yourself :

```text
export CELO_IMAGE=us.gcr.io/celo-org/geth:mainnet
export CELO_ACCOUNT_ADDRESS="YOUR_ACCOUNT_ADDRESS"
```

This file runs the light client, and contains information about the node in the form of the runtime parameters. This shell script requires Docker to be installed in order to work successfully because it contains the commands `docker pull` and `docker run` . To place the script on the filesystem of the VM, run the following commands

```text
curl -o start_celo.sh https://gist.githubusercontent.com/alchemydc/e28945f5059acd70969b39a50fd0f80a/raw/0d15cceb89ea86ca46df94441c06ecd88a4e6635/start_celo.sh 
```

![image](https://user-images.githubusercontent.com/80616939/120844824-33e86300-c52d-11eb-8b1f-db5e66ec76bb.png)

Move these files into a new directory named `celo-data-dir` by running these commands in the terminal:

```text
mkdir celo-data-dir && cd celo-data-dir

mv ../celo.env . 
mv ../start_celo.sh .

source celo.env && source start_celo.sh
```

The command `mkdir` is for making a new directory, `cd` is for changing the current directory, and `mv` is for moving files. `source` will read and execute the contents of the file.

The docker container should now be running.

## Starting the node

**Run this command from inside celo-data-dir :**

`docker run -v $PWD:/root/.celo --rm -it $CELO_IMAGE account new`

Copy the public address that is provided with your new node account :

![image](https://user-images.githubusercontent.com/80616939/122334208-96ab0880-cef6-11eb-9204-d1279da300a8.png)

Next, run `vim celo.env`

![image](https://user-images.githubusercontent.com/80616939/118758843-044b1280-b82d-11eb-88df-e83e5fb813f4.png)

* In vim press "i" for insert mode.
* Remove YOUR\_ACCOUNT\_ADDRESS with with the backspace or delete key.
* Paste in the public address by pressing "shift+insert".
* Exit vim by pressing ":", then typing "wq", then press return \(sometimes called enter\).
* Because we have changed the contents, remember to run `source celo.env`

## Running the light client

`chmod u+x ./start_celo.sh`, Gives the user permission to execute the script.

`./start_celo.sh`, Starts the light client.

`cat ./start_celo.sh`, Displays node information.

`docker stop geth`, Stops the light client.

* Use 2 terminal tabs. One for starting the light client, and the second for all other commands.    

`./start_celo.sh`

`celocli node:synced`

* If true, the node is connected.   

`celocli account:balance NODE_ADDRESS`

`docker stop geth`

![image](https://user-images.githubusercontent.com/80616939/118338503-8eb11080-b4d3-11eb-99a3-11417cf79b32.png)

## Sending Celo to and from the node

* Send both CUSD, and CELO native token to your node address.
* Have 2 terminal windows open. One is for running the light client, and the other for entering commands.
* Make sure both terminals are in the `celo-data-dir`.

`cat ./start_celo.sh`

`docker exec -it geth geth attach`

`exit`

`celocli account:unlock $PUBLIC_ADDRESS`

* Enter password 

`source celo.env`

`celocli transfer:dollars --from $NODE_ADDRESS --to $PHONE_ADDRESS --value=1e16`

Understanding this unit of measurement: 1e16 = 0.01 CUSD, 1e15 = 0.1 CUSD, 1e14 = 1.0 CUSD.

![image](https://user-images.githubusercontent.com/80616939/118338915-9c1aca80-b4d4-11eb-87b6-7970949923aa.png)

# Conclusion

Congratulations, the full node and light client should now be operational!  
Keep in mind this is not the only way to set up a full node. This node is now under your control and part of the Celo blockchain, so remember to store all related keys and passwords safely to avoid losing any money. Please experiment with this setup and personalize it. The `start_celo.sh` and `celo.env` files can be altered to suit your needs. 

# About the Author

This tutorial was put together by [Aidan Dedecker](https://github.com/Aidandedecker/), a bright young member of the Learn community.