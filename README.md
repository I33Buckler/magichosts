# Magic Hosts

Do you need to map some domain names to LAN addresses in your hosts file?
Tired of having to manually edit /etc/hosts every time you change WiFi networks?
This script automatically detects a target WiFi network and turns on/off LAN routing in
your /etc/hosts file accordingly. It is also optimized to only make changes if you've
actually changed networks, and it's 100% cron compatible.

## Authors

- Jason C. McDonald (CodeMouse92)

## Compatibility

This should work on any Debian-based system, and may also work on other Linux
systems. It is designed to work with any system that uses `/etc/hosts` for
local DNS resolution.

**If you encounter any problems with any Linux system, please report in Issues!**

## History

I run a local file server, which faces the world wide web via a Dynamic DNS service
with a domain and several subdomains I own. Unfortunately, my router was not designed
to handle the scenario where I would connect to that server using its external IP
(and thus domain name) from a connection on the same router! After ruling out the
viability of several common workarounds, I arrived at the conclusion I would need to
edit the `/etc/hosts` file on every computer on our network.

Unfortunately, that meant changing `/etc/hosts` every time I connected from a
different network, such as the WiFi at my favorite café. Finally, I decided to write
this handy little cron script to do that for me.

## Prerequisites

This script merely uses the features and native components of BASH, as well as
`iwconfig`, which is on most Linux systems.

## Installation

1. Ensure your `/etc/hosts` file is already set up with a LAN routing - that is,
it should include an entry that starts with `192.168`.
*This line should be uncommented before starting the script.*

2. Place the `magichosts` file in a convenient place for scripts, and be sure
the location is included in your environment path. Mark the script as executable
via `chmod +x magichosts`.

Here's an example install, placing the script in your `/usr/local/bin`
directory. Note that we only need the `sudo` because we're working in a system
directory.

```bash
# Move to the location where the script will live.
$ cd /usr/local/bin
# Download the script file from GitHub directly.
$ sudo wget https://github.com/CodeMouse92/magichosts/raw/master/magichosts
# Make the script executable. (You are encouraged to read the file BEFORE doing this, so you know what it does.
$ sudo chmod +x magichosts
```

3. Edit `magichosts`, and change the variable `NETWORK` on line 40 to be the WiFi network name where you want LAN
routing enabled. For example, you might change it to...

```bash
NETWORK="FBIVanOutside"
```

If you have multiple WiFi networks on your router, but they all follow the same naming convention, you only
need to include the part they have in common. For example, if you have `MyGreatNet_2G` and `MyGreatNet_5G`,
just set `NETWORK` to `"MyGreatNet"`.

4. Run the script for the first time, and then verify that `/etc/hosts/` is correct based on your network.
The script MUST be run with `sudo`.

```bash
$ sudo magichosts
$ cat /etc/hosts
```

Ensure the output of `/etc/hosts` is correct, given your internet connection.

5. Enable the script according to your preferred scheduler
- ### cron<br>
  Add the script to your *root* cron, via `sudo crontab -e`. Make sure you give the full path to the script.
  ```cron
  * * * * * /usr/local/bin/magichosts
  ```
  The script will run every minute (unless you modify the line).
  
- ### systemd<br>
    1. Change directory to /etc/systemd/system
    - Create the file `magichosts.service` with the following
    ```
    [Unit]
    Description=Script to swith /etc/hosts when connected to LAN at home
    
    [Service]
    Type=oneshot
    ExecStart=/usr/local/bin/magichosts
    ```
    - Create the file `magichosts.timer` with the following to run the contents of `magichosts.service` every minute.
    ```
    [Unit]
    Description=Check for wireless connection every minute
    
    [Timer]
    OnCalendar=*-*-* *:*:00
    Persistent=Ttrue
    
    [Install] 
    WantedBy=timers.target
    ```
    Confirm the details of the scheduling parameters. This will show the next 5 scheduled instances according to the parameters for OnCalendar.
    ```
    systemd-analyze calendar --iterations=5 '*-*-* *:*:00'
    ```
    Information on systemd timers can be found at https://wiki.archlinux.org/title/Systemd/Timers
  
    2. Enable the systemd timer with the following
       
      1. Reload systemd

sudo systemctl daemon-reload

ii) Enable the timer to run on boot

sudo systemctl enable magichosts

iii) Start the timer immediately

sudo systemctl start magichosts

    

## Usage

Under normal circumstances, `magichosts` will run without you needing to do anything. If you want
to force it to execute immediately, just run `sudo magichosts`.

The script follows these rules:

1. If there is no WiFi connection, it won't do anything.

2. If you have NOT switched between a target WiFi network and a non-target network since the last run,
it won't try to make any edits.

3. If you HAVE switched between a target WiFi network and a non-target network, it will either comment or
uncomment all the LAN routing lines (those starting with `192.168` in your `/etc/hosts` file.

## Known Issues

Right now, the script has the following limitations:

* It only checks the SSID of the connected WiFi network. It won't make changes based on other internet connections,
including ethernet and USB smartphone tether.

* If no WiFi network is connected, no changes will be made to `/etc/hosts`. Ever.
