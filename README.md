# Podman Quadlets, what are they
Podman Quadlets are a way of handling podman containers with Systemd, by creating Unit files for the containers.

## How does it work?
A Podman Quadlet needs a file.container file. 
This is a podman systemd file - https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html

The .container file, can be compaired to a docker-compose file. It is a declarative file, where we describe the options we want to run the container with.

```
[Unit]
Description=Oneline description goes here
After=List here which target this should start after.

[Container]
Image=URL to the image. Preferably the full path, IE docker.io/semaphoreui/semaphore:latest
ContainerName=Name of the container. This is will be the name in podman ps.
PublishPort=HostPort:ContainerPort
Environment=VARIABLENAME_IN_CAPS=value
EnvironmentFile=path to file with environment variables in it
Volume=HostPath:ContainerPath
Network=path to a .network file, if there is one
# See link above for more

[Service]
Restart=Restart Policy, Standard Systemd restart policy
TimeoutStartSec=900 # Systemd standard timeout is 90s, which can be too little to pull the image.

[Install]
WantedBy=list here which targets is required by this systemd unit.
```

**IMPORTANT NOTE: unlike docker-compose or podman-compose quadlets do not create the folders we specify under volumes, so be sure to create them manually with "mkdir -p /path/to/folder/for/volume"**

If we do not want our environment variables inside our .container file (like password and such), we can instead use the EnvironmentFile option, and point that to a file that contains all the environment variables for our container.
This file would be formated with one variable per line, like so:
```
VARIABLENAME_IN_CAPS=value
VARIABLENAME_IN_CAPS=value
VARIABLENAME_IN_CAPS=value
VARIABLENAME_IN_CAPS=value
VARIABLENAME_IN_CAPS=value
```

If we have several containers, that needs to work together, we can create a .network file.
Remember not to use the same network information as your "normal" network. This is an internal podman network.
```
# network ipaddr/cidr
Subnet=xxx.xxx.xxx.x/xx
# defgw ipp addr
Gateway=xxx.xxx.xxx.x
#label that describes what network is used for
Label=app=xxxxxxxx
```

Once the files we need have been created, we need to move them to where Systemd can find them. Where depends on if we are running our podman containers as **rootfull** or **rootless**

### rootfull

Copy the .container file, along with any .env and .network files created to one of the following locations:
  -  /etc/containers/systemd/
  -  /usr/share/containers/systemd/

When the files are in place we need to tell systemd about them. we do this by doing a daemon reload
```python
systemctl daemon-reload
```
This makes systemd create a filenameBeforeDotContainer.service file for our container. It places this in a subfolder in /run. Where it goes doesnt really matter much, because we only interact with the .service file through systemd.

Keep an eye on the systemd log (journalctl) while doing so, because any errors in parsing the .container file will show up there.
If no errors show up, we can use systemctl to check if the service file has been created
```bash
systemctl status filenameBeforeDotContainer.service
```

We can now start and stop the service like we do all other systemd services.
```bash
systemctl start filenameBeforeDotContainer.service
systemctl stop filenameBeforeDotContainer.service
```

### rootless

Copy the .container file, along with any .env and .network files created to one of the following locations:
  -  $XDG_CONFIG_HOME/containers/systemd/ or ~/.config/containers/systemd/
  -  /etc/containers/systemd/users/$(UID)
  -  /etc/containers/systemd/users/

When the files are in place we need to tell systemd about them. we do this by doing a daemon reload
```python
systemctl --user daemon-reload
```
This makes systemd create a filenameBeforeDotContainer.service file for our container. It places this in a subfolder in /run. Where it goes doesnt really matter much, because we only interact with the .service file through systemd.

Keep an eye on the systemd log (journalctl) while doing so, because any errors in parsing the .container file will show up there.
If no errors show up, we can use systemctl to check if the service file has been created
```bash
systemctl --user status filenameBeforeDotContainer.service
```

We can now start and stop the service like we do all other systemd services.
```bash
systemctl --user start filenameBeforeDotContainer.service
systemctl --user stop filenameBeforeDotContainer.service
```

### logging

when using Quadlets, podman will send systemlogs to the SystemD Journal. This includes the podman run output (where we can see if our container start, and if it does not, why it doesnt.) If we forget the create the required folders, the error messages will be output to the SystemD Journal. 

To see these messages, we can use the journalctl command.
```bash
journalctl -fe
```

The root user (and members of the wheel and adm groups) has access to this command out of the box.
If we are running our Quadlets as rootless, we need to give the user we are running them as, access to the journalctl command. This can be done by adding the user to the "systemd-journal" group
```bash
# this has to be done as root, or as a user with sudo rights...
usermod -aG systemd-journal <username>
```



## Will my Containers start on boot?
If you set the restart policy above to "always", then systemd will start the container on boot. It will also start it again, if you stop it with podman stop... It must be stopped with systemctl stop.
