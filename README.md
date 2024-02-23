
# arc-ansible
Ansible playbook for ARC installation


# Variables controlling the playbook

You find the variables in group_vars/all file. 



## General variables 

### domain
Domain name of the ARC server

Example: 

```
domain: "cern-test.uiocloud.no"
```



### default_user
This is the default user on the ARC server (assuming a cloud server). You can most probably leave it as is assuming you have configured your ansible inventory with the correct ansible_user name. I

Example: 

``` 
default_user:
   user: "{{ ansible_user }}"
   group: "{{ ansible_user }}"
```



If the group-name is different than the user-name - - make sure to put the correct one in the `group` variable. 


### submit_user
What user submits the batch system job. If the user does not exist, the playbook will create the user. This user should be a normal user. It could for instance just be the default_user set above. The batch call will be issued by this user, and the job in the batch system will run as this user. 

Example: 

```
submit_user: 
   user: "arcuser"
   group: "arcuser"
```

If the group-name is different than the user-name - make sure to put the correct one in the `group` variable. 

### nordugrid_os_dir

The os type of the ARC server. Used to point to the correct Nordugrid repo. 

Accepted values are: "centos/centos-stream/rocky/fedora/debian/ubuntu"

If `almalinux` - use `rocky` 


### nordugrid_os_v
The os version of the ARC server. Used to point to the correct Nordugrid repo. 

Accepted values are found here: http://www.nordugrid.org/arc/arc7/common/repos/repository.html

### nordugrid_release_v
This is the release version of the Nordugrid release package (not the ARC version itself). 

Typically 6.1 is the correct value for ARC 7. 


### site_type

This is a flag to trigger installation of ARC runtime environments needed for specific experiments. 
Accepted values are currently `atlas` or `default`. 

If you do not know what this means, select the ```default``` type

### arc_branch

What repo branch should ARC be of. The current released branch is "master", while the next upcoming major release can be found in "next". Currently ARC 7 is not yet released, and is in the "next" branch. Select this. 

Example: 

```
arc_branch: "next"
```

### use_repo
What Nordugrid repo to install from. 

Accepted values: 
- nordugrid - The default Nordugrid release repo 
- nordugrid-testing - The release candidate release repo 
- nordugrid-nightly - Nightly build repo


Before ARC 7 is released, use either nordugrid-testing or nordugrid-nightly. nordugrid-testing will be more stable than nordugrid-nightly


## Variables used to build the ARC configuration file `/etc/arc.conf`

### remotedelivery
Should ARC have a stand-alone remote data delivery server. 

http://www.nordugrid.org/arc/arc7/admins/reference.html#deliveryservice

Accepted values "yes", "no"


### scratchdir
If the compute nodes have a scratch directory, add the path here. If not leave string empty. 

http://www.nordugrid.org/arc/arc7/admins/reference.html#scratchdir


### lrms
What batch system you have. If no batch system connected, use "fork". 

Allowed values, see: http://www.nordugrid.org/arc/arc7/admins/reference.html#lrms


### controldir
ARCs job directory - storing all meta-data files related to a job. Local directory to the ARC server. 

If empty, ARC will use the default value "/var/spool/arc/jobstatus"

http://www.nordugrid.org/arc/arc7/admins/reference.html#controldir

### runtimedir
What directory to use for user-defined runtime environments. 

If left empty, no user-defined runtime directory will be created

http://www.nordugrid.org/arc/arc7/admins/reference.html#runtimedir

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


### queues
A list of batch system queues. If SLURM batch system, this is equivalent to the partition name(s). If fork, any uniqe string. Multivalued. 

http://www.nordugrid.org/arc/arc7/admins/reference.html#queue-name-block


Example: 
```
queues: 
  - main
```

### hepspec
If you have benchmarked your servers, add the value here. The values must be in HEPSPEC06 value or HEPSCORE value. 
If you do not have this, leave the parameter empty. 

Example: 

```hepspec: 14.998```

# Testing the installation
Once you have run the playbook you can submit a test-job using the ARC client installed on the same machine as the ARC server (done by this ansible playbook). 

1. Create an x509 user certificate for the user you want to submit as - if it is different than the ARC subitter configured in arc.conf (submit_user variable in the group_vars/all file). Example: cloudadm - here I am setting validity for 360 days, installing the usercert in the /home/cluodadm/.globus folder and forcing it
   ``` sudo arcctl test-ca usercert -v 360 -i cloudadm  -f ```
2. Create a proxy certificate using the already installed Test-CA - as same user as above (here cloudadm) do:
   ```arcproxy```
3. Go into the `testing` folder where some prepared test-files can be found
   ``` cd ~/testing```
4. Execute the command in submit-cmd.sh
   ``` /bin/bash submit-cmd.sh```

If you instead want to submit a test-job as the user used to map jobs already configured in arc.conf (submit_user in group_vars/all) - you need to su into that user. You can then skip step 1 as this is already taken care of in the ansible playbook. 

More on how to test your site: http://www.nordugrid.org/arc/arc7/users/submit_job.html

