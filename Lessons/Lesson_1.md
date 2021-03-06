-- *Slide* --
# Introduction
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/spartanlogo.png" />
-- *Slide End* --

-- *Slide* --
## Outline for Today
* Part 1: Architecture
* Part 2: Configuration
* Part 3: Accounts
* Part 4: Scheduler
* Lunch
* Part 5: Operations
* Part 6: Outreach
-- *Slide End* --

-- *Slide* --
## A Team-Building Exercise
Not every group that works together is a team. A team is a group that shares common goals, are mutually accountable for delivering goals, are interdependent, possess complementary knowledge and skills, interact effectively, and see integration with each other as a responsibility. As a team, we have a purpose - to build, maintain, and develop the Spartan HPC system and the new GPGPU partition. We have specific members with roles, we have resources. 
-- *Slide End* --

-- *Slide* --
## A Team-Building Exercise (cont..)
The people involved in Spartan support and development have a great diversity of skills, in breadth and depth, and will dovetail with a mentoring program. This workshop is to help everyone get a 'big picture' view of the system, to raise issues of concern and knowledge, to make suggestions to improve Spartan, and our delivery of the service.
-- *Slide End* --

-- *Slide* --
## Slack
The NeCTAR research cloud runs a Slack service for synchronous communication between groups and individuals. The main relevant channel is `#uom-hpc` and `#uom-ops`. Slack is useful for providing alerts from humans that require a quick intervention from others, or to report on active task activities.
`https://researchcloud.slack.com/`   
-- *Slide End* --

-- *Slide* --
## Documentation
* User documentation starts with https://dashboard.hpc.unimelb.edu.au. This is a git repository, located in `https://github.com/resbaz/spartan-hpc-docs/`  
* We also have `man spartan`, the examples in `/usr/local/common`, and the training courses. `https://github.com/UoM-ResPlat-DevOps/SpartanIntro`, `https://github.com/UoM-ResPlat-DevOps/HPCshells`, `https://github.com/UoM-ResPlat-DevOps/SpartanParallel`   
-- *Slide End* --

-- *Slide* --
## Documentation (cont..)
Most HPC operations information is in the ops-doc wiki `https://github.com/UoM-ResPlat-DevOps/ops-doc/wiki`, and software build information is at `https://github.com/UoM-ResPlat-DevOps/HPC` 
This course is stored at `https://github.com/UoM-ResPlat-DevOps/SysAdminCourse`   
-- *Slide End* --

-- *Slide* --
## Documentation (cont..)
There is some documentation at `https://unimelbcloud.sharepoint.com/teams/ResearchComputeServices/Shared%20Documents/`   
Important short notices can be added to the Message-of-the-Day at `/etc/my_motd`
-- *Slide End* --

-- *Slide* --
# Architecture
-- *Slide End* --

-- *Slide* --
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/spartanlayout.png" width="150%" height="150%" />
-- *Slide End* --


-- *Slide* --
## Nodes
Spartan is as a hybrid HPC and cloud compute system that is orientated towards maximising throughput in a cost-efficient manner. There is a smaller traditional HPC partition, a substantial GPGPU partition, and a larger partition of virtual machines from the Melbourne node of the NeCTAR research cloud. This matches the dominance of single-node jobs that were submitted on Spartan's predecessor, Edward. As typical with other HPC systems, it has a management node and two login nodes.
-- *Slide End* --


-- *Slide* --
## Nodes (cont..)
"Bare metal" HPC consists of is 44 nodes, 21 GB per core. 2 socket Intel E5-2643 v3 E5-2643, 3.4GHz CPU with 6-core per socket, 192GB memory, 2x 1.2TB SAS drives, 2x 40GbE network. There are currently 68 GPGU nodes dual socket 24 core E5-2650 v4 at 2.20GHz, with 4 Telsa P100s with 128 GB. There is currently 5 GPU nodes, 12xE5-2643 v3 at 3.40GHz with 251 GB and with 4 Tesla K80s. There is also 283 cloud virtual machines with 8 2.3GHz Haswell cores with 8GB per core.
-- *Slide End* --

-- *Slide* --
## Network
The cloud nodes are connected via Cisco Nexus 10Gbe TCP/IP 60 usec latency (mpi- pingpong); and the bare metal Mellanox 2100 Cumulus (mostly v3.5.3) Linux 40Gbe with TCP/IP 6.85 usec latency and then RDMA Ethernet 1.15 usec latency. Later was superior to Infiniband FD14 (1.17 usec). Four different VLANs are used for communication within the cluster (IPMI baremetal), build and management (baremetal), main cluster traffic, and RoCE traffic.
-- *Slide End* --

