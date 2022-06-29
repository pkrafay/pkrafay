Jump to
Confluence navigation
Side navigation
Page
This page was in the background for too long and may not have fully loaded. Try to refresh the page

Home

Recent

Spaces

People

Apps
Templates
Create
Search

4



Quickstart


Engineering
Development
Rafay Software Development Practice





Share

Faster development cycle on testbed
Created by Jainam M
Last updated: Jan 07, 20223 min read51 people viewed51 people viewed
 

[Note: the image size have now reduced after the compression of image efforts done by @Thirumal Venkat. Building code base on remote testbed now does not provide any real value. One can setup the codebase in their local mac machine] 

Background:
Today when we engineers develop code, this is the cycle that is followed:

Make changes

Build image

Push image

Then ssh into the testbed

Edit the deployment

Test the changes

Back to 1.

 

Problem:
There are many steps above that is time consuming and which eventually decreases the ease of development. In software development where we repeat the same cycles over and over again the time wastage is also multiplied leading to slower delivery.

Managing pods on terminal can be cumbersome. Instead, we can see it nicely on VSCode directly. Kubernetes extension on VSCode can help you filter logs, open pods, see deployments and delete/restart pods with single clicks.

Solution:
One way to avoid the time wastage is 

Build the repos on testbeds itself. This allows us to save time on step 3.

Automatically pick the latest image when built. This allows us to save time and typing in step 4 and 5.

Tip: Let’s say you are debugging the edgesrv pod. Edit the deployment and under the image attribute put edgesrv:latest.
Next time you build the image, all you have to do is delete the current edgesrv pod and when it comes back again automatically the latest image built is pulled.

Approach:
Setup rcloud repo on your testbed.

There is a script to do this - rcloud-setup.sh , so all you have to do is take a short power nap. But hold on. You’ll need to clone the rcloud repo on the testbed as it asks for authentication. Make sure you do the git clone at $HOME (/home/ubuntu) path 

git clone git@github.com:RafaySystems/rcloud.git

If Git SSH keys are not setup, set them up. Link here

And then run this script on your testbed: It will take about 30 mins to complete - 

rcloud-setup.sh
25 Mar 2021, 01:56 pm



Remote SSH setup:
On your local mac machine add this block to your ~/.ssh/config file


Host vm-core-dev-node-0 129.146.120.186
Hostname 129.146.120.186
StrictHostKeyChecking no
IdentitiesOnly yes
IdentityFile /Users/venkatamuralisaiviharipanathula/Desktop/dev-oci-kp.pem
User ubuntu

-> Replace vm-core-dev-node-0 to whatever the testbed name is - <hostname_given_by_you>
-> Replace the ip address 129.146.120.186 to your testbed ip address.
-> You can get the dev-oci-kp.pem from dev-noc-node’s /opt/rafay/keys path. Copy it to your local mac machine.

-> After copying to your local machine, give the .pem file path in the IdentityFile.


 Once you are done with the above test the connection from your mac terminal by typing - 
ssh <hostname_given_by_you> 
Troubleshoot: If the connection is not going through, it could be because your testbed is off or your config file entry is not proper. Recheck and retry.


VSCode setup

VSCode today is the most used code editor. One of the reasons for that is the concept of extensions which lets you do cool stuff. Think of VSCode as Android and the apps you get on play store as these extensions. 
The extensions that you will have to install is 
1. Remote - SSH (For accessing the codebase from your local Mac machine to your testbed. )
2. Kubernetes (For viewing pods and managing them)
3. Go (For linting and easy code navigation)

Now you can log in to your testbed via VScode and BAM! You are now navigating the code stationed on your testbed. Check out the gif on how to connect - 




 

 

Troubleshoot:
In case you are seeing ssh timeout error on VScode, increase the timeout period to 60 seconds as given here - https://stackoverflow.com/questions/59978826/why-ssh-connection-timed-out-in-vscode#:~:text=open your vscode Command Palette,longer duration than 15 secs.



 

Related pages


Development Workflow - Infra
Development Workflow - Infra
Engineering
Bootstrapping the Rcloud Git repository (Infra Development)
Bootstrapping the Rcloud Git repository (Infra Development)
Engineering
Developer Guidelines
Developer Guidelines
Engineering
Like6 people like this
No labels

Write a comment…

Rcloud-setup.sh
shell · 937 B

#!/bin/sh
cd ~

wget https://github.com/protocolbuffers/protobuf/releases/download/v3.14.0/protoc-3.14.0-linux-x86_64.zip

sudo apt install unzip
#Setup protoc
unzip protoc-3.14.0-linux-x86_64.zip -d $HOME/protoc-3.14/

#Setup go
wget https://golang.org/dl/go1.13.15.linux-amd64.tar.gz 
tar -xzvf go1.13.15.linux-amd64.tar.gz
mv go $HOME/go-1.13


echo 'export GOROOT="$HOME/go-1.13"' >> ~/.bashrc
echo 'export PATH="$HOME/protoc-3.14/bin:$HOME/go-1.13/bin:$PATH"' >> ~/.bashrc
echo 'export GOPATH="$HOME/rcloud"' >> ~/.bashrc

export GOROOT="$HOME/go-1.13"
export PATH="$HOME/protoc-3.14/bin:$HOME/go-1.13/bin:$PATH"
export GOPATH="$HOME/rcloud"

#Setup protoc and go-dep
sudo apt install make git python3-pip go-dep
pip3 install grpcio-tools

#Adding ubuntu user to docker group
sudo groupadd docker
sudo usermod -aG docker ${USER}


echo "Running dep ensure. This will take some time to complete"
cd $HOME/rcloud/src/rep; dep ensure


