# mesos-cli

Standalone commandline tool for interacting with an [Apache Mesos]("http://mesos.apache.com") cluster.

# Why?

Existing CLI tools for Mesos are tightly integrated into their parent projects(e.g. [[0]](https://github.com/apache/mesos/tree/master/src/cli), [[1]](https://github.com/mesosphere/mesos-cli)) and dependent on cumbersome libmesos packages.

`mesos-cli` is a lightweight alternative to these tools, leveraging the excellent [mesos-go]("https://github.com/mesos/mesos-go") library to communicate with Mesos via HTTP. `mesos-cli` additionally aims to add more convenient features than the original toolset.

`mesos-cli` is under development and not ready for use in a production environment.

# Installation

Binary packages are not yet available so you need to install from source.

    go get -u github.com/vektorlab/mesos-cli

 If you don't mind potentially overriding the default `mesos` command you may add an alias:

     echo "alias mesos=mesos-cli" >> $HOME/.bashrc

# Profiles
You can configure "profiles" by creating a JSON file at `~/.mesos-cli.json`:

```json
{
  "profiles": {
    "default": {
      "master": "localhost:5050"
    },
    "production": {
      "master": "production-host:5050"
    },
    "development": {
      "master": "development-host:5050"
    }
  }
}
```

# Usage

`mesos-cli` currently supports the following subcommands:

## run
`mesos run` implements the functionality of the [mesos-execute](https://github.com/apache/mesos/blob/master/src/cli/execute.cpp)
with some additional features.

```
mesos-cli run [OPTIONS] [ARG...]
```

### Example
With Docker containerizer:

```bash
mesos run --tail --image alpine:latest --shell 'date'
....
Wed Dec 14 23:16:50 UTC 2016
....
```

Or with native Mesos containerizer:
```bash
mesos exec --shell 'echo $(date) >> stdout'
```
*note:* Since native mesos containerizer doesn't redirect stdout/stderr by default you need to literally write to a file called `stdout`/`stderr` in the sandbox directory.

### Options

Option | Description
--- | ---
--master="127.0.0.1:5050" | Mesos Master
--task="" | Path to a Mesos TaskInfo JSON file
--param=[] | Docker parameters
-i, --image="" | Docker image to run
-v, --volume=[] | Volume mappings
-p, --ports=[] | Port mappings
-e, --env=[] | Environment Variables
-s, --shell="" | Shell command to execute
-t, --tail=false | Tail command output
-n, --name=mesos-cli | Task Name
-u, --user=root | User to run as
-c, --cpus=0.1 | CPU Resources to allocate
-m, --mem=128.0 | Memory Resources (mb) to allocate
-d, --disk=32.0 | Disk Resources (mb) to allocate
--privileged=false | Give extended privileges to this container
-f, --forcePullImage=false | Always pull the container image

## ps

List currently running tasks on a cluster

```
mesos-cli ps [OPTIONS]
```

### Options

Option | Description
--- | ---
--master="127.0.0.1:5050" | Mesos Master
--limit=100 | maximum number of tasks to return per request
--max=250 | maximum number of tasks to list
--name="" | regular expression to match the TaskId
-a, --all=false | show all tasks
-r, --running=true | show running tasks
--fa, --failed=false | show failed tasks
-k, --killed=false | show killed tasks
-f, --finished=false | show finished tasks

### Example

```bash
mesos ps
```

```
    ID                                        	FRAMEWORK	STATE       	CPUS	MEM	GPUS	DISK
    nginx.d6592dd7-d52a-11e6-bb61-6e9c129136b0	c654f3d1 	TASK_RUNNING	0.1 	64 	0   	0
```

## ls

List the sandbox directory of a task

```
Usage: mesos-cli ls [OPTIONS] TASKID
```

### Options

Option | Description
--- | ---
--master="127.0.0.1:5050" | Mesos Master
-a, --absolute=false | Show absolute file paths

### Example
```bash
mesos ls nginx.d6592dd7-d52a-11e6-bb61-6e9c129136b0
```

```
UID 	GID 	MODE      	MODIFIED                     	SIZE  	PATH
root	root	-rw-r--r--	2017-01-07 22:35:46 -0500 EST	1527  	stderr
root	root	-rw-r--r--	2017-01-08 17:39:03 -0500 EST	642717	stdout
```

## cat

Output the contents of a file

```
mesos-cli cat [OPTIONS] TASKID FILE
```

### Options

Option | Description
--- | ---
--master="127.0.0.1:5050" | Mesos Master
-n, --lines=0 | Output the last N lines
-t, --tail=false | Tail output

### Example
```bash
mesos cat nginx.d6592dd7-d52a-11e6-bb61-6e9c129136b0 stdout
```

```
172.17.0.1 - - [08/Jan/2017:03:35:46 +0000] "GET / HTTP/1.1" 200 612 "http://localhost:8080/ui/" "Mozilla/5.0 (X11;...
...
```

## local

`mesos local` provides a wrapper for launching a local Mesos cluster for development and testing purposes.
It requires that you have Docker installed locally, and uses the [vektorcloud/mesos]("https://github.com/vektorcloud/mesos") image.

```
mesos-cli local [OPTIONS] COMMAND [arg...]
```

### Commands
Command | Description
--- | ---
up | Start the local cluster
down | Stop the local cluster
status | Display the status of the local cluster
rm | Remove the local cluster

## Global Options

Option | Description
--- | ---
--profile | Profile to load
--config | Path to load config from
--level | Level of verbosity

## TODO

  * Support multiple TaskInfos array
  * Improve logging output
  * mesos top
