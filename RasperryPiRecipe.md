RasperryPi Recipe

* [Start SSL service for putty](#sslserver)
* [Raspberry PI – Create demon service (timelapse)](#networkshare)
* [Raspberry PI – Mounting a network share](#raspberrypi–mounting-a-network-share)

# <a name="sslserver"></a> Start SSL service for putty
```
sudo apt install openssh-server 
```

# <a name="networkshare"></a> Raspberry PI – Create demon service (timelapse)
```
sudo nano /lib/systemd/system/pitimelapse.service
```

```
# file: /lib/systemd/system/pitimelapse.service

[Unit]
Description=Python timelapse Service
After=multi-user.target

[Service]
Type=idle
ExecStart=/usr/bin/python /home/pi/timelapse/timelapse.py > /home/pi/timelapse/timelapse.log 2>&1
WorkingDirectory=/home/pi/timelapse
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```

Run

```
sudo systemctl daemon-reload
sudo systemctl start pitimelapse.service
```

For easy use add following alias to .bashrc

```

alias tlstart="sudo systemctl start pitimelapse.service"
alias tlstatus="sudo systemctl status pitimelapse.service"
alias tlstop="sudo systemctl stop pitimelapse.service"

```


# <a name="raspberrypi–mounting-a-network-share"></a> Raspberry PI – Mounting a network share


```
 sudo nano /etc/fstab
 ```
 
 ```
 # file: /etc/fstab
 
proc            /proc           proc    defaults          0       0
/dev/mmcblk0p6  /boot           vfat    defaults          0       2
/dev/mmcblk0p7  /               ext4    defaults,noatime  0       1
//192.168.200.NAS/shared  /home/pi/network/     cifs vers=3.0,user=[username],pass=[password],rw,noperm,x-systemd.automount 0 0
```

Note: Network share via NAS (i.e. Synology) need to put noperm option to allow current pi user to be able to write to the network share.


```
mkdir /home/pi/network 
sudo mount /home/pi/network 
sudo reboot
```

Note: *Wait for network at boot* should be turned on to mount network share properly.

```
sudo raspi-config
-> Boot Option
-> Wait for Netowrk at Boot
```

