# MHN-setup
MHN setup configured with Splunk forwarders 

## Overview
Setting up a network of honeypots to better understand the types of threats servers might face and also to carry out an analysis on the types of attacks, their geolocation and the impact they might have on an enterprise network. The traffic logged by the Modern Honey Network (MHN) will then be forwarded to Splunk for further analyses.  

![alt text](https://github.com/adepoju/MHN-setup/blob/master/splunk_dashboard.png "Splunk Dashboard")

## System requirement 
The MHN can be installed on various linux distros. I will be using Ubuntu 16.04. X64 with 15GB ram, 4 CPUs, and 80GB storage. This is more than the hardware requirements [recommended](https://github.com/threatstream/mhn/wiki/Hardware-Recommendations) but i am making use of GCP so why not :) .

## Download the MHN server
We need to download and install the MHN server where the honeypots will report to. Once we are done setting up the server, we can login to it via ssh. We need to install git (if not already installed) to clone the mhn server scripts. 

> * sudo apt-get install git -y

Then we clone the git repo:
> * cd /opt/
> * sudo git clone https://github.com/threatstream/mhn.git

Create a new user and add to sudo:
> * sudo adduser ‘user’
> * sudo gpasswd -a ‘user’ sudo 

### Install the MHN Server
Then we need to run the script to install the server.
cd mhn/
sudo ./install.sh
At this point, a bunch of scripts will run for a couple of minutes. We will then be prompted for a couple of configurations:

> MHN Configuration  
> python generateconfig.py  
> Do you wish to run in Debug mode?: y/n #select "n"  
> Superuser email: #enter email to be used to access MHN GUI  
> Superuser password:  #enter password to be used to access MHN GUI  
> Server base url ["http://1.2.3.4"]: #leave blank to keep default  
> Honeymap url ["http://1.2.3.4:3000"]: #leave blank to keep default  
> Mail server address ["localhost"]: #leave blank to keep default  
> Mail server port [25]: #leave blank to keep default  
> Use TLS for email?: y/n #select "n"  
> Use SSL for email?: y/n #select "n"  
> Mail server username [""]: #leave blank to keep default  
> Mail server password [""]: #leave blank to keep default  
> Mail default sender [""]: #leave blank to keep default  
> Path for log file ["mhn.log"]: #leave blank to keep default  


You will then prompted to integrate MHN with Splunk, ELK, and UFW. You can choose to do this or not. Once all of that is done, then we can login to our web app:
http://hostname or IP  

## UFW configuration
Now we will configure UFW ( a simple firewall on ubuntu) to restrict access to only users configured to access my MHN server, honeypots, myself. MHN uses port 3000 and 443, so will have to allow connections to those ports.

**Note:** iptables will be a better firewall to use here in order to maintain stateful connections but because i’m not running super sensitive things here, chose to go with ufw to keep things simple.

> sudo apt-get install ufw (should already be installed)  
> sudo ufw enable  
> sudo allow 443  
> sudo allow 3000  
> sudo allow 22  
> sudo allow from “my ip”  
> sudo allow from “my honeypot”  

These policy list will continue to grow as honeypots are being added to the honeynet.


## Deploying honeypots  
Now we can login to the MHN server using the credentials we created earlier. Once we are logged in to our web app, we can deploy honey pots to other servers by generating a deployment script:  
> * On the main dashboard of the web app, click ‘deploy’ in the top left corner of the page.
> * Select the type of honeypot you wish to deploy.
> * Copy the deployment command
> * Login to your honeypot server and run the command as root.

**NOTE:** It is recommended to run honeypots on a different server from MHN server. Multiple honeypots can be run on the same server. 

Choosing which honeypot to run depends on the amount of interaction you will like to make with each one. This [guide](https://www.symantec.com/connect/articles/guide-different-kinds-honeypots) provides good details on the classification of honeypots. I configured a couple of honeypots across GCP and AWS.


## Integrating MHN with Splunk
At this point, we should be getting some traffic on one or more of our honeypots. The MHN web app is great at depicting the source ip and locations of the attacks; i will like to further explore the traffic being monitored by forwarding logs from the MHN server to Splunk.  

First we need to start generating logs on the MHN server:  
> * cd /opt/mhn/scripts/
> * sudo ./install_hpfeeds-logger-splunk

Now we need to install Splunk and configure forwarders from the MHN server. This [guide](https://medium.com/@smurf3r5/splunk-enterprise-on-digital-ocean-ubuntu-16-x-95c31c7e7e2c) explains how to install splunk on Linux machines. Once we have splunk installed and have accepted the license, we can now link the MHN Splunk app with our Enterprise splunk. The MHN splunk [app](https://splunkbase.splunk.com/app/2707/) comes with a prebuilt collection of dashboards that can be utilized or tweaked to analyze our traffic.

## Splunk our log file
Our logfile (‘/var/log/mhn/mhn-splunk.log’) file should have been logging all types of attacks on our honeynet. Now we need to populate our dashboard with the data:  
> * Select settings from the splunk server 
> * Select Data input > File & Directories
> * Navigate to the logfile
> * Click through the rest of the steps with the defaults

## Reports 
### Top Attacks

![alt text](https://github.com/adepoju/MHN-setup/blob/master/mhn_dashboard.png "MHN Top Attacks")

### Attacks by Geolocation

![alt text](https://github.com/adepoju/MHN-setup/blob/master/mhn_attacks.png "Attacks by Geolocation")


Over the next few weeks, these reports will continue to build; i will work on getting value out of the data being generated.