-- *Slide* --
## Network (cont...)
The new GPGPUs have six racks requiring network for data, inband communication and out-of-band communication, with 14 nodes per rack, requiring 168 data ports, 84 x 1G inband ports and 84 x 1G out-of-band ports with Mellanox ConnectX direct connect cards The network connection speed will be to 2 x 50Gbps (total of 100Gbps) connections per node, all connected to a Mellanox SN2100 switch for data, and more generic switches for inband and out-of-band.
-- *Slide End* --

-- *Slide* --
## Storage
There are mountpoints to home, projects (/project /home for user data & scripts, NetApp SAS aggregate 70TB usable in /data/projects) and applications directories across all nodes, being replaced with CephFS using Near-Line SAS. Applications directory currently on management node; was a virtual machine, now physical.  
-- *Slide End* --

-- *Slide* --
## Storage (cont..)
Bare metal nodes have /scratch shared storage for MPI jobs (Dell R730 with 14 x 800GB mixed use SSDs providing 8TB of usable storage, NFS over RDMA)., /var/local/tmp for single node jobs, pcie SSD 1.6TB. 
-- *Slide End* --

-- *Slide* --
## Storage (cont..)
```
(vSpartan) [root@spartan-build vSpartan]# df -H
Filesystem                                 Size  Used Avail Use% Mounted on
...
/dev/sdb1                                  8.8T  2.2T  6.7T  24% /usr/local
/dev/sda2                                  518M  165M  354M  32% /boot
spartan-stg2.hpc.unimelb.edu.au:/projects   70T   66T  3.7T  95% /data/projects
tmpfs                                      6.8G     0  6.8G   0% /run/user/0
ceph-fuse                                  6.0T  5.1T  950G  85% /home
ceph-fuse                                  429T  216T  214T  51% /scratch
ceph-fuse                                  500T  172T  329T  35% /data/cephfs
```
-- *Slide End* --

-- *Slide* --
## Storage (cont..)
The default allocation for projects is 500GB, and 50GB for /home directories. A little bit of room for variation on the former. The Project storage is currently divided up into a 70TB mount on NetApp and is currently being moved to CephFS. There is 429TB of high-performance scratch/. The application directory is on `/usr/local` is also mounted across all nodes.
-- *Slide End* --

-- *Slide* --
## Operating Systems
Like nearly all other HPC centres in the world, Spartan uses Linux and specifically the Red Hat distribution for its operating system; RHEL 7.* is used for all nodes, are we're subscribed to Unimelb Red Hat Satellite Server `https://sputnik.its.unimelb.edu.au` 
-- *Slide End* --

-- *Slide* --
## NeCTAR Research Cloud
There is also significant use of OpenStack's Nova (compute) service for provisioning and decommissioning of virtual machines on demand, which includes the management and login nodes. Specifically Spartan makes use of the following from the NeCTAR research cloud.
-- *Slide End* --

-- *Slide* --
## NeCTAR Research Cloud (cont..)
Cell: melbourne-qh2-uom   
Tenant: vSpartan f31a36cc4dec4727a70b87917936f73e   
Instance flavor: mel.hpc.8c64g (8 vcpus, 64gb of ram, 30gb root, no ephemeral)   
OS image: os image list --private - usually the latest spartan-rc-gold-YYYYMMDD   
-- *Slide End* --

-- *Slide* --
## DNS
DNS is managed through `https://ns-q.melbourne.nectar.org.au:9000`. The reverse IP entry ${ip_address} IN PTR vm-${ip_address}.melbourne.nectar.org.au must be deleted otherwise ssh hostbased authentication will not work.
-- *Slide End* --

-- *Slide* --
## Monitoring
Our monitoring system is conducted through the NeCATR monitoring services (`https://mon.rc.nectar.org.au/`), which for Spartan includes PuppetBoard, Nagios, amd Grafana. In the near future will we also be implementing Xdmod.
-- *Slide End* --

-- *Slide* --
# Configuration
-- *Slide End* --

-- *Slide* --
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/longines1897.jpg" width="75%" height="75%" />
-- *Slide End* --

-- *Slide* --
## Spartan's Configuration Files
Spartan's configuration files are stored on a the NeCTAR Git repository as Puppet files. Then they are reviewed through Gerrit and Jenkins. The combination of these features ensures sanity checking and creates "paired systems administration".
-- *Slide End* --

