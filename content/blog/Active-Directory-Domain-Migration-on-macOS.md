---
title: "Active Directory Domain Migration on MacOS"
date: 2019-05-03T11:28:26-04:00
draft: false
tags: ["active directory", "jamf"]
description: "Migrating AD domains on macOS"
meta_img: ""
author: "Kyle LeBlanc"
---

Joining Windows machines to an AD domain is easy, intuitive, and downright simple. The same can't really be said for machines running macOS. The task of binding these devices to the same AD world that your Windows machines are on can be tricky and cumbersome, but this is becoming all the more common as macOS is taking up a larger seat in the enterprise/corporate world.

So we know how finicky binding macOS to AD is, now imagine automating the process to swap from one AD to a new one, on almost 1,400 macOS machines.

TL;DR version: I was able to automate this process by scripting the following sequence of events and then deploying this with Jamf to our endpoints: Setup a log file, alert user, gather current AD/user info, remove user form FileVault, unbind the machine from old AD domain, bind to new AD domain, alert admin via email on success/failure, reset local user permissions, remove any local AD cache files, perform authenticated reboot, allow the user to change their password for the new domain, launch a login script to add the user back into FileVault.

That is a mouthful. Here is a closer look at how this was done:

First, here is a little background on the environment in which this was done. All machines running macOS were enrolled in our JSS, bound to an AD domain, had FileVault enabled with the only two users allowed to unlock the disk being the assigned user and our administrator account. All machines needed to be on premise (office location did not matter, as we had offices/macOS machines distributed globally) but our AD domain was not accessible outside of our enterprise network.

I never like to surprise any user that I am working with, let alone with a delicate task like this at hand. The first part of my script was to alert the user that this process was beginning. This was done using the pre-installed Jamf Helper that is installed with enrollment to the JSS. This provided a nice alert (company branding included) that the user could trust and know it was from IT. Once that was done it was time to dig into the fun stuff.

![User Notification](/img/usernotif.png)

I created a log file in `/Library/Logs/` to keep tabs in case anything had gone wrong. This was invaluable when troubleshooting with test machines and making sure the process was locked down as well as if there were any hiccups when deploying this on production machines.

I had passed in a handful of variables from the JSS policy that this script was attached to including the old AD domain name, the new AD domain name, the administrator email address to send alerts to, as well as a binding accounts credentials for the old AD domain. I still need some additional variables to get the ball rolling. Moving forward I collected the current OS version, the current domain the machine was bound to, the machine name, currently logged in username, their UID, and their home directory.

Once I had all the pieces to this puzzle it was time to start moving them around. The first step to perform is removing the user from FileVault. Allowing the user to remain in FileVault during this process had caused a number of issues when testing; primarily around permissions, logging in, and new password changes.

At this point in the process, we are in a pretty good state to start throwing some stuff around. Unbinding the machine from the old domain is a pretty simple process. I like to double check my work so I decided to put this in an if condition so that if for some reason we were already on the new domain (or not on a domain at all) this wouldn't run (I apply this check again in one of the next steps). If the statement returns true, that the current domain = the old domain, then I call `dsconfigad -remove` using the earlier fetched credentials/domain info to pull this machine off the old domain.

The machine is now in a clean state to begin binding to the new domain. The actual binding is pretty short and is actually done in a separate Jamf policy (called in this script used the `policy -event` trigger). The main reason for doing this was for consistency and to obscure the new domain binding account credentials. Seeing as this script is cached on the machine before being run, I wanted to make is as secure as possible. Of course, we didn't want to continue up a creek without a paddle so an additional check was put in place at this point to ensure the bind was successful. I fetched the now current AD and compare to what the new one should be, if we don't have a match then an email was sent to the earlier supplied email address with the current machine name and then the statement would exit the script halt the process. I originally had the if statement send an email on a successful bind but these alerts turned out to be extremely noisy and non-actionable so I turned those off. I really only needed to be notified of failures.

Now that we are on the new domain (yay!) it was time to adjust some permissions. I had to fetch the following for the users account on the new domain and saving each one into an appropriate variable: UID, Primary GID, and the new AD Node Name. I then swapped the old UID for the new UID for the logged in user, as well as changed the users home directory permissions to use the new UID and GID.

Before I sent the machine off to do a restart we had to clean a few things up. This included removing any `sqlindex` files from `/var/db/dslocal/nodes/Default/` and renaming the logged in users .plist file from loggedInUser.plist -> loggedInUser.plist.OLD in `/var/db/dslocal/nodes/Default/uers/`. These were two tricky and critical steps. It's possible to even say this was the crux of the entire migration process. Without removing these files, and even though the machine is technically bound to the new domain, the user would never get prompted for a password change and their old domain password would still work. It was critical that the user changed their password on login to the new domain to prevent issues with other AD connected services (SSO, Exchange, etc).

At this point, we are pretty clear to restart the machine. A simple `shutdown -r now` wasn't really up to par for this course. Seeing as all of our machines were FileVault enabled, and we had to remove the user from FileVault to perform some permission/password changes, we couldn't just reboot. This would lock the user out of their machine and would cause a ticket/walk-up on an otherwise successful migration.

To get around that issue, I was able to utilize the `fdesetup authrestart` command. By default, this requires user input for a FileVault username and accompanying password. However, if you echo this information to a .plist file for the user then you can bypass user input altogether. I dumped an echo of some xml including the necessary password to unlock FileVault into `/tmp/authrestart.plist` and finally initiated the start with `fdesetup authrestart -inputplist < /tmp/authrestart.plist`.

From here it's pretty much all wrapped up. The remaining steps take place after the machine comes back up. The user will be prompted to change their password, creating new caches for the new domain. This will cause the login trigger to fire in the JSS and proceed with a login script which will add the user back into FileVault by prompting for their new password, and finally, it will clean itself up by removing the migration script once and for all.

Although this seems like a lengthy process (and creating it was very lengthy), it really only takes no more than ~3 minutes to complete on the user machine, reboot included. I learned a lot about how macOS interacts with Active Directory and some lower level macOS operations. When deploying this out to production machines I did run into a couple of hiccups here and there, primarily with the reboots. For a reason that is still unclear to me, there would be specific circumstances when the authrestart would not fire and would require us to have someone from Help Desk reboot the machine with either the recovery key or admin credentials.

Now that everything is said and done I am proud to say that this project was an overall success. I take a lot of pride in completing this, especially when finding information on this task was pretty scarce on the internet. If you've made it this far I really appreciate you taking the time to read this, and if you are going through a similar project and have any questions please reach out!
