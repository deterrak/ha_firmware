#!/bin/bash

################################
# 1) Install the Prerequisets:
################################

# Update your apt repository
sudo apt-get update

# Install expect:
sudo apt -y install expect

################################
# install optional packages:
################################
# Install samba to allow in-place edits of the exel file via a windows based computer (optional) 
sudo apt -y install samba

# Change the samba configuration file
# first lets backup your configuration, just in case.
sudo cp /etc/samba/smb.conf smb.conf_backup

# next copy in our new samba configuraiton
sudo cp smb.conf_example /etc/samba/smb.conf

# restart samba using the new file.
sudo service smbd restart

# you should now be able to access the /home/stanley/ folder using a windows file share

############################
# 3) Setup your SCP server:
############################

# you need to configure a user on an SSH enabled machine with sufficient disc space to store multiple. I've been using a SCP enabled fileserver with a user called 'firmware'. There is a tab in the excel file for the fileserver user credentials.

# NOS operating systems. You must also configure the excel spread sheet to reflect the IP address of
# this machine and the user log in credentials. each NOS version is stored in the path:
#      /home/firmwareUpdate/NOS/<nosX.y.z>

############################################
# 4) Copy the expect scripts to /home/stanley
############################################

# The expect scripts are located in /opt/stackstorm/packs/stanley/
# you will need to copy these files to /home/stanley/

sudo cp stanley/*exp /home/stanley/

# Make them executable
sudo chmod a+x /home/stanley/*.exp

#################################
# 5) create the logFile directory
#################################
LOG_DIR="/home/stanley/updateFirmwareLogs/"
if [ ! -d "$LOG_DIR" ]; then
  # Control will enter here if $DIRECTORY doesn't exist.
  mkdir $LOG_DIR
fi

##################################
# 6) install the ha_firmware pack
##################################

# the location for packs is /opt/stackstorm/packs/
# untar the ha_firmware archive and install it in the packs folder.

# make sure that you are logged in and have a token.

st2 login -w st2admin

# now install the ha_firmware pack
st2 run packs.setup_virtualenv packs=ha_firmware

# if upgrading, it is safer to remove first, then intall the new version.
# st2 pack remove ha_firmware

#############################################
# 7) Describe your network in the excel file.
#############################################

# The firmwareUpdate.xlsx file will describe your network. It is where you will 
# define each VDX switch in your network. This includes the login credentials, the 
# switch IPs, and the links that compose the network topology. There are inster-switch 
# link (ISL) which comporise the VDX fabric, as well as links that connect to server 
# and to the core network. Interfaces that are mission-critial are defined in the Excel 
# file. For each interface a health test is run prior to conducting the firmware upgrade. 
# All these links must be up inorder to preserve the integrity of the HA posture. The 
# workflow will fail if the health of the network is compromised. 

###############################
# 8) Setup your firmware server
###############################

# Firmware files can be located anywhere. The default transport is SFTP. The parent directory is 
# ~/NOS/ all firmware files are stored under this folder. 
# The fileserver's IP and loging credentials are stored in the fimrwareUpdate.xlsx file.

############################################################
# 9) Run the 'ha_firmware.upgradeFirmwareMistral' workflow.
############################################################

# From the actions tab, select the ha_firmware pack and run the 'upgradeFirmwareMistral' workflow.
# Select the 'upgrade group' that you wish to perform the upgrade on.
# Choosd 'run' to activate, monitor the progress from the history tab.

