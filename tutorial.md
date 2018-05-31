# A tutorial on getting XNAT’s container service up and running

This tutorial assumes you have a working XNAT instance. You must have administrator priveleges on your XNAT instance to use the Container Service.  If you are not sure how to set up an XNAT instance and would like a quick way to get up and running.  

## Installing the Container Service plugin.

Download the .jar file of the “Latest Release” on this page to <xnat-home>/plugins and restart the tomcat server

In more detail: 
..*Nagivate to the plugins directory under your XNAT home directory.  

If you are using the Vagrant VM standard setup described above, this is how you get to your home directory on the virtual machine (and give yourself write permission):
```....cd <xnat-installation-directory>xnat-vagrant/configs/xnat-release
   ....vagrant ssh
   ....sudo -i
```
Now you should have a `root@xnat-release` command prompt.  Finally

`cd /data/xnat/home/plugins`

(or cd to wherever <xnat-home>/plugins is on your XNAT instance.)

..*Download the "Latest Release" from [this page](https://github.com/NrgXnat/container-service/releases).  As of this writing (5/30/18) the link to the latest release was [here](https://github.com/NrgXnat/container-service/releases/download/1.5.1/containers-1.5.1-fat.jar).
In your XNAT home directory, you can download it with:
```wget https://github.com/NrgXnat/container-service/releases/download/1.5.1/containers-1.5.1-fat.jar``` (or, if you are reading this IN THE FUTURE, the new "latest release" that has superceded 1.5.1).

..*Restart tomcat
`service tomcat7 restart`

Now, under Administer -> Plugin Settings in the XNAT menu you should see the Container Service pages. 