-- *Slide* --
Setup a NeCTAR Gerrit Account with either Launchpad, Yahoo!, or OpenID.
`https://review.rc.nectar.org.au/login/`
To clone the puppet repository for Spartan's configuration, see the instructions here:
`https://github.com/UoM-ResPlat-DevOps/ops-doc/wiki/Spartan-Git-Repos`
Make changes locally, and commit.. `$ git add`, `git commit`, `git review`. 
-- *Slide End* --

-- *Slide* --
## Review Process on Gerrit
Gerrit is a web-based code collaboration tool. Whilst usually used by software developers, we also use it for configuration review. Modifications to the configuration code can be viewed on web browser and approve or reject those changes. It has an interesting interface. Jenkins provides the integration.
e.g., `https://review.rc.nectar.org.au/#/c/12944/`
-- *Slide End* --

-- *Slide* --
# Accounts
-- *Slide End* --

-- *Slide* --
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/karaage.png" width="150%" height="150%" />
-- *Slide End* --

-- *Slide* --
## Karaage
Karaage is a cluster account management tool. It can manage users and projects in a cluster and can store the data in various backends and uses a Django framework. User information can be stored in LDAP, Active Directory, or passwd files. On Spartan we use a local LDAP. Karaage has an email notifcayion service, allows for the creation of accounts by project leaders, track software usage, and cluster utilisation (per institute, system, or project) by CPU hours.
-- *Slide End* --

-- *Slide* --
## Karaage (cont...)
Spartan's Karaage is located at `https://dashboard.hpc.unimelb.edu.au/karaage/`. This is also the site of the Spartan website. We are also considering shifting some of the Slurm database functions (at least as a backup) to this site as well.
-- *Slide End* --

-- *Slide* --
## Users
Users belong to accounts, and accounts belong to projects. When a user applies for an account and project this will trigger a pending user and project on Karaage, after it has been approved by the Head of Research Compute. It is worth checking to see if the project applicant has actually requested a cluster acocunt. Sometimes the user just forgets to tick the box. However, if you see a professor/manager applying, they actually might not need access to the clusters. Check with the users before ticking it for them.
-- *Slide End* --

-- *Slide* --
## Users (cont..)
Once project lead user has been approved, go to the newly created projects and click on Grant Access next to the user to make them the Project leader. This will allow us to hand off the management of the projects to the users. Then create a project directory. If a directory with project name in `/data/projects` (usually punimXXXX) has not been created: `cd /root/` `./mkproject.sh $project_name`
-- *Slide End* --

-- *Slide* --
## Users (cont..)
Users often have problems with logging in for a variety of reasons. Some forget their password (refer to `http://dashboard.hpc.unimelb.edu.au` or ServiceNow), or confuse their password with the University password. A failure to login several times will result in fail2ban triggering. A particular challenge arises when people wish to become users but have no SAML identity. In these cases, the account has to be manually created, the user assigned to the project, and their password reset.
-- *Slide End* --

-- *Slide* --
# Scheduler
-- *Slide End* --

-- *Slide* --
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/slurm.jpg" width="150%" height="150%" />
-- *Slide End* --

-- *Slide* --
## Slurm
The Slurm Workload Manager (formerly SLURM, Simple Linux Utility for Resource Management), is a free and open-source job scheduler for Linux and Unix-like systems and combines the functions of a job scheduler and resource manager. It is notable for its efficiency, scalability, reporting tools, and modular design with around 100 optional plugins.
-- *Slide End* --

-- *Slide* --
## Slurm (cont...)
Man pages exist for all Slurm daemons, commands, and API functions (c.f., `https://slurm.schedmd.com/man_index.html`). The Slurm daemons manage nodes, partitions, jobs. The partitions are effectively job queues with resource and user constraints. Jobs are allocated to nodes within a partition according to proirity.
-- *Slide End* --

-- *Slide* --
## Slurm (cont...)
The main daemon for slurm is `slurmctld` which coordinates the queueing of jobs, monitoring node states, and allocating resources to jobs. In addition there is a slurmd daemon running on each compute node - this also has to be restarted after a configuration change. 
-- *Slide End* --

-- *Slide* --
## Slurm (cont...)
The slurm configuration file is located at `/usr/local/slurm/etc/slurm.conf` on the management node. This includes various resource limits, partitions etc. If Slurm is reconfigured, run `/usr/local/resplat/sbin/slurm-run-jobs.sh` to bring all partitions back up.

The user commands include: `sacct`, `salloc`, `sattach`, `sbatch`, `sbcast`, `scancel`, `scontrol`, `sinfo`, `smap`, `squeue`, `srun`, `strigger` and `sview`. 
-- *Slide End* --

