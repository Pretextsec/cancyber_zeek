# CanCyber Zeek Module

For [CanCyber.org](https://cancyber,org) members. The **CanCyber Foundation** provides free threat hunting capabilities to Canadian industry and their suppliers. With funding provided by the Government Canada and *Canadian Safety and Security Program* [(CSSP)](http://www.science.gc.ca/eic/site/063.nsf/eng/h_5B5BE154.html).

## Access

An API key in config.zeek is required to use this module with CanCyber.org. Use [dovehawk.io](https://dovehawk.io) to connect to your own MISP instance. When you [download a package](https://endpoint.cancyber.org/tool.php) from CanCyber.org the api key should be automatically included. If you are downloading source from GitHub, add your API key to config.key.

## Summary

This module uses the the built-in Zeek Intelligence Framework to load and monitor signatures from MISP enriched with commercial indicators and manually curated Zeek content based signatures (Zeek Signature Framework).  The module also includes a customized version of Jan Grashoefer's expiration code.

The script, signatures, and indicators are downloaded automatically every 6 hours.  Indicators should expire after 6.5 hours if removed from MISP.

Indicators are downloaded and read into memory automatically.  Content signatures are stored locally in `signatures/cancyber_sigs.sig` and will require a zeek restart `zeekctl restart` to take effect.


## Requirements

**Zeek** 3.0 or higher.

**zkg** *zeek package manager*.

**curl** is required for ActiveHTTP requests.


## Extra Documentation

CanCyber general [Documents Repo](https://github.com/cancyber/documents])

## Slack Channel

[CanCyber Slack](https://cancyber.slack.com)

[Request an invite](https://cancyber.org/contact.php)


## Monitoring and Intelligence Context

The CanCyber Zeek module outputs hits to the console, logs to file, and reports hit metadata and up to 2000 bytes of a signature-hit detected session to the CanCyber dashboard.  The [CanCyber Dashboard](https://dashboard.cancyber.org/) is the best place to review hits and get context on the activity and get actor attribution.  Additionally, analysts are available in the [Slack group](https://cancyber.slack.com) to help interpret hits.


## Web Dashboard

[CanCyber Dashboard](https://dashboard.cancyber.org/)


## Module contents:

`scripts/cancyber_sigs.zeek`: Module zeek-script source.

`scripts/cancyber_expire.zeek`: Expiration code, derived from Jan Grashoefer

`signatures/cancyber_sigs.sig`: Content based signatures.

`__ load __.zeek`: Module designator.

`config.zeek`: Your API key goes in here.

`config.zeek.orig`: Clean copy of config.zeek.

`zkg.meta`: Zeek package manager identifier.

`update.py`: Python script to update signatures.

`README.md`: This read me file.

`cluster.md`: zeekctl cluster setup for multiple servers.



## Package Install

1. Install Zeek

  - Mac: `brew install zeek`

  - Ubuntu: `apt-get install zeek`

  - Centos: `yum install zeek`

  Configure zeek interface setting: */usr/local/etc/node.cfg*

```
[zeek]
type=standalone
host=localhost
interface=en0
```

2. Install zpk (Zeek package manager):

Requirements: [Python 3](https://realpython.com/installing-python/) and [pip](https://bootstrap.pypa.io/get-pip.py).

`pip install zkg`

3. Setup zkg:

  - `zkg autoconfig`

  - Edit *site/local.zeek* (example location */usr/local/Cellar/zeek/3.1.1/share/zeek/site/local.zeek*)

    `@load packages`

4. Install cancyber_zeek.bundle package:

  From a preconfigured (API key included) install cancyber_zeek.bundle [download here](https://endpoint.cancyber.org/tool.php):
  
  - `zkg unbundle cancyber_zeek.bundle`

```
The following packages will be INSTALLED:
https://github.com/cancyber/cancyber_zeek (master)

Proceed? [Y/n] `Y`
Loaded "https://github.com/cancyber/cancyber_zeek"
Unbundling complete. 
```

The cancyber_zeek.bundle is tar.gz compressed file pre-loaded with your API key and the most recent CanCyber content signatures.

  **Or:**

  From Github source:
  
  - `zkg install https://github.com/cancyber/cancyber_zeek` (then edit the *packages/cancyber_zeek/config.zeek* to have your cancyber tool api key [not misp key]). Copy a new config `cp config.zeek.orig config.zeek`.
  
```
module cancyber_zeek;

export { 

	global APIKEY = "**##APIKEY##**";

	global CCSOURCE = "zeek"; # available options zeek, isp, dev, exp
}
```

When installing from Github source you'll need an API key to add to `config.zeek`. To find your API key, go to the [tools download page](https://endpoint.cancyber.org/tool.php) and Scroll down to the **Key Revokation** section to grab a recent key. API keys are system-generated 64 digit alphanumeric sequence similar to a sha256.

5. Test run Zeek with the cancyber_zeek module from the command line:

`zeek -i en0 cancyber_zeek`

Typical errors here would be missing *config.zeek* or network errors in enterprise environments connecting to tool.cancyber.org.


```
listening on en0

Refresh period is now 6.0 hrs
Downloading CanCyber Signatures 2020/03/24 12:02:03
Cancyber Source Directory: /usr/local/Cellar/zeek/3.1.1/share/zeek/site/cancyber_zeek/./scripts/.
Downloading Indicators...
Processing Indicators...
Number of Indicators 426893
 Intel Indicator Counts:
    Intel::DOMAIN:    36275
    Intel::ADDR:        4464
    Intel::URL:        250793
    Intel::SUBNET:    0
    Intel::SOFTWARE:  10
    Intel::EMAIL:     826
    Intel::USER_NAME: 0
    Intel::FILE_HASH: 127887
    Intel::FILE_NAME: 6637
Finished Processing Indicators
```

Control-C to exit.
```
1585065905.746575 received termination signal
1585065905.746575 109231 packets received on interface en0, 8406 (7.15%) dropped
Zeek Terminating - Cancelling Scheduled Signature Downloads
```

6. [Test indicators](https://cancyber.org/testing.php)

`nslookup malware-c2.com`

```
Server:		8.8.8.8
Address:	8.8.8.8#53

Non-authoritative answer:
Name:	malware-c2.com
Address: 35.183.9.38
```

Zeek will print a sighting and send it to CanCyber:

```
CanCyber Sighting: ZEEK|uid:C4eMxUSgTo4FzJsg4|ts:1585053181.470496|orig_h:192.168.1.90|orig_p:63181/udp|resp_h:8.8.8.8|resp_p:53/udp|msg: Intel hit on malware-c2.com at DNS::IN_REQUEST|node:zeek|d:OUTBOUND|service:DNS|orig:32|o_pkts:0|o_bytes:0|o_state:1|resp:0|r_pkts:0|r_bytes:0|r_state:0|start_time:1585053181.470496|duration:0.0|q:A
Sighting Result ===> {"result":"Hit Recorded!"}
```

7. zeekctl deployment for always on monitoring

  - `zeekctl deploy` (typical errors here would be missing *config.zeek*).
  
```
checking configurations ...
installing ...
removing old policies in /usr/local/var/spool/installed-scripts-do-not-touch/site ...
removing old policies in /usr/local/var/spool/installed-scripts-do-not-touch/auto ...
creating policy directories ...
installing site policies ...
generating standalone-layout.zeek ...
generating local-networks.zeek ...
generating zeekctl-config.zeek ...
generating zeekctl-config.sh ...
stopping ...
stopping zeek ...
starting ...
starting zeek ...
```

  
  Zeekctl will install the module and start it. See the cron section below to add the maintenance commands.


8. Review [CanCyber Dashboard](https://dashboard.cancyber.org/)


## Command line alternative:

If running zeek directly on the command line, reference the CanCyber folder with the module:

`sudo zeek -C -i en0 cancyber_zeek`

```
listening on en0

Refresh period is now 6.0 hrs
Downloading CanCyber Signatures 2020/03/24 08:28:53
Cancyber Source Directory: ./cancyber_zeek/./scripts/.
Downloading Indicators...
Updating File ../signatures/cancyber_sigs.sig
Finished Updating File: ../signatures/cancyber_sigs.sig
Processing Indicators...
Number of Indicators 426067
 Intel Indicator Counts:
    Intel::DOMAIN:    36232
    Intel::ADDR:        4157
    Intel::URL:        250790
    Intel::SUBNET:    0
    Intel::SOFTWARE:  9
    Intel::EMAIL:     819
    Intel::USER_NAME: 0
    Intel::FILE_HASH: 127422
    Intel::FILE_NAME: 6637
Finished Processing Indicators
```


## Zeekctl Maintenance

Use zeekctl to launch zeek, rotate logs, and restart after crashes.


### zeekctl cron jobs

```
# cron regular check
*/5 * * * * /usr/local/bin/zeekctl cron

# restart to load new signatures twice daily
1 */12 * * * /usr/local/bin/zeekctl restart
```

## Updates

To upgrade to the latest version of the CanCyber Zeek module using zkg, execute:

`zkg upgrade`

```
The following packages will be UPGRADED:
  https://github.com/cancyber/cancyber_zeek (master)

Proceed? [Y/n] `Y`
Upgraded "https://github.com/cancyber/cancyber_zeek" (master)
```

## Uninstall

- Stop Zeek

  `zeekctl stop`
  
- Remove package

  `zkg remove cancyber_zeek`
  
```
The following packages will be REMOVED:
  https://github.com/cancyber/cancyber_zeek

Proceed? [Y/n] **Y**
Removed "https://github.com/cancyber/cancyber_zeek"
```


