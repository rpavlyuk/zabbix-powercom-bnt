# zabbix-powercom-bnt
Zabbix templates and agent files to monitor Powercom BNT-xxx UPS series

## Solution
The template allows to monitor core UPS parameters:
* Input/output AC voltage
* Input/output frequence
* Battey Charge level (%)
* Load level (%) 

Access to Powercom UPS device is made via [Network UPS Tools / NUT] (http://networkupstools.org) client. This means that NUT CLient `upsc` has to be installed on the box where Zabbix Agent is running.

Template also has two graphs built-in:
* AC Voltage
* UPS Load

## Supported Hardware
* Powercom BNT-xxx

Tested on:
* Powercom BNT-800AP
* Powercom BNT-600AP

## Installation
Installation instructions provided below are for RedHat-family Linux, however it won't be much different for other distros.

### On a box where Zabbix Agent is running
* Install NUT packages:
```
sudo yum install nut nut-client
```
* Scan your system to find how NUT sees your UPS. Here
```
sudo nut-scanner -U
```
* A sample of an output:
```
[rpavlyuk@liberty ~]$ sudo nut-scanner -U
Neon library not found. XML search disabled.
Scanning USB bus.
[nutdev1]
	driver = "usbhid-ups"
	port = "auto"
	vendorid = "0D9F"
	productid = "0001"
	bus = "002"
```
* Configuration section that `nut-scanner` produced shall be added to NUT configuration file `/etc/ups/ups.conf`
* (Re)start NUT services
```
systemctl restart nut-server
```
* Enable NUT service to start on boot
```
systemctl enable nut-server
```
* Test if `upsc` can connect to UPS and pull the data (assumed that device in `/etc/ups/ups.conf` is named `nutdev1`)
```
upsc nutdev1
```
* A correct (working) output should be like this:
```
[rpavlyuk@liberty ~]$ upsc nutdev1
battery.charge: 100
battery.temperature: 0
device.mfr: PowerCOM
device.type: ups
driver.name: usbhid-ups
driver.parameter.bus: 002
driver.parameter.pollfreq: 30
driver.parameter.pollinterval: 2
driver.parameter.port: auto
driver.parameter.productid: 0001
driver.parameter.synchronous: no
driver.parameter.vendorid: 0D9F
driver.version: 2.7.4
driver.version.data: PowerCOM HID 0.5
driver.version.internal: 0.41
input.frequency: 50
input.voltage: 228
output.frequency: 50
output.voltage: 228
ups.load: 6
ups.mfr: PowerCOM
ups.mfr.date: 1980/02/08
ups.productid: 0001
ups.status: OL
ups.vendorid: 0d9f
```
* Now, let's set up Zabbix Agent. Copy configuration parameters file available in this project to the target location:
```
sudo cp -a ./agent/etc/zabbix/zabbix_agentd.conf.d/powercom_ups_mon.conf /etc/zabbix/zabbix_agentd.conf.d/
```
* Restart Zabbix Agent
```
systemctl restart zabbix-agent
```
### On Zabbix Server
* Log in as a user with administrative privileges
* Go to **Configuration** -> **Templates** and click _Import_ button
* Select file `zbx_template_Powercom_BNT-xxx_NUT.xml` as a file to import
* Once imported, link the template to the host configuration which runs Zabbix Agent configured in previous section
* You should see the data from UPS coming in a while on **Monitoring** -> **Latest Data** page