-- *Slide* --
## Partitions
Partitions (equivalent of queues) can be viewed with the `sinfo` command. Nodes can belong to multiple partitions simultaneously. For example, all nodes are in the `debug` partition. Detailed information about a partition and constraints can be viewed with `scontrol show partition`.
-- *Slide End* --

-- *Slide* --
## Partitions (cont...)
Spartan's main partitions are the `cloud` partition, the `physical` partition, and the `gpgpu` partition. As the names indicate the cloud partition consists of nodes that are virtual machines, where as the physical partition consists of nodes that are on bare metal, running with a higher-speed interconnect, and a partition for LIEF grant gpgpu projects. Of note are project specific partitions (e.g., punim0095), departmental partitions (e.g., `water`, `ashley`), and partitions with different architecture (e.g., `gpu`).
-- *Slide End* --

-- *Slide* --
## Job Submission
Like other scheduling systems Slurm requires that a user submit a batch request of resources, for a particular period of time. The resources include nodes, tasks, tasks-per-node, cpus-per-task, generic resources (e.g., GPUs). Submission is with the `sbatch jobname` command, or `sinteractive` for interactive jobs. Numerous example scripts are in the `/usr/local/common/` directory. Note in particular those in the `IntroLinux`, `HPCshells`, `array`, and `depend` directories.
-- *Slide End* --

-- *Slide* --
## Job Control
Users and administrators may cance a job with `scancel [jobid]`   
Users and administrators can retrieve detailed information about a job with: `scontrol show job [jobid]` or `jobscript [jobid]`
-- *Slide End* --

-- *Slide* --
## Job Control (cont..)
Administrators can specify the new walltime for a job. e.g., `scontrol update jobid=1042541 TimeLimit=30-0:0:0`   
We have a script for for checking bad jobs `check_bad_jobs.sh` and one for users `check_my_bad_jobs.sh`
-- *Slide End* --

-- *Slide* --
## Slurm Metrics
Managers often want usage metrics. The following sreport command provides examples of utilisation: e.g.,
`sreport -t Hours cluster Utilization start=2016-01-01 end=2016-11-10`   
`sreport cluster AccountUtilizationByUser cluster=spartan user=khalid start=2016-01-01 end=2017-01-17`
-- *Slide End* --

