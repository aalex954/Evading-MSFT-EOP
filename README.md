# Evading-MSFT-EOP
Some research into MSFT EOP and how to trick it.

## Original Intent: 
To bypass Microsoft Exchange Online Protection / Safe Links during phishing engagements by recognizing known EOP IPs and presenting an alternative web page. 

## Findings

A number of fingerprints were found in early testing. although the test was short some findings are interesting. 

<hr/>


### EOP and/or Safe Links accesses links within emails from IPs not officially published by MSFT

During testing a hit on the [HTTP-Fingerprinting-Server](https://github.com/aalex954/HTTP-Fingerprinting-Server) was logged:
```X-Forwarded-For: 40.94.27.42```

A check for ```40.94.27.42``` within the published Microsoft IPs did not result in a match.

- https://learn.microsoft.com/en-us/microsoft-365/enterprise/urls-and-ip-address-ranges?view=o365-worldwide
- https://endpoints.office.com/endpoints/worldwide?clientrequestid=b10c5ed1-bad1-445f-b386-b919946339a7

BUT! When using [ASN-2-IP](https://github.com/aalex954/ASN-2-IP) to query for IPs associated with MSFT owned AS blocks a match was found!

![image](https://github.com/aalex954/Evading-MSFT-EOP/assets/6628565/333fd5a0-17a8-434f-b98d-bb7cd10614f1)

_```40.94.27.42``` is contained in a MSFT owned subnet ```40.80.0.0/12```_

WHOIS

```
NetHandle:      NET-40-74-0-0-1
OrgID:          MSFT
Parent:         NET-40-0-0-0-0
NetName:        MSFT
NetRange:       40.74.0.0 - 40.125.127.255
NetType:        allocation
RegDate:        2015-02-23
Updated:        2021-12-14
Source:         ARIN


OrgID:          MSFT
OrgName:        Microsoft Corporation
```

<hr/>

### Additional Fingerprints

Although subject to change at any time a few consistent fingerprints were gathered.
Using some JavaScript the landing page of the "honeypot" posts some information about the browser back to the fingerprinting server.

Consistent values returned:

- screenResolution: 1280x960
- browserPlugins: ['Chrome PDF Plugin', 'Chrome PDF Viewer', 'Native Client']

### MSFT IPs

The cross-platform PowerShell script [ASN-2-IP](https://github.com/aalex954/ASN-2-IP) can be used to generate a csv containing all IP address blocks belonging to Microsoft. This script uses the BGPView API to generate all subnets associated with AS Block registrant names containing "microsoft".

Additionally, the [MSFT-IP-Tracker](https://github.com/aalex954/MSFT-IP-Tracker) repository publishes a daily csv containing the same above subnets.
note: Due to BGPView API IP access restrictions the API calls are blocked from GitHub CD actions. This prevents the search for AS Names containing "microsoft". As a workaround, a different API is used which allows only AS Numbers to be queried. This means I have to update the repository when AS Number ownership changes. This may cause inaccuracies.  

### EOP / Safe Links IPs

Once the process of fingerprinting detonated honeypot links is done a dynamically updated list can be created and used with tools like evilginx.
Currently, all found IP subnets returned by [ASN-2-IP](https://github.com/aalex954/ASN-2-IP) could be fed into the evilginx blacklist feature prevent EOP or Safe Links from identifying a phishing email or campaign.
In fact, if the blacklist redirect url was set to the same web site that was used to categorize the phishing domain it would obfuscate the phishing link even further.

## Future Goals

EOP Honeypot

1. Continuously send email to an owned Outlook and M365 mailbox.
2. Fingerprint EOP and Safe Links when they attempt to detonate the included link.
3. Identify EOP signatures.
4. Return a list of IPs associated with EOP and Safe Links.

Real-Time EOP IP Tracking

1. Continuously maintain a published list of IPs associated with EOP and Safe Links.
2. Should be cloud hosted and automatic.

Incorporate Findings into Evilginx TTPs

## Associated Projects

### [HTTP-Fingerprinting-Server](https://github.com/aalex954/HTTP-Fingerprinting-Server)

Simple Python web server for HTTP request and browser fingerprinting with whitelist and callback functionality.

### [ASN-2-IP](https://github.com/aalex954/ASN-2-IP)

Fetches AS Numbers associated with the specified organization, retrieves the IP prefixes for each AS Number, and writes the deduplicated IP prefixes to a file.

### [MSFT-IP-Tracker](https://github.com/aalex954/MSFT-IP-Tracker)

Tracks a range of Microsoft owned ASNs and publishes a daily release containing a list of IPv4 and IPv6 address in CIDR notation.
