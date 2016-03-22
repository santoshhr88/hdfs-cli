## HDFS-CLI

HDFS-CLI is an interactive command line shell that makes interacting with the Hadoop Distribted Filesystem (HDFS)
simpler and more intuitive than the standard command-line tools that come with Hadoop. If you're familiar with OS X, Linux, or even Windows terminal/console-based applications, then you are likely familiar with features such as tab completion, command history, and ANSI formatting.

### Binary Package

[Pre-Built Distribution](https://github.com/dstreev/hdfs-cli/releases)

Download the release files to a temp location.  As a root user, chmod +x the 3 shell script files then run the 'setup.sh'.  This will create and install the hdfscli application to your path.

Try it out on a host with default configs:

    hdfscli -a

### Release Notes
#### 2.3.1-SNAPSHOT (in-progress)
    - Added new 'nnstat' function to collect Namenode JMX stats for long term analysis. 
    
See [NN Stat Feature](https://youtu.be/CZxx_BxCX4Y)
    
#### 2.3.0-SNAPSHOT 
    - Added new 'lsp' function.  Consider it an 'ls' PLUS.

#### 2.2.1-SNAPSHOT 

	- External Config Support (See Below)
	    - Supports NN HA
    - Auto Config support using a default config directory.

#### 2.2.0-SNAPSHOT 

	- Setup Script to help deploy  (bin/setup.sh)
	- hdfscli shell script to launch (bin/hdfscli.sh)
	- Support for initialization Script (-i <file>)
	- Kerberos Support (See Below)

#### 2.1.0

	- Added support for create/delete/rename Snapshot

#### 2.0.0

	- Initial Forked Release of P. Taylor Goetz.
	- Update to 2.6.0 Hadoop Libraries
	- Re-wrote Command Implementation to use FSShell as basis for issuing commands.
	- Provide Context Feedback in command window to show local and remote context.
	- Added several missing hdfs dfs commands that didn't exist earlier.

### Building

This project requires the artifacts from https://github.com/dstreev/stemshell , which is a forked enhancement that has added support of processing command line parameters and deals with quoted variables.

### Basic Usage
HDFS-CLI works much like a command-line ftp client: You first establish a connection to a remote HDFS filesystem,
then manage local/remote files and transfers.

To start HDFS-CLI, run the following command:

	java -jar hdfs-cli-full-bin.jar
	
To connect to HDFS:

	hdfs-cli$ connect hdfs://localhost:8020
	
### Command Documentation

Help for any command can be obtained by executing the `help` command:

	help pwd

Note that currently, documentation may be limited.

#### Local vs. Remote Commands
When working within a HDFS-CLI session, you manage both local (on your computer) and remote (HDFS) files. By convention, commands that apply to both local and remote filesystems are differentiated by prepending an `l`
character to the name to denote "local".

For example:

`lls` lists local files in the local current working directory.

`ls` lists remote files in the remote current working directory.

Every HDFS-CLI session keeps track of both the local and remote current working directories.

### Support for Kerberos Clusters

Use the `-k,--kerberos` option when starting hdfs-cli to connect to a secure cluster.  You will need the following:
	- kinit to get a valid Kerberos Ticket for the target cluster.
	- Ensure you've install Unlimited JCE on your host.
	- Provide the REALM you are connecting to.
	- (Optional) Provide the "Namenode" Principal Id (default='nn'). For an HDP cluster, this default value is sufficient to connect.
	- (Optional) Provide the "Namenode" Host (default='_HOST').  For an HDP cluster, this default value is sufficient to connect.

NOTE: If you are using the '-a,--auto' or '-c,--config' option with the supporting Kerberos configurations (in core-site and hdfs-site), do NOT set this 'kerberos' option.  The required values will be extracted from those configurations.	
	
Example Connection parameters.
	
	# Allow Connections to a cluster with a REALM (HDP.LOCAL).  Default nn principal Id and Host will be used.
	hdfscli -k HDP.LOCAL
	
	# Allow Connections to a cluster with a REALM (HDP.LOCAL) and a custom nn principal.  Default Host will be used.
	hdfscli -k HDP.LOCAL,namenode
	
	Allow Connections to a cluster with a REALM (HDP.LOCAL) and a custom nn principal.
	hdfscli -k HDP.LOCAL,namenode,m1.hdp.local

This can be used in conjunction with the 'Startup' Init option below to automatically connect to commonly used clusters.

### Support for External Configurations (core-site.xml,hdfs-site.xml)

Sometimes you need to have specific properties set via 'core-site' and 'hdfs-site' property files.  This includes the configuration settings needed to connect to a High-Availability Namenode Cluster.

The '--config' option takes 1 parameter, a local directory.  This directory should contain hdfs-site.xml and core-site.xml files.  When used, you'll automatically be connected to hdfs and changed to you're hdfs home directory.

Example Connection parameters.

    # Use the hadoop files in the input directory to configure and connect to HDFS.
    hdfscli --config ../mydir

This can be used in conjunction with the 'Startup' Init option below to run a set of commands automatically after the connection is made.  The 'connect' option should NOT be used in the initialization script.

### Support for default configuration

The '-a,--auto' option has no parameters.  When specified, the default /etc/hadoop/conf directory will be searched for core-site and hdfs-site files.  This option is similiar to using: hdfscli --config /etc/hadoop/conf

    # Basic Usage for Default
    hdfscli -a
    
This can be used in conjunction with the 'Startup' Init option below to run a set of commands automatically after the connection is made.  The 'connect' option should NOT be used in the initialization script.

### Startup Initialization Option

Using the option '-i <filename>' when launching the CLI, it will run all the commands in the file.

The file needs to be location in the $HOME/.hdfs-cli directory.  For example:

	# If you're using the helper shell script
	hdfscli -i test
	
	# If you're using the java command
	java -jar hdfs-cli-full-bin.jar -i test
	

Will initialize the session with the command(s) in $HOME/.hdfs-cli/test. One command per line.

The contents could be any set of valid commands that you would use in the cli. For example:

	connect hdfs://m1.hdp.local:8020
	cd user/dstreev
	
Obviously, the first command should be to connect.

### Enhanced Directory Listing (lsp)

Like 'ls', you can fetch many details about a file.  But with this, you can also add information about the file that includes:
- Block Size
- Access Time
- Ratio of File Size to Block
- Datanode information for the files blocks (Host and Block Id)

Use help to get the options:
    
    help lsp

```    
usage: stats [OPTION ...] [ARGS ...]
Options:
 -d,--maxDepth <maxDepth>      Depth of Recursion (default 5), use '-1'
                               for unlimited
 -f,--format <output-format>   Comma separated list of one or more:
                               permissions_long,replication,user,group,siz
                               e,block_size,ratio,mod,access,path,datanode
                               _info (default all of the above)
 -o,--output <output>          Output File (HDFS) (default System.out)
```

When not argument is specified, it will use the current directory.

Examples:
    
    # Using the default format, output a listing to the files in `/user/dstreev/perf` to `/tmp/test.out`
    stats -o /tmp/test.out /user/dstreev/perf

Output with the default format of:

    permissions_long,replication,user,group,size,block_size,ratio,mod,access,path,datanode_info
    
```
   rw-------,3,dstreev,hdfs,429496700,134217728,3.200,2015-10-24 12:26:39.689,2015-10-24 12:23:27.406,/user/dstreev/perf/teragen_27/part-m-00004,10.0.0.166,d2.hdp.local,blk_1073747900
   rw-------,3,dstreev,hdfs,429496700,134217728,3.200,2015-10-24 12:26:39.689,2015-10-24 12:23:27.406,/user/dstreev/perf/teragen_27/part-m-00004,10.0.0.167,d3.hdp.local,blk_1073747900
   rw-------,3,dstreev,hdfs,33,134217728,2.459E-7,2015-10-24 12:27:09.134,2015-10-24 12:27:06.560,/user/dstreev/perf/terasort_27/_partition.lst,10.0.0.166,d2.hdp.local,blk_1073747909
   rw-------,3,dstreev,hdfs,33,134217728,2.459E-7,2015-10-24 12:27:09.134,2015-10-24 12:27:06.560,/user/dstreev/perf/terasort_27/_partition.lst,10.0.0.167,d3.hdp.local,blk_1073747909
   rw-------,1,dstreev,hdfs,543201700,134217728,4.047,2015-10-24 12:29:28.706,2015-10-24 12:29:20.882,/user/dstreev/perf/terasort_27/part-r-00002,10.0.0.167,d3.hdp.local,blk_1073747920
   rw-------,1,dstreev,hdfs,543201700,134217728,4.047,2015-10-24 12:29:28.706,2015-10-24 12:29:20.882,/user/dstreev/perf/terasort_27/part-r-00002,10.0.0.167,d3.hdp.local,blk_1073747921
```
With the file in HDFS, you can build a hive table on top of it to do some analysis.  One of the reasons I created this was to be able to review a directory used by some process and get a baring on the file construction and distribution across the cluster.  

#### Use Cases
- The ratio can be used to identify files the are below the block size (small files).
- With the Datanode information, you can determine if a dataset is hot-spotted on a cluster.  All you need to a full list of hosts to join the results with.

### Available Commands

#### Common Commands
	connect		connect to a remote HDFS instance
	help		display help information
	put			upload local files to the remote HDFS
    get			(todo) retrieve remote files from HDFS to Local Filesystem

#### Remote (HDFS) Commands
	cd		 change current working directory
	ls		 list directory contents
	rm		 delete files/directories
	pwd		 print working directory path
	cat		 print file contents
	chown	 change ownership
	chmod	 change permissions
	chgrp	 change group
	head	 print first few lines of a file
	mkdir	 create directories
	count    Count the number of directories, files and bytes under the paths that match the specified file pattern.
	stat     Print statistics about the file/directory at <path> in the specified format.
	tail     Displays last kilobyte of the file to stdout.
	text	 Takes a source file and outputs the file in text format.
	touchz Create a file of zero length.
	usage	 Return the help for an individual command.

	createSnapshot	Create Snapshot
	deleteSnapshot	Delete Snapshot
	renameSnapshot	Rename Snapshot

#### Local (Local File System) Commands
	lcd		 change current working directory
	lls		 list directory contents
	lrm		 delete files/directories
	lpwd	 print working directory path
	lcat	 print file contents
	lhead	 print first few lines of a file
	lmkdir	 create directories

### Known Bugs/Limitations

* No support for paths containing spaces
* No support for Windows XP
* Path Completion for chown, chmod, chgrp is broken

### Road Map

- Support input script
- Support input variables
- Expand to support Extended ACL's (get/set)
- Add Support for setrep
- HA Commands
	- NN and RM
	