-- *Slide* --
## Deployment
To build a bare-metal or cloud node see: `https://github.com/UoM-ResPlat-DevOps/ops-doc/wiki/Build-Spartan-Nodes`   
If a node needs to be upgraded etc, set it for drain. Check the workload of non-cloud partitions beforehand (`sinfo -p $partition`), and jobs on the nodes (`spartan-b: ~/jobsonnode.sh $node` or `squeue -w $nodename`
-- *Slide End* --

-- *Slide* --
## Deployment (cont...)
Then run the drain command `scontrol update nodename=$nodes state=DRAIN reason="$reason"`. The node, if idle, will go to drain. If it it has jobs running it will be marked as `draining`. Node status can be checked with `downbecause`. To return drained nodes to production use `scontrol update nodename=$nodes state=RESUME`.
-- *Slide End* --

-- *Slide* --
## Deployment (cont..)
* You can stop Slurm from scheduling jobs on a per partition basis by setting that partition's state to DOWN. Set its state UP to resume scheduling. For example:
`$ scontrol update PartitionName=$partition State=DOWN`   
`$ scontrol update PartitionName=$partition State=UP`
-- *Slide End* --

-- *Slide* --
## Spartan Upgrades
When upgrading Slurm, SlurmDBD must be upgraded first, always. `https://slurm.schedmd.com/quickstart_admin.html#upgrade`. Detailed instructions on the Wiki `https://github.com/UoM-ResPlat-DevOps/ops-doc/wiki/Spartan-Upgrades`.   
We have a risk register for when significant failure results due to open bugs on our infrastructure. `https://github.com/UoM-ResPlat-DevOps/ops-doc/wiki/Risk-Register`
-- *Slide End* --

-- *Slide* --
# Operations
-- *Slide End* --

-- *Slide* --
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/operation.jpg" />
-- *Slide End* --

-- *Slide* --
## Project Workflow
Spartan uses Trello to manage project workflow through "cards" with associated users and content checklists. This is usually for multi-task objectives that are generated internally rather than from user requests. However sometimes user requests via the helpdesk can become Trello cards as well, if of sufficient complexity (the split between helpdesk tickets and Trello cards is not well defined).
-- *Slide End* --

-- *Slide* --
## Project Workflow (cont...)
To access the Spartan HPC Trello, signup at `https://trello.com/signup` and login at `https://trello.com/login` (Google account option). The page URL is located at: `https://trello.com/b/kvLgvEZa/hpc` with the following Menus; "To Do", "On Hold", "Doing", "Admin & Doco", and "Done".
-- *Slide End* --

-- *Slide* --
## Helpdesk
Spartan (currently) uses the NeCTAR Freshdesk system for managing user request tickets at `https://support.ehelp.edu.au/helpdesk/dashboard`. Our group is `UoM HPC`. General users are accredited by administrators and the existing current implementation has a national scope. In nearly all cases, ticket creation and responses are carried out by email as the single point of entry and communication.
-- *Slide End* --

-- *Slide* --
## Helpdesk (cont..)
Freshdesk comes with SOAP etc and several REST APIs (ticket, user, agent, companies, forum (several subcategories), solution (several subcategories), time entries, survey, groups), full text-search capability, built-in customer satisfaction surveys, and an extensive range of customer support contact methods (email, web, phone, instant messaging). Performance reports are automatically sent to the Head of Research Compute Services.
-- *Slide End* --

-- *Slide* --
## Helpdesk (cont..)
As a general workflow if you can respond to a ticket, assign it to yourself first, then respond. Avoid responding to tickets owned by another person, but if you have an idea or suggestion, send it through Slack to the owner or as a private note on the ticket. Make sure that close tickets (set to resolved). Provide solutions to the user's problem or, if this is not possible, provide alternatives. Own the problem. Take the user's perspective. 
-- *Slide End* --

-- *Slide* --
## HPC Support Policy 
We have an HPC Support Policy (`goo.gl/UPv4c8`). This includes policy for locking accounts, time of operations, code of conduct, service level targets, incident response handling, and priority users.
-- *Slide End* --

-- *Slide* --
## Software Builds
As with all HPC systems, whenever possible software is built from source. Usually this to ensure maximum optimisation of code, and sometimes it is to ensure that the software can be installed in the first place. Most of all however, it is to ensure that multiple version of the same software can be installed. This provides consistency and better reproducibility in code, whilst also allowing for different features. To achieve this, environment modules are employed. 
-- *Slide End* --

-- *Slide* --
## Software Builds (cont...)
In particular Spartan uses the Easybuild system to ensure build consistency and which incorporates the LMod environment modules system. LMod is virtually identical in most cases to the TCL-based environment modules system. 
-- *Slide End* --

-- *Slide* --
## Software Builds (cont...)
Easybuild requires a little explanation. Essentially it provides a repository of Python scripts to a particular software build ("Easybuild recipes"), which includes version numbers, dependencies, toolchains etc. These recipes in turn call an EasyBlock, which is a configuration style (e.g., Binary, ConfigureMake) and often specific to an application.
-- *Slide End* --

-- *Slide* --
## Software Builds (cont...)
Easybuild exists as a build user and has its own module file. It is important to purge existing modules before running an EasyBuild script.
```
ssh root@spartan-build.hpc.unimelb.edu.au   
su easybuild   
module purge   
module load EasyBuild   
cd /usr/local/easybuild/ebfiles_repo/Valgrind   
eb Valgrind-3.13.0-GCC-6.2.0.eb   
```
-- *Slide End* --

-- *Slide* --
## Licensed Software
Licensed software has a number of implementations. Some licenses (e.g., Gurobi) require a license file added to the software directory and the user sourcing that path (typically added to the `.bashrc`). Others (e.g., Matlab) have a FlexLM license file on our license server (`lic.hpc.unimelb.edu.au`). In most cases we should create a new group in Karaage and assign qualified users to that group. This incldes software that we have a sitewide license for (e.g., Gaussian).
-- *Slide End* --

-- *Slide* --
# User Outreach
-- *Slide End* --

-- *Slide* --
## Training
Many users (even post-doctoral researchers) require basic training in Linux command line, a requisite skill for HPC use. Extensive training programme for researchers available using andragogical methods, including day-long courses in "Introduction to Linux and HPC Using Spartan", "Linux Shell Scripting for High Performance Computing", and "Parallel Programming On Spartan". We have run specialist courses (e.g., for Orygen) are planning a Spartan to NCI transition course, and a GPGPU programming course.
-- *Slide End* --

-- *Slide* --
<img src="https://raw.githubusercontent.com/UoM-ResPlat-DevOps/SysAdminCourse/master/Images/hypnotoad.png" width="150%" height="150%" />
-- *Slide End* --
