# CUPS + AirPrint on Raspberry PI
copied from http://linuxwin.com/cups-airprint-on-raspberry-pi/ as backup

### CUPS

1. Install CUPS and Enable Remote Access
```console
apt-get install cups
```

2. Add Pi user to lpadmin group
```console
usermod -aG lpadmin pi
```

3. Enable Remote Access by commenting Listen localhost:631 and add Port 631 or Listen 631 then Allow access
```console
sudo nano /etc/cups/cupsd.conf
```
Content
```console
# Only listen for connections from the local machine
# Listen localhost:631
Port 631

< Location / >
# Restrict access to the server...
Order allow,deny
Allow @local # or all
< /Location >

< Location /admin >
# Restrict access to the admin pages...
Order allow,deny
Allow @local # or all
< /Location >

< Location /admin/conf >
AuthType Default
Require user @SYSTEM

# Restrict access to the configuration files...
Order allow,deny
Allow @local
< /Location >
```

4. Restart CUPS
```console
/etc/init.d/cups restart
```

Now you are able to access remotely x.x.x.x:631 and add the printer


### AirPrint

1. Install AirPrint and configure it
```console
apt-get install avahi-discover cups cups-pdf python-cups
apt-get install hplip
```

2. Make your old printer AirPrint capable
 
```console
/etc/init.d/avahi-daemon start
mkdir /opt/airprint
cd /opt/airprint
wget -O airprint-generate.py --no-check-certificate https://raw.github.com/tjfontaine/airprint-generate/master/airprint-generate.py or if down https://raw.github.com/Ahaus314/airprint-generate/master/airprint-generate.py
chmod +x airprint-generate.py
sudo ./airprint-generate.py -d /etc/avahi/services
/etc/init.d/avahi-daemon restart
ls /etc/avahi/services/
```

3. Add those files with those contents
```console
sudo nano /usr/share/cups/mime/local.convs
```
Content
```console
image/urf application/vnd.cups-postscript 66 pdftops
```

```console
sudo nano /usr/share/cups/mime/apple.types
```
Content
```console
image/urf urf (0,UNIRAST)
```

```console
sudo nano /usr/share/cups/mime/apple.convs
```
Content
```console
image/urf application/pdf 100 pdftoraster
```

4. Open port 631
```console
iptables -A INPUT -i eth0 -p tcp -m tcp –dport 631 -j ACCEPT
iptables -A INPUT -i eth0 -p udp -m udp –dport 631 -j ACCEPT
```

5. Disable power saving (optional)
```console
sudo nano /etc/modprobe.d/8192cu.conf
```
Content
```console
# Disable power saving
options 8192cu rtw_power_mgnt=0 rtw_enusbss=1 rtw_ips_mode=1
```

### Connect from Windows
http://server_ip:631/printers/printer_name
