# Hitron Router Python Module
A Python 3 module to interact with [CGNV4 routers from Hitron Technologies](https://www.hitrontech.com/product/cgnv4/).

## Installation
### Requirements
You need [Python 3](https://wiki.python.org/moin/BeginnersGuide/Download) and [the Requests module](https://requests.readthedocs.io/),

### Process
Simply download hitron.py and you can either run it as a command or import as a module:

#### Command line usage
Running the script without any arguments outputs the help file:
```
usage: hitron.py [-h] [--reboot] [--status] [--test TEST] [--host HOST]
                  [--user USER] [--pw PW] [--force] [--retry RETRY]
                  [--verbose]

Communicates with Hitron CGNV4 router

optional arguments:
  -h, --help     show this help message and exit

Commands:
  Provide one or more of these to run on the device

  --reboot       Trigger reboot
  --status       Show router status
  --test TEST    Ping test to <destination-ip>

Router details:
  Any missing will be prompted on launch

  --host HOST    IP or hostname of Hitron router
  --user USER    Username for admin account on Hitron router
  --pw PW        Password for admin account on Hitron router

Options:
  Modify output of commands

  --force        Forces --reboot without testing
  --retry RETRY  Attempts to login (default: 2)
  --verbose      Increase verbosity of other commands
```

#### Module
You can import the module into a Python script like so:
```
#!/usr/bin/env python3
from hitron import Hitron
```

You can print a summary of information from the dashboards (this command is expected to be extended over time):
```
>>> router.status()
Device booted 2 hours, 44 minutes & 27 seconds ago
WAN connected to exchange with IP 1.2.3.4/24
VMB GRE tunnel configured and online with range Tunnel is testing online with IP range 5.6.7.8/29
```

You can run a simple ping test and reboot if that ping fails:
```
router = Hitron('ip', 'user', 'pass')
if not router.ping('8.8.8.8'):
  router.rebootAndTest('8.8.8.8')
```
When you run this script, it will:
* Attempt to login to "ip" using HTTP
* Ping 8.8.8.8 to test the router's internet connectivity
* If 0-1 pings fail it exits without doing anything
* If 2-4 pings fail it will:
  * Trigger a reboot
  * Attempt to login until the router comes back online
  * Attempt to ping 8.8.8.8 until they work

There are also several helper functions to obtain information from the router status pages:
```
>>> router.uptime()
'2 hours, 31 minutes & 13 seconds'
```

Specific fields can be obtained from the dashboard of the router:
```
>>> router.sysInfo('wanIp')
'70.80.90.100/24'
>>> router.sysInfo('swVersion')
'4.5.10.201-CD-UPC'
>>> router.sysInfo('nonExistentField')
False
```

You can output more information about how long each command takes to run by setting "times" to "True" like this:
```
>>> router = Hitron('ip', 'user', 'pass', times=True)
Login completed in 1 second

>>> router.ping('8.8.8.8')
0% packet loss over 4 pings to 8.8.8.8 in 4 seconds
```

### Virgin Media Business (CGNV44-FX4)
This module was originally written for [Virgin Media Business](https://www.virginmediabusiness.co.uk/help-and-advice/products-and-services/hitron-router-guide/) routers that provide static IPs over GRE tunnels.

If you have one of these routers, your GRE tunnel will negotiate _after_ the DOCSIS connection is up, so you can check the status of your GRE tunnel with this, which returns a tuple of (Status, Message) like so:
```
>>> router.vmbGreStatus()
(False, 'Tunnel is currently negotiating RADIUS')
>>> router.vmbGreStatus()
(True, 'Tunnel is testing online with IP range 50.60.70.80/29')
```

## TODO
* HTTPS login, I suspect this fails currently due to the self-signed SSL certificate
* Testing with other models/variants of Hitron router

## Updates2024

Model is Chita as found from http://192.168.0.1/1/Device/CM/Basicinfo
{"errCode":"000","errMsg":"","modelName":"CHITA","GuiStyle":"RED"}

JS code is http://192.168.0.1/webpages/lib/jquery.hitronext.js
isChita function returns true due to output of Basicinfo

Encrypt the login data as json:
{"username":"", "password":"", "forcelogin":"0"}

password is encrypted using SJCL (https://github.com/bitwiseshiftleft/sjcl)
key is username followed by password to encrypt.
parameters to SJCL:
var p = { adata: randomize(1, 0),
                iv: randomize(4, 0),
                salt: randomize(2, 0),
                iter: 10000,
                mode: 'gcm',
                ts: 64,
                ks: 128 };
In python we can use https://github.com/berlincode/sjcl:
from sjcl import SJCL

encrypt(self, plaintext, passphrase, mode="gcm", count=10000, dkLen=??):

Then post this json to: http://192.168.0.1/1/Device/Users/Login
json result should be returned containing result = success
If so then redirect to /index.html#status_system/m/1/s/1


