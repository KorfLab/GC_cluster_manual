Title:   How To Use the UC Davis Genome Center Computer Clusters
Authors:  Keith Dunaway, Stella Hartono, and Keith Bradnam
Date:    2014-05-20
Address: Genome Center, UC Davis, Davis, CA, 95616  
 
 

<center>

# How To Use the Genome Center Computer Clusters

### Keith Dunaway, Stella Hartono, and Keith Bradnam

### Version 1.2
[![](http://i.creativecommons.org/l/by-nc/4.0/88x31.png "Creative commons license")](http://creativecommons.org/licenses/by-nc/4.0/)
This work is licensed under a Creative Commons Attribution-NonCommercial 4.0 International License
</center>
---


## Table of Contents

+ [Part 1: Introduction][1.1]
	+ [What is the Genome Center Cluster?][1.2]
	+ [How to get an account to use the cluster][1.3]
	+ [Computer cluster terminology][1.4]
	+ [Best and Worst Practices][1.5]
		+ [What you should do (good practices)][1.6]
		+ [What you should NOT do (worst practices)][1.7]
+ [Part 2: How to submit jobs to cluster][2.1]
	+ [Login to cluster][2.2]
	+ [Transfer files to and from the cluster][2.3]
		+ [Upload Examples][2.4]
		+ [Download Examples][2.5]
		+ [Copying Multiple Files][2.6]
	+ [Installing Programs on the Cluster][2.7]
		+ [Precompiled binary][2.8]
		+ [Compiling source on the server][2.9]
	+ [QLOGIN][2.10]
	+ [QSUB][2.11]
	+ [QSTAT and QDEL][2.12]
	+ [O and E Status files][2.13]
+ [Part 3: Cluster Description][3.1]
+ [Part 4: Qsub and qlogin parameters][4.1]
+ [Part 5: Qstat options/arguments][5.1]
+ [Appendix 1: Frequently Asked Questions][A1.1]
	+ [How do I login without having to type my password?][A1.2]
+ [Appendix 2: Thanks][A2.1]
+ [Appendix 3: Release Notes][A3.1]


---


## Part 1: Introduction [1.1]

This document is designed to give users who are new to cluster computing a working knowledge of how to use the Genome Center Cluster that is managed by the [UC Davis Genome Center](http://genomecenter.ucdavis.edu).  If you have any questions that are not covered in this manual, contact Keith Dunaway at [kwdunaway at ucdavis dot edu](mailto:kwdunaway@ucdavis.edu) (but please check to see if your question is addressed in the [FAQ section][A1.1] first).


### What is the Genome Center Cluster? [1.2]

The GC Cluster is a group of linked computers that are configured in such that they can perform a large number of compute tasks in parallel. The hardware specifications of each computer in the cluster may vary, but they all run the same operating system (Linux). The Genome Center maintains two different clusters (merlot and shiraz) and hardware specifications of all of the computers in both clusters can be found in the [Cluster Description][3.1] section.


### How to get an account to use the cluster [1.3]

The GC Cluster is available to labs and staff at UC Davis who are associated with the Genome Center. You will need to a) have a Genome Center account and b) have special ‘cluster permission’ added to this account. To accomplish either or both, please have your Principal Investigator email Mike Lewis (the cluster manager) at [mclewis at ucdavis dot edu](mailto:mclewis@ucdavis.edu) with the following information:

1.  Prefered contact email address
2.  Desired username 
3.  What lab(s) you work in

Please allow a couple of days for the account to be created.


### Computer cluster terminology [1.4]

In order to use the cluster, you first need to *login* to the desired cluster (merlot or shiraz). Computer clusters have a main computer that is designated as the *head node*. This is the computer that you will connect to when you login to the cluster. From the head node you can submit a task to any computer in the cluster, which are known as the *nodes*. Each task that you need to run is submitted as a *job*. Computer clusters excel at tasks which can be broken down into many small jobs (e.g. BLAST searches), which can then be run in parallel across multiple nodes. 


### Best and Worst Practices [1.5]

This section outlines the best (and worst) practices for working with the cluster. Remember that the cluster is used by many people and if you break the cluster, it’s broken for everyone (not just yourself). So **_please read the following BEFORE starting to use the cluster!_** 


#### What you should do (good practices) [1.6]

1.  Compress your larger files (greater than 25 MB) *before* uploading/downloading them to/from the cluster (use a Unix tool such as `gzip`). This will greatly minimize the transfer time which saves you time *and* frees up bandwidth for other users.
2.  Test new jobs using the `qlogin` tool before submitting them using the `qsub` tool. This will allow you to test your script and ensure it runs without errors  as well as keep the head node free.
3.  Name all of your jobs with the same file extensions (e.g. ‘.bash’). This will be invaluable when organizing multiple log files and the job submission files you create.


#### What you should NOT do (worst practices) [1.7]

1.  Do not run programs on the head node! This node is only to be used for submitting jobs to other nodes by using the `qlogin` command. See the section on [QLOGIN][2.10] for more information.  
2.  Do not stay logged in with `qlogin` any longer than necessary. When using `qlogin`, you tie up a node until you logout or get disconnected (even if you are idle). This ties up resources that someone else could be using.
3.  Do not overuse disk space; make sure that you do not exceed 50 GB. The cluster is shared with >20 other users and there’s only 5 TB of disk space. Keep the most important files (programs, scripts) on the cluster and copy others (result) to your computer then delete them from the cluster.  If you ever get a space error, you must clean up space to reduce your footprint.  To remove a file once disk space is full, type `> filename` then delete the file.




## Part 2: How to submit jobs to cluster [2.1]

All of the commands found in this manual are written from the point of view of someone connecting to the cluster from a Unix/Linux computer. Please open a suitable terminal application to use these commands. In later versions of this document, we plan on introducing equivalent Windows commands. The following sections all assume that you have a working GC cluster account (see Introduction section above). Also, while this manual only refers to the merlot cluster for explanations, shiraz works the same for all examples. The address for the shiraz cluster is shiraz.genomecenter.ucdavis.edu.


### Login to cluster [2.2]

Use the `ssh` command to connect to the cluster using your cluster login name. E.g. for a user ‘john’ who is logging in to the ‘merlot’ cluster:

```bash
ssh john@merlot.genomecenter.ucdavis.edu
```

**Note:** you will be asked to enter your cluster account password.

Once you login, you should be in your home directory, location: `/home/[username]/`.  So john’s home directory would be `/home/john/`.  Please contact the cluster administrator if this is not the case.


### Transfer files to and from the cluster [2.3]

Use the `scp` Unix command to copy files to and from the cluster. We advise that you run `scp` from your *local machine* and not while logged in to the cluster. If you are not familiar with the `scp` command, the basic syntax is:

```bash
scp [[user@]host1:]file1 [[user@]host2:]file2
```


#### Upload Examples [2.4]

The following example assumes that a user (‘john’) wants to transfer a file (‘seqs.fa’) that is in their current directory on their local machine to their home directory on the cluster:

```bash
scp seqs.fa john@merlot.genomecenter.ucdavis.edu:~/seqs.fa
```

**Note:** you will be asked to enter your cluster account password]

