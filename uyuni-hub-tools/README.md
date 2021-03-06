# hubtools
Tools to install and configure SUSE Manager Hub.

##
Files:
/opt/uyunihub/hub_dailyrun.py
/opt/uyunihub/hub_scp_bootstrap_repos.sh
/opt/uyunihub/smconfig.yaml
/srv/formula_metadata/uyunihub/form.yml
/srv/formula_metadata/uyunihub/metadata.yml
/srv/salt/uyunihub/uyunihub.sls
/srv/salt/uyunihub/update_config_channels.py
/srv/salt/uyunihub/sync_software.py
/srv/salt/uyunihub/register_slave.py
/README.md

## Installation instructions

* download the files and copy to the hub-master
* populate file `/opt/uyunihub/smconfig.yaml` with the needed properties
* Run /opt/uyunihub/hub_dailyrun.py is their are only SUSE Manager Server registered to this master. If not, enter the systemgroup name as parameter (eg hub-slaves)
* In the GUI of the SUSE Manager HUB master, create a systemgroup (eg hub-slaves)
* Under the tab **Formulas** select **Uyunihub**
* There will be a new tab with the name **Uyunihub** and fill in all the information.
* If there are already hub slaves, add them to the system group

## Add a new SUSE Manager as a HUB slave server
* Install a new SUSE Manager Server
* Register the server against the SUSE Manager HUB master as a salt minion
* Configure the SUSE Manager Server via yast susemanager_setup
* On the SUSE Manager HUB server:
  * Add the system to the above created systemgroup.
  * On the systemgroup details page go to the tab **Formulas** and select sub-tab **Uyunihub**
  * Add the new server as slave and, if needed, add any extra base channels the hub-slave should receive
  * Perform a high state on the new SUSE Manager hub slave

## Daily jobs running on SUSE Manager HUB master
* Create a cron job to run the script /opt/uyunihub/hub_scp_bootstrap_repos.sh daily. Or if you want weekly. This script will sync the bootstrap repositories to all systems.  
* Create a cron job to run the script /opt/uyunihub/hub_dailyrun.py daily. Or if you want weekly. This script will update the /srv/formula_metadata/uyunihub/form.yml with actual data. This should also run after installing these tools.
* example:

## Jobs running on SUSE Manager HUB slave
* The job update_config_channels.py will run every day and will update all salt configuration channels. The highstate will create a cron job for this. This will be logged to /var/log/rhn/uyunihub/update_config_channels.log
* The job sync_software.py will normally only run during a highstate. When channels to be synchronized are changed (currently only adding, see below), a highstate has to be performed on all slaves. This highstate will update the uyunihub.yaml and execute this script. Every night, taskomatic will synchronize all assigned channels automatically. This will be logged to /var/log/rhn/uyunihub/sync_software.log. 
* The job register_slave.py will normally only run during a highstate and only the first time. This will be logged to /var/log/rhn/uyunihub/register_slave.log 

## What is not in (on the moment)
* Software channels and configuration channels are not been removed via the formula. 

## Known issues
* revision numbers of init.sls can not be set using the API. They will be reported as to be changed on every run of update_config_channels.py
* revision numbers of none init.sls can not be set using the API on creation. During the next change, the revision will be updated. They will be reported as to be changed on every run of update_config_channels.py

