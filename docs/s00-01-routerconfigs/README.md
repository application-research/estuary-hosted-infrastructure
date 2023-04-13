# Stage 00 - Step 1 - MikroTik Initial Router Configuration 🚧
* Plug in the MikroTik router.
* Plug in an out of band serial device to its console port and login.
* Copy ip_settings.cfg.dist to ip_settings.cfg 🚧
* Run `python3 mikrotik.py --initial` to apply the initial configuration settings (Might end up as Ansible, probably) 🚧

At this point your MikroTik installation is completed, but you will need to set up the core switch + cabling before you can get WAN and LAN working.