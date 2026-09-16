# Getting started on Slurm clusters

This how to will show how to setup a project on a compute cluster. The detailed procedure
will show the case of the CalculQuebec servers including the signup process but procedure
will likely be similar across different compute platforms. 

The paths shown are specific to the alliance can setup and are shown for reference. Your 
specific compute provider may have a different server architecture that requires different 
signup and connection modalities.

The document is laid out as follows:
- [Setting up (optional)](#setting-up-on-alliance-can---optional)
    - [Setting up CalculQuébec Account](#setting-up-account)
    - [Requesting Access to a resource](#requesting-access-to-a-resource)
- [Running analyses](#running-a-traclus-batch-job)
    - [Cloning and compiling traclusdl](#cloning-and-compiling-traclusdl)
    - [Checking the build](#checking-the-build)
    - [Copying input files](#copying-input-files)
    - [Setting up jobs](#setting-up-jobs)
    - [Starting a job](#starting-a-job)
    - [Downloading results back to your local machine](#downloading-the-data-back-to-your-computer)

## Setting up on Alliance Can - optional
### Setting up account

Accounts for calcul Québec have to be created for a main researcher. A procedure can be 
found [here](https://docs.alliancecan.ca/wiki/Apply_for_a_CCDB_account/fr). You'll need 
to create the access with an institutional address.

If you are a team member you'll have to provide a CCRI code that will put a requests
to your main researcher to give you access 

### Requesting access to a resource

Once you've created an account and have been approved as a member of a team, you'll need 
to request access to one of the compute resource. You can do this by asking for access 
[here](https://ccdb.alliancecan.ca/me/access_systems). General purpose compute clusters 
have thus far been adequate for the usage of this method with metropolitan level analyses.

This [link](https://docs.alliancecan.ca/wiki/National_systems) shows the different systems
available.

### Connecting to the cluster

Connection to the cluster is done via SSH on your terminal. You'll have to become 
familiar with basic Unix command lines like mkdir, cd, ls, zip. To connect to the cluster,
you'll have to setup 2FA with Duo which is discussed [here](https://docs.alliancecan.ca/wiki/Multifactor_authentication/en).
You'll then open a terminal and use the following command

```
ssh USERNAME@CLUSTER.alliancecan.ca
```
That command will open a remote terminal in the home directory on your chosen cluster
## Running a traclus batch job
### Cloning and compiling TraclusDL

Once you have access to to the terminal, used the following command to navigate to 
your project folder. The folder uses your main researcher's identifier and your user
name to setup a working directory. The main reasearcher id usually follows the following
pattern def-filast

```
cd ./links/projects/your-main-researcher-id/your-username
```

You can then clone the traclusdl directory using the following command:

```
git clone https://github.com/chairemobilite/traclusdl.git
```

This will create the traclusDL directory in your cluster project folder. You'll need to 
compile traclusdl. To do so, start by navigating into the directory:

```
cd traclusdl
```

then run the build command in order to be able to run the script

```
cargo build --release
```
You will need to select one of the installed rust distributions then the build will
proceed
### Checking the build 

To check the build run the following command 
```
./target/release/traclusdl_cli --version
```

This should show `traclusdl_core X.X.X` 

To get the various options to run the script, you can use the following command

```
./target/release/traclusdl_cli --help
```
### Copying input files

Input files need to be copied over from your local machine to the server. The easiest 
option on linux is to use scp. The input format is described in the README.md of the repo.
It's suggested to zip the various input files and copy them over. This should
be done in your local terminal not in the ssh window:

```
scp ./path-to-your-zip/your-file.zip USERNAME@CLUSTER.alliancecan.ca:/home/username/links/projects/def-filast/username/
```

Then in the ssh, run the following command to :
```
unzip /home/username/links/projects/def-filast/usermane/your-file.zip
```
This should extract the files locally or create a subdirectory depending on how you set it up. Alternate methods for uploading files are laid out [here](https://docs.alliancecan.ca/wiki/Transferring_data)

### Setting up jobs
The most straightforward way to run a job is through a batch file. To view a more complete
write-up. You can refer to the alliancecan [wiki](https://docs.alliancecan.ca/wiki/Running_jobs)
Start by navigating your folder 

```
cd /home/username/links/projects/def-filast/username
```
As a basic setup create and edit the following file and

```
nano batch-setup.sh
```

Your batch file should look something like what is below. The batch file can run multiple files or change the run options. 
You may need to change the number of cpus depending on the specs of your specific cluster. 

```
#!/bin/bash
#SBATCH --time=05:00:00
#SBATCH --account=def-filast
#SBATCH --mincpus=192
#SBATCH --cpus-per-task=192
#SBATCH --mem-per-cpu=2G

./traclusdl/target/release/traclusdl_cli -f ./yourfile-1.csv -d 250 -a 5 -s 1000 -n 1000 -m parallel-rayon
./traclusdl/target/release/traclusdl_cli -f ./yourfile-2.csv -d 250 -a 5 -s 1000 -n 1000 -m parallel-rayon
./traclusdl/target/release/traclusdl_cli -f ./yourfile-2.csv -d 250 -a 10 -s 1000 -n 450 -m parallel-rayon

sleep 30

```

### Starting a job

You can then run the job by using the following command:
```
sbatch batch-setup.sh
```
You can check the job is running using the following command:

```
sq
```
The job should appear in your job list. Once complete it will no longer appear and you can check the `slurm-job-number.out` file to check on job progress.


### Downloading the data back to your computer
Once the job is complete you can use the zip command to put all the relevant folders into a zip file. It's suggested that you put the files into a directory 
and from the parent directory use 

```
zip -r your-dir.zip ./your-dir
```
Once that is done, go back to the terminal in your local machine terminal and type:

```
scp  USERNAME@CLUSTER.alliancecan.ca:/home/username/links/projects/def-filast/username/your-dir.zip ./
```
which should copy the results back to your machine.
