G11DockerWP
===========

A Docker-based solution for cloning existing WordPress sites.  For background, see the post on [Gear 11](http://gear11.com/2014/01/wordpress-docker/)

Thanks to [Eugene Ware](https://github.com/eugeneware/docker-wordpress-nginx)
and [jbfink](https://github.com/jbfink/docker-wordpress)

## Prerequisites
* Docker (requires 64-bit Linux, use VirtualBox for other OSes).  If you are on Windows or OSX and don't already have a VM, then see **Boot2Docker** below
* Existing installer package of your WordPress site created with [Duplicator](http://wordpress.org/plugins/duplicator/)

## Boot2Docker
If you already have a Linux host or VM, skip to **Installation** below.

Boot2Docker ( https://github.com/boot2docker/boot2docker ) makes it easy to run Docker on Windows or OSX if you don't already have a Linux VM that you like to work with.  

####Step 1. Install Boot2Docker
Install Boot2Docker per the instructions.  To verify, open a command prompt and make sure your Boot2Docker VM starts and stops correctly:

````
C:\Users\Me> boot2docker up
Waiting for VM and Docker daemon to start...
..................ooo
Started.
Writing C:\Users\Me\.boot2docker\certs\boot2docker-vm\ca.pem
Writing C:\Users\Me\.boot2docker\certs\boot2docker-vm\cert.pem
Writing C:\Users\Me\.boot2docker\certs\boot2docker-vm\key.pem
Docker client does not run on Windows for now. Please use
    "boot2docker" ssh
to SSH into the VM instead.

C:\Users\Me> boot2docker halt

C:\Users\Me> 
````
Note the Virtual Machine name, which is probably 'boot2docker-vm', for use below.

####Step 2. Configure the Virtual Machine
Map the VM port 80 to your host port 80, so that you can access the VM from your desktop browser, and map the directory of the Duplicator installation files to a share:

````
C:\Users\Me> c:\VirtualBox\VBoxManage modifyvm "boot2docker-vm" --natpf1 "http,tcp,127.0.0.1,80,,
80"
C:\Users\Me> c:\VirtualBox\VBoxManage sharedfolder add "boot2docker-vm" --name "wp-install" --hostpath "C:\path-to-duplicator-files"
````

####Step 3. Start the VM, and log into it via SSH
Execute the following in your command prompt:
````
C:\Users\Me> boot2docker up
...
C:\Users\Me> boot2docker ssh
````

####Step 4: Mount the Duplicator files
Mount the share you created in Step 2 to the directory ~/wp-install on the VM.  This will make your Duplicator files accessible to the VM.  Later on, we'll make this VM directory available to the Docker container.

````
docker@boot2docker:~$ mkdir ~/wp-install
docker@boot2docker:~$ sudo mount -t vboxsf wp-install ~/wp-install
````

Proceed to *Installation* below.

## Installation
On your VM, execute the following
```
$ git clone https://github.com/gear11/G11DockerWP
$ cd G11DockerWP
$ sudo docker build -t="g11-docker-wp" .
```

## Usage

To clone your wordpress site:
```
$ sudo docker run -i -p 80:80 -v (Duplicator files dir):/wp-install -t g11-docker-wp -u (new WP url)
 ```
 For example, to start a new copy of your site on your localhost, with
 your Duplicator package ~/wp-install:
 ```
$ sudo docker run -i -p 80:80 -v ~/wp-install:/wp-install -t g11-docker-wp -u http://127.0.0.1
 ```
 
 Your site should now be cloned and available at http://127.0.0.1 .
 It is a fully independent clone, including:
 * All users and posts
 * All media
 * All links working properly
 * All plugins installed
 
If you'd like to have Docker start services in the background and yield a shell prompt,
start with the -s option:
 ```
$ sudo docker run -i -p 80:80 -v ~/wp-install:/wp-install -t g11-docker-wp -u http://127.0.0.1 -s
 ```
If you snapshot your running image, you can restart it without any options:
 ```
$ sudo docker commit <container id> g11-docker-wp/snap1
...
$ sudo docker run -i -p 80:80 -v ~/wp-install:/wp-install -t g11-docker-wp/snap1
 ```
 It will start much faster since it doesn't have to do any setup.
