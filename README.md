# Fail2Ban Ban Repeat Offenders

It's difficult to find a concise tutorial on how to ban repeat offenders with fail2ban. In this small tutorial, we will be using the built in **recidive.conf** to ban repeat offenders. For a server that is repeatedly hit with malicious login attempts, this will increase security and clear the logs. Note that this methodology was completed on Ubuntu 16.04 with fail2ban version 0.9.3-1.

## Install fail2ban and make a jail
If you haven't already, install fail2ban:
`sudo apt-get update && sudo apt-get install fail2ban -y`
And now create your first jail.
`sudo nano /etc/fail2ban/jail.local`
Here are the files for the entire jail:

    [sshd]
    enabled = true  
    port = 22  
    filter = sshd  
    logpath = /var/log/auth.log  
    findtime = 600  
    maxretry = 10  
    bantime = 600  
    ignoreip = //YOUR IP HERE//
    
    [recidive]  
    enabled = true  
    port = 22  
    filter = recidive  
    logpath = /var/log/fail2ban.log  
    findtime = 86400  
    maxretry = 2  
    bantime = 3600  
    ignoreip = // YOUR IP HERE //
    protocol = all  
    action = iptables-allports[name=recidive, protocol=all]

Here's an explanation of what's happening above.

 - The sshd filter monitors ssh connections in */var/log/auth.log* and will ban users for **600 seconds** after they have had 10 bad failed attempts (feel free to change any values).
 - The recidive filter monitors the list of users sshd has banned in */var/log/fail2ban.log*. If it sees an IP twice, then it will ban that IP for 3600 seconds. This is the **repeat offender** banning.
 - I'm not sure if action and protocol are required, but I'd rather have them in the file anyway.


## Finishing up and glossary
When converting from an existing fail2ban install, you'll have to do a feel things differently.

    sudo service fail2ban stop
    echo > /var/log/fail2ban.log 
    ## NOTE: you just need to clear out fail2ban.log, might have to sudo -i, then run that command
    sudo service fail2ban start
    sudo service fail2ban status

This will start with a fresh log, which may be necessary to get the desired bans. Make sure to run **sudo service fail2ban restart** after any major changes.

### Glossary:
**findtime**: window of time to allow retry attempts, in seconds

**maxretry**: amount of times a user can retry their username/password

**bantime** : amount of time to ban IP address, in seconds

**ignoreip**: whitelist of IPs to never ban


