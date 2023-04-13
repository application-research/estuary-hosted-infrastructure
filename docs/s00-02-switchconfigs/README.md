# 🚧 Stage 00 - Step 2 - Core switch + BMC switch Configuration 🚧
* Plug in the core switch.
* Plug in an out of band serial device to its console port and login.
* Copy switch-serial.cfg.dist to switch-serial.cfg and fill it out with the settings needed to access the serial console. 🚧
* Copy switch_settings.cfg.dist to switch_settings.cfg and fill it out with the details of your IP address configurations. 🚧
* Run `python3 switches.py --initial` to apply the initial configuration settings (Might end up as Ansible, probably) 🚧

At this point your switch installation is completed, but you will need to set up the cabling before you can get WAN and LAN working.