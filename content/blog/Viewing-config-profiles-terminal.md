---
title: "Viewing Configuration Profiles in Terminal"
date: 2019-05-09T13:53:36-04:00
draft: false
tags: ["jamf", "bash", "terminal", "configuration profiles"]
description: "Viewing Configuration Profiles in Terminal"
meta_img: "/img/configterminal.png"
author: "Kyle LeBlanc"
---

Often times I find myself wanting to view a print out of currently installed configuration profiles on a machine, but either can't or don't want to use the GUI. This would normally consist of going to `System Preferences > Profiles` to view a list of all profiles installed.

This method is useless if I am ssh'd into a box or am running a script remotely, etc.

You can run `profiles -C` in terminal to get a basic output of your profiles but if they were created using Jamf, then they're not named in a very human-readable format.

```bash
Kyle:~ kleblanc$ profiles -C
_computerlevel[1] attribute: profileIdentifier: 000000-0000-0000-A000-4A414D460003
_computerlevel[2] attribute: profileIdentifier: AD8AB2-B036-43CD-8C76-E9F0CC288495
_computerlevel[3] attribute: profileIdentifier: com.jamfsoftware.tcc.management
_computerlevel[4] attribute: profileIdentifier: 7FCBA6-B939-45B3-AEDF-8CB51F7F362C
_computerlevel[7] attribute: profileIdentifier: com.org.tccprofile
_computerlevel[8] attribute: profileIdentifier: E55CEA-FABB-4D67-9B93-186FA1DE0FE5
```

As you can see above, only 2 of the installed profiles spit out a identifiable name since these were created using manually and proper names were assigned. Profiles created in Jamf are given a random string identifier.

Now, if you print out the above command in a verbose mode, `profiles -C -v` you get more data than we need for this:

```bash
_computerlevel[4] attribute: name: MDM Profile
_computerlevel[4] attribute: configurationDescription: MDM Profile
_computerlevel[4] attribute: installationDate: 2018-11-12 19:33:46 +0000
_computerlevel[4] attribute: organization: PatientPing
_computerlevel[4] attribute: profileIdentifier: 0000-0000-0000-A000-4A414D460003
_computerlevel[4] attribute: profileUUID: 00000000-0000-0000-A000-4A414D460003
_computerlevel[4] attribute: profileType: Configuration
_computerlevel[4] attribute: removalDisallowed: FALSE
_computerlevel[4] attribute: version: 1
_computerlevel[4] attribute: containsComputerItems: TRUE
_computerlevel[4] attribute: internaldata: TRUE
_computerlevel[4] payload count = 3
_computerlevel[4]            payload[1] JAMF Manual Enrollment Payload: MDM
_computerlevel[4]            payload[1] desc. Config to connect to your Server
_computerlevel[4]            payload[1] type	= com.apple.mdm
_computerlevel[4]            payload[1] organization		= JAMF Software
_computerlevel[4]            payload[1] identifier= 000000-0000-A000-4A414D460004
_computerlevel[4]            payload[1] uuid = 0000-0000-0000-A000-4A414D460004
_computerlevel[4]            payload[2] name	= CN JSS Certificate Authority
_computerlevel[4]            payload[2] description	= A Root CA Certificate
_computerlevel[4]            payload[2] type	= com.apple.security
_computerlevel[4]            payload[2] organization		= Org
_computerlevel[4]            payload[2] id 2B5410-C803-46D3-A403-265C558520C4
_computerlevel[4]            payload[2] uuid = 2310-C803-46D3-A403-265C558520C4
_computerlevel[4]            payload[3] name	= JAMF
_computerlevel[4]            payload[3] desc Config to connect to your Server
_computerlevel[4]            payload[3] type com.apple.security
_computerlevel[4]            payload[3] organization		= Org
_computerlevel[4]            payload[3] identifier = com.jamfsoftware
_computerlevel[4]            payload[3] uuid = 0000-0000-0000-A000-4A414D460006
```

The above is just a snippet of the verbose command showing the output for 1 (one) single configuration profile, now imagine having 10+ installed and getting all of that output.

You can however see the first line of this output contains `attribute: name: MDM Profile`. This is what we want to capture. You can cut this output in half by just piping the output to grep and looking for attribute: `profiles -C -v | grep attribute`.

To take it one step further and make the output look really readable, we can pipe it a couple more times to do some string manipulation:

`profiles -C -v | grep attribute | awk '/name/{$1=$2=$3=""; print $0}' | sed 's/^ *//'`

the above will provide an output like:

```bash
Browser Settings
Default Security Policy
Privacy Preferences Policy Control
Approved Kernel Extensions
Office WiFi
TCC Whitelist
FV Key Escrow [10.13+]
MDM Profile
```

I love one-liners and I found this very useful to put into policies, extension attributes, and to just run when ssh'd into boxes that may not be reporting back to the JSS.