In this case, the `~/` syntax is a shortcut to refer to the user’s home directory on the cluster, which could also be accessed as `/home/john`. Whatever is specified after `john@merlot.genomecenter.ucdavis.edu:` should be a valid location on the cluster where you have permission to write files to (in most cases, this should be your home directory on the cluster). You can optionally rename a file while copying it:

```bash
scp data.txt john@merlot.genomecenter.ucdavis.edu:~/data.fq
```


#### Download Examples [2.5]

In this example we assume that our user ‘john’ wants to copy a file (‘seqs.blast’) from his home directory on the cluster to whatever his current directory is on his local machine:

```bash
scp john@merlot.genomecenter.ucdavis.edu:~/seqs.blast .
```

**Note:** The dot character in this example is the method that Unix uses to refer to your *current directory*. You could replace this with any directory location on your local machine that you have permission to write to.


#### Copying Multiple Files [2.6]

We could also transfer multiple files by using similar command, in which we specify the destination at the end:

```bash
scp <file1> <file2> <file3 file4 file5 <john@merlot:~/destination>
```

Therefore, using previous examples, this will copy data.txt, data1.txt, and data2.txt from local computer to data_new folder in merlot:

```bash
scp ./data.txt ./data1.txt ./data2.txt john@merlot.genomecenter.ucdavis.edu:~/data_new/
```

However, the best way to transfer multiple files is to copy an entire folder. This is done by using the `-r` option of the `scp` command:

```bash
scp -r john@merlot.genomecenter.ucdavis.edu:~/results .
```

In this example, the directory ‘results’ that exists inside John’s home directory on the cluster is copied to his current directory on his local machine.


### Installing Programs on the Cluster [2.7]

You cannot assume that any program that you want to use (BLAST, bowtie etc.) will already be installed on each node of the cluster. In most cases, you must install the program you want to run yourself. There are two ways to do this.


#### Precompiled binary [2.8]

The best way to install programs on the cluster — and this is also the easiest way — is to upload *precompiled binaries* of the program of interest. You will need to go to the website of the person/group who developed that software and download the correct version of the software (make sure the binaries were compiled for Linux). It’s best to use 64-bit compiled binaries. However, 32-bit binaries will work if 64-bit versions are unavailable. 

After downloading the program and copying it to the cluster you will then have to modify your `$PATH` environment variable in order to let the cluster know where it can find these programs. E.g. assuming that our user John has downloaded the ‘Bowtie’ program to his ‘programs’ directory on the cluster, he would then add this directory to his `$PATH` like so:

```bash
PATH=$PATH:/home/john/programs/Bowtie/
```

This would assume that the directory `/home/john/programs/Bowtie` contains one or more executable programs that can to be run. Many bioinformatics software packages will often place the executable programs (known as *binaries*) into a ‘bin’ subdirectory. E.g. you might install NCBI BLAST+ and end up with the BLAST programs (`blastn`, `tblastx`, etc.) in the following directory:

```bash
/home/john/programs/NCBI_BLAST/bin/
```

An easy way to add multiple directories to your PATH in one go, is by referring to your home directory using the `~/ `syntax. E.g. to add the Bowtie and NCBI BLAST+ directories to your PATH at the same time, you could do the following:

```bash
PATH=$PATH:~/programs/Bowtie/:~/programs/NCBI_BLAST/bin
```

Remember, you have to add paths to the PATH variable whenever you start a new instance. This includes every new `qlogin`, `qsub`, and `ssh` connection.


#### Compiling source on the server [2.9]

