# organize
Description: Space for me to record my effort to get more organized (with automation!)

## What is This?
Right now it's just a place where I'm keeping track of ideas about orgizing my mind and my work.  ADHD is hard.  Aging with ADHD is the first circle of hell.  I'm taking this moment of clarity to create a roadmap for implementing visual queues and documentaiton of my important work.

## Some Ideas
As far as I can tell, if I'm going to continue to thrive (survive?) in the tech space there are some essential things that need to be squared away.  To me, they seem to be:

### 1. Storage
This comes in two flavors: local, remote.  Detail about this can be found in storage.adoc.

#### Local Storage
To me this is priority number one, because I live in a relatively safe place and I'm confident that something disasterous is unlikely to happen.  For this, I need a good NAS.  My trusty Synology DS918+ has served the role for half a decade, but I need more control over my storage appliance to enable automations that aren't easy to do on a Synology.  

- Problem: The need for local storage I can integrate with other automations.

Requirements:  
1. 2+ GbE (or better) interfaces.  1 for public, 1 for storage
2. iSCSI volumes
3. SAMBA shares
4. NFS shares
5. ~~AoE volumes (need a couple)~~
6. Data checksumming (b/c background radiation and magnetosphere flux)
7. About 4 TiB of total storage space
8. A means of exporting this data to a cartridge system for GTFO situations (natural disasters, civil/political unrest, quarantine zoning, etc)
- Options: 
  * (1) A DIY 4-drive ZFS array via TrueNAS SCALE
  * (2) A DIY 4-drive Btrfs single volume over a thick provisioned LVM2 RAID6

Option 1: Has the advantage of more effectively safe-guarding data, native support for file and block storage provisioning, and works well with VM's.  It has the disadvantage of requiring additional fast flash storage to compensate for performance issues. 

Option 2: Has the advantage of being cheaper (will work with an RPi3) to impliment and is already familiar.  It has the disadvantage of needing special considerations for databases and virtual machines and requires more steps to recover data in the event of corruption as well as having two layers of file system that need scrubbing (LVM2 and Btrfs)

TODO: Need to address how to get the data off the device.  Perhaps a rotating disk in a protective case will suffice.  Should have a copy of storage device config and a script to automatically restore configs and data to a (new?) storage device in aftermath of a disaster.  Needs investigation.

#### Remote Storage
A cloud copy of your data is essential to ensure disaster recovery in the event that the original infrastructure is unsuable.  For this, it's important to make sure the correct data is being backed up.  In general, if you value it enough to put it on a dedicated local appliance, it should be remote as well.  I'll go with that for now.

- Problem: I need to get data here, elsewhere for safe keeping

Requirements: 
1. A non-trivial amount of data with a meager budget
2. Client-side encrytion of that data
3. Delta transfer
- Solution: rclone encrypted share to rclone sync to Wasabi is the cheapest option I've found so far.  My local Kallithea instance is a great example of this working.

### 2. Services
Services also seem to come in two flavors: virtual machines and containers.  The latter are easier to automate so that will be my focus.  In order to solve this problem, I need a container host.  For my home needs, Docker Swarm is sufficient, but because I need to stay up-to-date with OpenShift for work, ~~Open Kubernetes Distribution will be my poison of choice~~ Kubernetes will be my platform of choice since OpenShift relies on Java and I hate Java.  For this, I need several servers and this is where the virtual machines come in.

For more detail on the virtual machine setup, please refer to virtual_machines.adoc

- Problem: Need several servers for OKD, have room for 1 physical machine due to economics and downsizing of my workspace

Requirements:
1. Needs to be able to run a minimum of 7 VM's, no less than 4GiB of RAM each.
2. Each VM needs a minimum of 4 vCPU and I will NOT overprovision by more than 2x so 12-16 threads
3. 2+ GbE (or better) interfaces; 1 for public lan, 1 for storage
4. Hypervisor should support live migration so physical host expansion is possible
- Solution: Us my current EPYC 7401P workstation w/ ~~Proxmox VE~~ XCP-ng.  Local storage on this machine is not critical.  iSCSI is inefficient and bandwith is limited.  ~~I suggest creating an AoE share (large file on dedicated dataset) on a the NAS/SAN storage appliance and allow the hypervisor to treat it as an LVM pool.~~  However, given that XCP-ng is derived from a Red Had dsitro base, and Red Hat has never given two licks about ATA-over-Ethernet--and I don't have the equipment for FCoE, iSCSI it is.

#### Which Services, and in What Order?
1. MariaDB, PostgreSQL, ArangoDB
2. Source Repository (Distributed-capable, so probably something utilizing mercurial)
3. Automated backup (local and cloud) and archival to optical disc
4. Kanboard
5. Calendaring - ? 
6. Ticketing - Roundup
7. Project Management Dashboard - ... An API driven dashboard which gets all of its data (as requested) from the sources above
8. Centralized Authorization and Authentication

##### Some thoughts about the above...
I figure each component should be very Unixy.  In other words, it does its one thing and exposes an API which can be used to present all the data in a single pain of glass, hopefully allows for monitoring (alerts and such) and modifying the content and state of work as well.

