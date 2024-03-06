
# arc-ansible
Ansible playbook for ARC installation

It assumes the following

- Already installed and working batch system.  SLURM or HTCondor are the best supported.
- The ARC server must be able to submit jobs to the batch system (i.e. be on the same network)
- Minimally 1 compute node if the lrms parameter below is different than `fork`
- Shared filesystem between the ARC server and the compute nodes for the sessiondir and cachedir. 
- The ARC server must have port 443 open for incoming connections.
- Advanced: If you are using remote datadelivery server(s) (variable `remotedelivery` is "yes"), make sure that the port configured for datadelivery requests is open in your firewall (e.g. 443). The sessiondir and cachedir (se explanation below) should typically be on this remote delivery rather than the ARC-CE and you must ensure that these directories are shared across the cluster - both ARC-CE and compute nodes. 

As part of this installation the server is set up with a host certificate generated by a Test-CA for testing purposes. In production make sure to replace this with a proper host certificate and update the arc.conf accordingly. See http://www.nordugrid.org/arc/arc7/admins/reference.html#x509-host-key and http://www.nordugrid.org/arc/arc7/admins/reference.html#x509-host-cert for details. 

# Variables controlling the playbook

You

```yaml

- hosts: frontend
  name: ARC installation
  gather_facts: yes
  become: yes
  roles:
    - role: arc_frontend
      sessionbasename: '/mnt/grid/sessiondir'
      cachebasename:   '/mnt/grid/cachedir'
      lrms_bin_path:   '/usr/local/bin'

```

## General variables

### domain
Domain name of the ARC server

Example: 

```
domain: "cern-test.uiocloud.no"
```

### submit_user
What user submits the batch system job. The batch call will be issued by this user, and the job in the batch system will run as this user. 

Example: 

```
submit_user: 
   user: "arcuser"
   group: "arcuser"
```

If the group-name is different than the user-name - make sure to put the correct one in the `group` variable. 


### use_repo
What Nordugrid repo to install from. 

Accepted values: 
- nordugrid - The default Nordugrid release repo 
- nordugrid-testing - The release candidate release repo 
- nordugrid-nightly - Nightly build repo


Before ARC 7 is released, use either nordugrid-testing or nordugrid-nightly. nordugrid-testing will be more stable than nordugrid-nightly


### arc_branch

What repo branch should ARC be of. The current released branch is "master", while the next upcoming major release can be found in "next". Currently ARC 7 is not yet released, and is in the "next" branch. Select this. 

Example: 

```
arc_branch: "next"
```

### site_type

This is a flag to trigger installation of ARC runtime environments needed for specific experiments. 
Accepted values are currently `atlas` or `default`. 

If you do not know what this means, select the ```default``` type


## Variables used to build the arc.conf

### controldir
ARCs job directory - storing all meta-data files related to a job. Local directory to the ARC server. 

If empty, ARC will use the default value "/var/spool/arc/jobstatus"

http://www.nordugrid.org/arc/arc7/admins/reference.html#controldir


### sessionbasename
This variable is used to build a set of session directory folder names. 

The sessiondir is ususally an existing shared filesystem that all compute nodes have access to. 
Use your actual existing shared filesystem for this. 

Example results: sessiondir01, sessiondir02 etc. 

Example: 
```
sessionbasename: "/mnt/grid/sessiondir"
```


http://www.nordugrid.org/arc/arc7/admins/reference.html#sessiondir

### cachebasename
Similar as sessiondir. 

http://www.nordugrid.org/arc/arc7/admins/reference.html#cachedir

### scratchdir
If the compute nodes have a scratch directory, add the path here. If not leave string empty. 

http://www.nordugrid.org/arc/arc7/admins/reference.html#scratchdir


### runtimedir
What directory to use for user-defined runtime environments. 

If left empty, no user-defined runtime directory will be created. Need to be set if site_type != default

http://www.nordugrid.org/arc/arc7/admins/reference.html#runtimedir


### lrms
What batch system you have. If no batch system connected, use "fork". 

Allowed values, see: http://www.nordugrid.org/arc/arc7/admins/reference.html#lrms


### lrms_bin_path
It is assumed that the LRMS bin folder is `/usr/bin`. If that's not the case please update `lrms_bin_path` accordingly (e.g. `lrms_bin_path: /usr/local/bin`)

Allowed values, see: http://www.nordugrid.org/arc/arc7/admins/reference.html#lrms


### queues
A list of batch system queues. If SLURM batch system, this is equivalent to the partition name(s). If fork, any uniqe string. Multivalued. 

http://www.nordugrid.org/arc/arc7/admins/reference.html#queue-name-block


Example: 
```
queues: 
  - main
```


### remotedelivery
Should ARC have a stand-alone remote data delivery server. If it does then make sure that your inventory ansible file has a `datadelivery` group. 

Information about the configuration in arc.conf on the ARC-CE: http://www.nordugrid.org/arc/arc7/admins/reference.html#deliveryservice

Information about how to set up and configure a standalone remote data delivery server: http://www.nordugrid.org/arc/arc7/tech/data/dds.html

Accepted values "yes", "no"


### remotedelivery_port
What port the remote datadelivery server listens for datastaging requests. 

Accepted values: Any port-number you are using, e.g. 443. 


### hepspec
If you have benchmarked your servers, add the value here. The values must be in HEPSPEC value or Hepscore23 value. 
If you do not have this, leave the parameter empty. 

The hepspec is used in the arc.conf as the `benchmark` value in the `queue` block: http://www.nordugrid.org/arc/arc7/admins/reference.html#reference-queue-benchmark - it is only relevant if you are relying on accounting from ARC. 

Example: 

```hepspec: 14.998```


## Variables related to the Nordugrid repository

### nordugrid_os_v

The os version of the ARC server. Used to point to the correct Nordugrid repo. 

Accepted values are found here: http://www.nordugrid.org/arc/arc7/common/repos/repository.html

The value is automatically filled based on the OS version of your server. 


### nordugrid_os_dir

The os type of the ARC server. Used to point to the correct Nordugrid repo. 

Accepted values are: "centos/centos-stream/rocky/fedora/debian/ubuntu"

If `almalinux` - use `rocky` 

The value is automatically filled based on the OS version of your server (ansible_os_family). 


### nordugrid_release_v

This is the release version of the Nordugrid release package (not the ARC version itself). 

The value is 6.1 for ARC 7 on RedHat flavour servers, and 6.2 for ARC 7 on Debian flavour servers. 

The value is automatically filled based on the OS version of your server (ansible_os_family). 


### nordugrid_os_vname

(ansible_distribution_release)

The os version name of the ARC server, only relevant for Debian flavour servers. Used to point to the correct Nordugrid repo. 

Accepted values are found here: http://www.nordugrid.org/arc/arc7/common/repos/repository.html - example `bookworm` for Debian. 

The value is automatically filled based on the OS version of your server. 


# Testing the installation

Once you have run the playbook you can submit a test-job using the ARC client installed on the same machine as the ARC server (done by this ansible playbook). 

1. Create a proxy certificate:
   ```arcproxy```
2. Go into the `testing` folder where some prepared test-files can be found
   ``` cd ~/testing```
3. Execute the command in submit-cmd.sh
   ``` /bin/bash submit-cmd.sh```

More on how to test your site: http://www.nordugrid.org/arc/arc7/users/submit_job.html