There are some programs that require other programs to work. For instance, [CEAS](http://liulab.dfci.harvard.edu/CEAS/) needs R and relevant R libraries to run properly. So, you first need to download and install the *source code* (**not** the pre-compiled one). Using the R example:

1.  Download the [source code of R](http://cran.r-project.org/src/base/R-2/) (.tar.gz), and **not** the [pre-compiled one](http://cran.r-project.org/bin/linux/), then scp it to your cluster home directory.
2.  Untar and unzip the source code by typing this on cluster home directory. The command is:

```bash
tar zxvf R-X.XX.X.tar.gz
```

This will create the R-X.XX.X directory. Change directory to the R directory afterwards.
3.  Most programs will include the "configure" command. Therefore, in order to install the program to your home directory, type into the terminal:

```bash
./configure --prefix=$HOME/folder
```

R will do some OS configurations and the terminal will show a bunch of command lines which may take a while. If your program doesn’t do anything after you type configure, skip to step 4.
4. Next, type this into the terminal:

```bash
make
```

The program will then be compiled. The terminal will show more command lines. Again, this may take a while (usually longer than configure).
5. To install the program in the directory that you specified by `--prefix=$HOME/folder,`you will need to type:

```bash
make install
```


### QLOGIN [2.10]

The best way to run jobs on the cluster is by using the `qsub` command. However, you will first want to make sure your jobs run without error. So, you should login to a node with the same requirements that your job will need in order to run. For example:

```bash
qlogin
```

In this example, we logged into a node using the default parameters. This means there is a single processor allocated for your login and the shell is /bin/bash. This default shell is used for the head node as well as all of the documentation in this manual. If you wish to use a different shell, you can change it after logging in by typing:

```bash
SHELL=<shell_path>
```

A common desire for jobs is to use multiple processors. This can be designated using the `-pe threaded` option for the `qlogin` command. It is important to note that you cannot designate more processors than a node has, so check the [Cluster Description][3.1] to ensure a node can meet the requirements of your job. In the following example, we designate 4 processors:

```bash
qlogin -pe threaded 4
```

Now you can use `qlogin` to login to a node (not the head node) to test all of your programs/scripts in order to ensure proper functionality. Please see [qsub and qlogin parameters][4.1] for a full list of parameters.



### QSUB [2.11]

After you have used `qlogin` to test that your programs will work, you can then submit jobs to the cluster using the `qsub` command. To use `qsub`, you will first need to create a text file which details all of the Unix steps that will run the submitted job. Some of these steps will be the bioinformatics programs that you need to run, but other steps will help you configure your environment, prepare output directories, etc. Technically, this text file is a bash script that will run as its own program. Use a Unix text editor such as `nano` (or any other that is installed and which you are familiar with) to create your qsub command file, and write each step on its own line:

```bash
nano example_qsub.bash
```

Then type the necessary commands within the document itself. Here is an example file where our user (John) is preparing a qsub file which will let him run the bowtie program:

```bash
#!/bin/bash
PATH=$PATH:/home/john/programs/Bowtie/
export PATH
mkdir /home/john/output
mkdir /home/john/output/bowtieoutput
bowtie -q -p 4 -o /home/john/output/bowtieoutput /home/john/genome/mm9 /home/john/input/input.fastq
```

There are a couple of important things to note in this example. First, the extension of this file is .bash. While not necessary, it is a good practice for organizational purposes. Second, the first line of the document should declare what shell you expect to be running in: `#!/bin/bash`. Again, not necessary but good practice. The next line adds a directory to the `$PATH` and if this was not done, the node processing the bash commands would not be able to find the bowtie program. Finally, the `-q `option was added when calling `bowtie` which tells the program to only send essential information to the screen. This will be important when checking the status of your processes. Please see [QSTAT and QDEL][2.12] and [O and E Status files][2.13] for more information.

After you are done creating and saving your bash file, you will want to submit it to the cluster to run. This is done by using the qsub command:

```bash
qsub -S /bin/bash -pe threaded 4 example_qsub.bash
```

In this example, we specify two parameters. The `-S` option designates the shell to be `/bin/bash` (which is the default shell when you login to the cluster but not the default shell for `qsub`). When using `qsub`, you must designate the shell in the command line instead of in your job bash file. This is the ONLY difference between calling `qsub` and `qlogin`. The `-pe threaded` option allocates 4 processors to the task submitted through `qsub`. Please see [qsub and qlogin parameters][4.1] for a full list of parameters. 

Once you submit a job using `qsub`, your job will be added to the que. It will wait there until a node with the designated parameters is free to run your job. You can check the status of your job using the `qstat` command (see [QSTAT and QDEL][2.12] for a more detailed explanation). Once your job runs, it will do all of the commands outlined in your job file as well as create status files (see [O and E Status files][2.13] for a more detailed explanation). After your job is finished, all new files will be readily available for viewing and download through the head node.



### QSTAT and QDEL [2.12]

Once you submit tasks through `qsub`, you should check the status of these jobs using `qstat`. Just type the following into the terminal:

```bash
qstat
```

The screen should show you a screen that looks something like this:

```bash
job-ID prior name user state submit/start at queue slots ja-task-ID
-------------------------------------------------------------------
4221962 0.50500 example_qsub.bash  john r  04/30/2012 09:31:12 fat.q@fat-node-0-0.merlot.geno   4 
```

This gives you a lot of information but the most important is found under `state` at column 5. The state should be `qw` (cued) or `r` (running). However, sometimes there is an error that is noted by having a state of `Eqw` (error in cued) or `Er` (error running). If this is the case, then you need to cancel the job immediately by typing either one of these:

```bash
qdel 4221962
qdel example_qsub.bash
```

Sometimes there is a job submission error so you should resubmit the job. If you still get the error, check your job by submitting them manually through `qlogin`. 

You can also check the status of all of the nodes on the cluster using `qstat`. This is useful for looking at the availability of certain nodes you wish to task for your job. For instance, if you want to allocate 8 processors to your job, you can check the status of the cluster first to see if any nodes are available that can handle this. If not, you may think about lowering the amount of processors required to run your job. Type the following to get a full list of cluster status information:

```bash
qstat -f
```

This should print out a huge list of information for each set of nodes. Below is an example of what the output will look like:

```bash
queuename        qtype resv/used/tot. load_avg  arch states
---------------------------------------------------------------------
all.q@compute-0-1.merlot.genom BIP  0/0/2  0.00  lx26-amd64
---------------------------------------------------------------------
all.q@compute-0-10.merlot.geno BIP  0/0/2  -NA-  lx26-amd64  E
---------------------------------------------------------------------
```

The 6 columns explained:

1.  `queuename` - name of node. You will not have access to some of these so only focus on the ones you do have access to. 
2.  `qtype` - lists what types of jobs can be run on this node (should read `BIP`). If not, there may be an error with that node.
3.  `resv/used/tot.` - shows the amount of processors for these three classes in #/#/# format. For example, if there is a queue with no reserved processors, 9 currently used processors, and 16 total processors then the column would read `0/9/16`. Very useful for processor job determination.
4.  `load_avg` - shows average processors used.
5.  `arch` - architecture of the node. It should be the same for every queue: `1x26-amd64`.
6.  `states` - shows if there is an error on the subcluster. Usually blank (if working properly). If there are any characters in this column then the subcluster is not working properly.


### O and E Status files [2.13]

Once your job starts running after submitting (using `qsub`), two files will be created in your home directory with the following format:

```bash
example_qsub.bash.o4221962
example_qsub.bash.e4221962
```

The output (o) file contains everything that your job would normally print to screen. The error (e) file contains any error messages produced from your job. Sometimes, the print to screen output is found in the error file so check both files to see all of this information.

It is important to save this information for troubleshooting and bookkeeping purposes. However, you may want to clean up your home directory by moving these files to a subfolder. I suggest creating a folder (logs for example) and moving them there. If you named all of your submissions with at `.bash` extension, then moving the files is relatively easy. Just type the following:

```bash
mv *.bash.o* logs/
mv *.bash.e* logs/
```

## Part 3: Cluster Description [3.1]

This section describes the physical properties of the cluster as well as the nodes (good reference for submissions). Contact Mike Lewis for current information. 	Also, visit the following link for more information: [http://wiki.bioinformatics.ucdavis.edu/images/e/e7/Cluster_presentation.pdf](http://wiki.bioinformatics.ucdavis.edu/images/e/e7/Cluster_presentation.pdf)

## Part 4: Qsub and qlogin parameters [4.1]

This section will outline all of the qsub options. Everything that works for qsub works for qlogin.

```bash
usage: qsub [options]

  [-a date_time]              request a start time

  [-ac context_list]            add context variable(s)

  [-ar ar_id]               bind job to advance reservation

  [-A account_string]           account string in accounting record

  [-b y[es]|n[o]]             handle command as binary

  [-c ckpt_selector]            define type of checkpointing for job

  [-ckpt ckpt-name]            request checkpoint method

  [-clear]                 skip previous definitions for job

  [-cwd]                  use current working directory

  [-C directive_prefix]          define command prefix for job script

  [-dc simple_context_list]        delete context variable(s)

  [-dl date_time]             request a deadline initiation time

  [-e path_list]              specify standard error stream path(s)

  [-h]                   place user hold on job

  [-hard]                 consider following requests "hard"

  [-help]                 print this help

  [-hold_jid job_identifier_list]     define jobnet interdependencies

  [-hold_jid_ad job_identifier_list]    define jobnet array interdependencies

  [-i file_list]              specify standard input stream file(s)

  [-j y[es]|n[o]]             merge stdout and stderr stream of job

  [-js job_share]             share tree or functional job share

  [-jsv jsv_url]              job submission verification script to be used

  [-l resource_list]            request the given resources

  [-m mail_options]            define mail notification events

  [-masterq wc_queue_list]         bind master task to queue(s)

  [-notify]                notify job before killing/suspending it

  [-now y[es]|n[o]]            start job immediately or not at all

  [-M mail_list]              notify these e-mail addresses

  [-N name]                specify job name

  [-o path_list]              specify standard output stream path(s)

  [-P project_name]            set job's project

  [-p priority]              define job's relative priority

  [-pe pe-name slot_range]         request slot range for parallel jobs

  [-q wc_queue_list]            bind job to queue(s)

  [-R y[es]|n[o]]             reservation desired

  [-r y[es]|n[o]]             define job as (not) restartable

  [-sc context_list]            set job context (replaces old context)

  [-shell y[es]|n[o]]           start command with or without wrapping <loginshell> -c

  [-soft]                 consider following requests as soft

  [-sync y[es]|n[o]]            wait for job to end and return exit code

  [-S path_list]              command interpreter to be used

  [-t task_id_range]            create a job-array with these tasks

  [-tc max_running_tasks]         throttle the number of concurrent tasks (experimental)

  [-terse]                 tersed output, print only the job-id

  [-v variable_list]            export these environment variables

  [-verify]                do not submit just verify

  [-V]                   export all environment variables

  [-w e|w|n|v|p]              verify mode (error|warning|none|just verify|poke) for jobs

  [-wd working_directory]         use working_directory

  [-@ file]                read commandline input from file

  [{command|-} [command_args]]

account_string     account_name

complex_list      complex[,complex,...]

context_list      variable[=value][,variable[=value],...]

ckpt_selector      'n' 's' 'm' 'x' <interval> 

date_time        [[CC]YY]MMDDhhmm[.SS]

job_identifier_list   {job_id|job_name|reg_exp}[,{job_id|job_name|reg_exp},...]

jsv_url         [script:][username@]path

mail_address      username[@host]

mail_list        mail_address[,mail_address,...]

mail_options      'e' 'b' 'a' 'n' 's'

working_directory    path

path_list        [host:]path[,[host:]path,...]

file_list        [host:]file[,[host:]file,...]

priority        -1023 - 1024

resource_list      resource[=value][,resource[=value],...]

simple_context_list   variable[,variable,...]

slot_range       [n[-m]|[-]m] - n,m > 0

task_id_range      task_id['-'task_id[':'step]]

variable_list      variable[=value][,variable[=value],...]

wc_cqueue        wildcard expression matching a cluster queue

wc_host         wildcard expression matching a host

wc_hostgroup      wildcard expression matching a hostgroup

wc_qinstance      wc_cqueue@wc_host

wc_qdomain       wc_cqueue@wc_hostgroup

wc_queue        wc_cqueue|wc_qdomain|wc_qinstance

wc_queue_list      wc_queue[,wc_queue,...]

ar_id          advance reservation id

max_running_tasks    maximum number of simultaneously running tasks

```


## Part 5: Qstat options/arguments [5.1]

```bash

usage: qstat [options]

    [-ext]              view additional attributes

    [-explain a|c|A|E]        show reason for c(onfiguration ambiguous), a(larm), suspend A(larm), E(rror) state

    [-f]               full output

    [-F [resource_attributes]]    full output and show (selected) resources of queue(s)

    [-g {c}]             display cluster queue summary

    [-g {d}]             display all job-array tasks (do not group)

    [-g {t}]             display all parallel job tasks (do not group)

    [-help]              print this help

    [-j job_identifier_list ]     show scheduler job information

    [-l resource_list]        request the given resources

    [-ne]               hide empty queues

    [-pe pe_list]           select only queues with one of these parallel environments

    [-q wc_queue_list]        print information on given queue

    [-qs {a|c|d|o|s|u|A|C|D|E|S}]   selects queues, which are in the given state(s)

    [-r]               show requested resources of job(s)

    [-s {p|r|s|z|hu|ho|hs|hd|hj|ha|h|a}] show pending, running, suspended, zombie jobs,

                     jobs with a user/operator/system/array-dependency hold, 

                     jobs with a start time in future or any combination only.

                     h is an abbreviation for huhohshdhjha

                     a is an abbreviation for prsh

    [-t]               show task information (implicitly -g t)

    [-u user_list]          view only jobs of this user

    [-U user_list]          select only queues where these users have access

    [-urg]              display job urgency information

    [-pri]              display job priority information

    [-xml]              display the information in XML-Format

pe_list         pe[,pe,...]

job_identifier_list   [job_id|job_name|pattern]{, [job_id|job_name|pattern]}

resource_list      resource[=value][,resource[=value],...]

user_list        user|@group[,user|@group],...]

resource_attributes   resource,resource,...

wc_cqueue        wildcard expression matching a cluster queue

wc_host         wildcard expression matching a host

wc_hostgroup       wildcard expression matching a hostgroup

wc_qinstance       wc_cqueue@wc_host

wc_qdomain        wc_cqueue@wc_hostgroup

wc_queue         wc_cqueue|wc_qdomain|wc_qinstance
```


## Appendix 1: Frequently Asked Questions [A1.1]

If your question is not found under this section, please see the Troubleshooting/FAQ section found on our website.


### How do I login without having to type my password? [A1.2]

Some of us might find it redundant to type `ssh john@merlot.genomecenter.ucdavis.edu` each time we want to login to merlot or shiraz cluster (like myself). Therefore instead of typing the full path each time, we just need to type one word. For example, "cookie" when we want to login to merlot cluster or "myfavoritewine" when we want to login to shiraz cluster.

In order to do this, we need to set up a password-free access to the cluster and then make an alias. For the example below, let’s say that John want to make pw-free access from his macbook to merlot cluster and alias it with "cookie"

**Part 1: Set up password-free access**

+ On your local computer terminal (not the cluster terminal), type the following:

```bash
ssh-keygen -t rsa
```

Press enter/yes to all questions asked. This will generate a file named `id_rsa.pub` at directory `~/.ssh/`. id_rsa.pub is a key file containing “mark” from your computer.

+ Copy the file to merlot cluster using scp.
```bash
scp ~/.ssh/id_rsa.pub john@merlot.genomecenter.ucdavis.edu:~/
```
+ Then login to merlot cluster with your username and password.

```bash
ssh john@merlot.genomecenter.ucdavis.edu

<enter password>
```

+ Add the content of your "~/id_rsa.pub" to merlot’s authorized_keys list in “~/.ssh/authorized_keys”. 

`cat ~/id_rsa.pub >> ~/.ssh/authorized_keys`

+ You are done! But it is *very* important to delete the id_rsa.pub from the home directory so your account won’t get hacked (id_rsa.pub is basically the key to your merlot account).

`rm ~/id_rsa.pub`

+ Now log out, then try login again:

```bash
ssh john@merlot.genomecenter.ucdavis.edu
```

If you are successful, then the cluster won’t ask for password. If you are not, check if you have done all parts correctly.  Do the same thing to all other cluster computer you want the pw-free access from (e.g. shiraz). You could also set pw-free access from merlot to your local computer, or merlot-to-shiraz, or shiraz-to-merlot, so you could login from anywhere. Just make sure that all id_rsa.pub and its content is only stored in ~/.ssh/authorized_keys and not lying around on your Desktop folder, for example.

**Warning Regarding Passwordless SSH:**

+ *Making SSH passwordless is potentially a very dangerous situation if you don't actually have a password protected screensaver, or if you frequently use a shared computer.*
+ *Anyone who has access to launching a terminal on your computer can potentially delete all of your data on the cluster. It is unlikely, but sometimes things happen accidentally.*
+ *Each user is responsible for their Genome Center account. If it is hacked (randomly) because of a weak password, or if someone causes malicious damage to the cluster (gaining access from a passwordless ssh login), the owner of the account is responsible!*

**Part 2: Generating alias**

Make sure all the pw-free access work fine before setting up alias.

First, edit your .profile in home directory and, if you have it, .bash_profile in home directory:

```bash
nano ~/.profile
```

Then type this at the very bottom of the .profile (or .bash_profile) then save. Be careful to not introduce space anywhere (especially not in between = and ").

```bash
alias cookie="ssh john@merlot.genomecenter.ucdavis.edu"
alias myfavoritewine="ssh john@shiraz.genomecenter.ucdavis.edu"
```

Restart terminal by Alt-Q to quit terminal completely and open new terminal. Then type "cookie" in your terminal, and you should log in to merlot. You can rename "cookie" to anything you want, e.g. merlot or shiraz, by changing the alias name:```bash
alias merlot="ssh john@merlot.genomecenter.ucdavis.edu"
alias shiraz="ssh john@shiraz.genomecenter.ucdavis.edu"
```

## Appendix 2: Contributions and Thanks [A2.1]

The authors contributed to this document in the following ways:

+ Keith Dunaway wrote the bulk of the document
+ Stella Hartono wrote the FAQ section.
+ Keith Bradnam structured the formatting, provided major editing during preliminary versions, converted the document to Markdown and uploaded it to GitHub.

We would also like to thank Paul Lott and Gina Turco for editing after v1.0.


## Appendix 3: Release Notes [A3.1]

v1.1 - 5/14/2013 - Updated answering user questions, changed authors, added thanks section.

v1.0 - 7/30/2012 - Published online for UC Davis Genome Center users to view.

v0.1 - 5/3/2012 - Released manual to genome center employees and students by sharing in Google Docs.

