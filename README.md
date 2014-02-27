xsibackup
=========
XSIBACKUP 2.1.2 Automated Backups for ESXi 5.X

 Copyright (C) 2013  33HOPS, Sistemas de Información y Redes, S.L.

    This program is free software: you can redistribute it and/or modify
    it under the terms of the GNU General Public License as published by
    the Free Software Foundation, either version 3 of the License, or
    (at your option) any later version.

    This program is distributed in the hope that it will be useful,
    but WITHOUT ANY WARRANTY; without even the implied warranty of
    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
    GNU General Public License for more details.

    You should have received a copy of the GNU General Public License
    along with this program.  If not, see <http://www.gnu.org/licenses/>.


IMPORTANT:

If you receive any arror and want to contact us please provide the argument list as parsed
to XSIBackup otherwise we cannot figure out what's going wrong.

XSIBackup is compatible with ESXi 5.1.0 and ESXi 5.5.0, pressumably it is also compatible
with ESXi version 5.0.0 but this has not been fully tested. It has not been tested at all
with versions prior to ESXi 5, in any case if you have time you can give it a try and
let us know how it went.

You can use any character in the variable values
with the only exception of the equal sign (=) and doublequote ("). If you use an equal sign
in a variable value the variable will be truncated at this symbol and if you use a doublequote
you will get this error:

root# -sh: syntax error: bad function name

Changes (since former version 2.1.1), current 2.1.2:

Now the subject of the e-mail is settable, --subject="Your very own subject"
Now you can use physical paths for the --backup-point variable instead of the path using the
mount point of the datastore, i.e. /vmfs/volumes/52bc5283-1db14cfc-90cc-94de80136f37/BACKUP
as well as /vmfs/volumes/datastore2/BACKUP so both ways are supported.

LIMITATIONS:

We have cleared most of XSIBACKUP's limitations inherited from the original project that were:
not being able to manage paths with spaces in between (*) and deleting all snapshots before doing
a hot backup operation. From version 2.1.0 XSIBACKUP should be able to backup any existing
ESXi 5.X box preserving all existing snapshots and no matter how it was configured.

(*)In any case, before naming your virtual machines and directory structure ask yourself if
you really need them to have interstitial spaces as it will normally not give you any advantage
but can give you lots of headaches at the time of working with the inners. The most advisable
way of naming your VMs is with a fixed lenth code where every part of the code has its meaning,
per instance WX001, W7002, LR003 standing for Windows XP #1, Windows 7 #2 and Linux Red Hat #3
respectively. This way you can programatically process the VM names in case you need it in a 
much more convenient way. 

FACTS:

In order to allow outgoing e-mail comunicacions through the configured SMTP port XSIBackup
will open the firewall by adding a new service named SMTPout and an associated outgoing 
rule. This is far more convenient than having to open it up manually and does not affect
security as the port is opened just before sending the e-mails and is closed right after.

You have to copy xsibackup to a persistent path in your system like /vmfs/volumes/datastore1 
also do make sure it has the necesary rights by issuing the following command from the same 
directory

chmod 0700 xsibackup

Then you can execute it by issuing

./xsibackup from the same directory

Or

/full/path/to/xsibackup

Of course if you execute it without any argument it will simply print out the help.

DESCRIPTION

© XSIBackup uses the ESXi built in command line options to create fully unattended 
backup solutions, this means you can create a backup schema that will for example 
backup all your running virtual machines every night, send you a detailed e-mail 
report after each backup operation and provision space once the backup disk is full
by deleting the older backup folders. Only folders with the name format used by 
© XSIBackup will be deleted to provision space so older backups can coexist in the same 
disk by simply renaming the folder. © XSIBackup has built-in capabilities for e-mail 
submission through AUTH PLAIN authentication schema since version 1.0.3. 

You can choose to carry out a "hot backup" (while the virtual machines are running) or a
"cold backup" (switches off every virtual machine before copying it) by setting the option
--backup-how=hot(default)|cold. If you choose "cold" © XSIBackup will issue a shutdown 
to the virtual machine instance and wait 30 seconds for the virtual machine to be shutdown 
cleanly. If after the initial 30 seconds period the VM continues to be on the program will 
wait 30 more seconds checking the VM state every 10 seconds after which © XSIBackup will 
consider the virtual machine can't be shutdown and it will be powered off. If the VM does 
not have © VMware Tools installed on it © XSIBackup will simply power it off.

Please note that when a VM can't be shutdown cleanly most of the times it is a program or 
service that keeps it from being shut down the right way. Powering off is the last recurse
for © XSIBackup to be able to turn off the VM and back it up. If you see VMs that were
powered off before backup in the reports please check the operating system and fix whatever
is wrong (of course check that you have the last version of VMware Tools installed in every 
VM), most of the times nothing serious will happen but in certain circumstances you could 
lose data by forcing a power off.


MANUAL

INSTALL:

First of all you need to allow SSH connections to your ESXi box, you can acomplish this by
using the vSphere Client. Click on the server node and go to the Configuration tab, then
look for the Security Profile entry in the left side menu under the "Software" heading and
click on it. You will see the firewall entries on the right. Click on the Properties link
on the right and checkmark the SSH server and SSH client entries, then reboot the server. 

Once you have reboot the server simply copy xsibackup file to your /vmfs/volumes/datastore1
folder by using any SSH/SCP client. You can use WinSCP at http://winscp.net/eng/download.php 
or any Linux/Unix command line like this (from the same directory where your downloaded
xsibackup file is) linux# scp xsibackup root@[yor esxi IP]:/vmfs/volumes/datastore1

The easy way:

Cut & paste the following line in your ESXi command line and press enter. You can get this URL by
clicking the right mouse button and choosing "copy link url" in many browsers.

wget http://sourceforge.net/projects/xsibackup/files/xsibackup_2.1.2.zip/download -O xsibackup.zip

This will download version 2.1.1 to the file xsibackup.zip in your local directory structure. You
can then unzip it

unzip xsibackup.zip

Asign the apropiate permissions

root# chmod 0700 xsibackup

And do a quick test by executing the script without any arguments

root# ./xsibackup

If you see the help then everything went right and you can start using xsibackup.

USAGE:

Example 1 (check the process and the e-mail submission before a hot backup):
xsibackup --backup-point=/vmfs/volumes/backup --backup-type=running --mail-from=email.sender@yourdomain.com 
--mail-to=email.recipient@anotherdomain.com --smtp-srv=smtp.33hops.com --smtp-port=25 --smtp-usr=username 
--smtp-pwd=password --test-mode=true

Example 2 (backup all running VMs while on, --backup-how parameter is omited as hot backup is the default):
xsibackup --backup-point=/vmfs/volumes/backup --backup-type=running --mail-from=email.sender@yourdomain.com 
--mail-to=email.recipient@anotherdomain.com --smtp-srv=smtp.33hops.com --smtp-port=25 --smtp-usr=username --smtp-pwd=password

Example 3 (cold backup 2 given VMs):
xsibackup --backup-point=/vmfs/volumes/backup --backup-how=cold --backup-type=custom --backup-vms=WINDOWSVM1,LINUXVM2 
--mail-from=email.sender@yourdomain.com --mail-to=email.recipient@anotherdomain.com --smtp-srv=smtp.33hops.com:25 
--smtp-port=25 --smtp-usr=username --smtp-pwd=password

Example 4 (hot backup 3 given VMs):
xsibackup --backup-point=/vmfs/volumes/backup --backup-type=custom --backup-vms="WINDOWSVM1,LINUXVM2,New Virtual Machine"
--mail-from=email.sender@yourdomain.com --mail-to=email.recipient@anotherdomain.com --smtp-srv=smtp.33hops.com:25 
--smtp-port=25 --smtp-usr=username --smtp-pwd=password

OPTIONS:

--backup-point

Full path to the backup mount point in the local server where it will tipically be under 
/vmfs/volumes, i.e. /vmfs/volumes/backup.
You must use the mount point of local devices when passing it to the --backup-point variable
real pysical paths are not supported!!!

--backup-how (hot | cold)

Hot backup is the default method, it makes a backup while the VM is on. You can chose to make a cold backup and the VM will be cleanly shutdown befor backup and switched on right after.

--backup-type (custom | all | running)

Custom: if this methos is chosen then a list of the VMs to backup must be passed to the --backup-vms option.
All: backup -all- VMS.
Running: backup only running virtual machines. They will be cleanly shutdown and backed up then switched on again.

--backup-vms

List of virtual machines to backup as a colon separated list, i.e: --backup-vms=VM1,VM2,VM3, only needed if "custom" is selected as the --backup-type

--mail-from

E-mail address as from where the HTML e-mail report will be sent.

--mail-to

E-mail address to which the HTML e-mail report will be sent.

--subject

Subject of the email to send, per instance --subject="Your very own subject"

--smtp-srv

SMTP server that we will use to send the HTML e-mail report through.

--smtp-port

SMTP server port used for communication

--smtp-usr

SMTP username we will use in the plain text SMTP authentication. Please note this is the only authentication method 
supported by esxbackup by now.

--smtp-pwd

SMTP password used for authentication against the SMTP server.

--test-mode=true

Perform a general test without backing up the selected virtual machines. You can try your configuration and the e-mail 
receipt before putting it into production.


CRON SETUP

NOTICE: Soon you will be able to setup the cron job from XSIBackup (scheduled for v 2.1.0 nov-2013) but by now...

Now that you have played around with XSIBackup and tested it from the command line you will probably want to program a cron job under ESXi 
to get your virtual machines backed up automatically every night. To acomplish this you will need to do the following:

1 - Edit /etc/rc.local.d/local.sh, the system's startup script to get the cron service pid and to kill the process before writting anything to the crontab.

2 - Edit /etc/rc.local.d/local.sh, the system's startup script to inject the cron line you will need into the ESXi 5.X root's crontab located at /var/spool/cron/crontabs/root

3 - Start the cron service again.

To acomplish this do the folowing:

root# vi /etc/rc.local.d/local.sh

At the end of the file right before the "exit 0" statement write the following:

/bin/kill $(cat /var/run/crond.pid) # Gets the cron service pid and simply kills it.

The next line writes a tipical cron line to the crontab, in this example we execute "xsibackup" passing it some arguments to backup all running virtual machines to /vmfs/volumes/backup
storing a log to /vmfs/volumes/datastore1/backup.log and adding the cron line to /var/spool/cron/crontabs/root

/bin/echo "5 0 * * * /vmfs/volumes/datastore1/xsibackup --backup-point=/vmfs/volumes/backup --backup-type=running 
--mail-from=fromuser@yourdomain.com --mail-to=recipient@yourdomain.com --smtp-srv=smtp.yourserver.com --smtp-port=25 
--smtp-usr=yourusername --smtp-pwd=yourpassword > /vmfs/volumes/datastore1/backup.log 2>&1" >> /var/spool/cron/crontabs/root

Finally we start the cron service again
# /usr/lib/vmware/busybox/bin/busybox crond
