#[dmake](#dmake)

dmake: A script to make running and managing docker containers easier. By setting some information in the *.dmake files (container_name, name, etc), you can easily run, exec, rm, etc.
## *.dmake files
To initialize a directory for a specific docker container, use:
```
dmake init
```
This will:

1. Run git init to initalize the directory
1. Create a gitignore file containting the user.dmake file (see below)
1. Create user.dmake and run.dmake files

###[user.dmake](#user-dmake)
The user.dmake file is specific to this user's environment and should not be checked into source control. It contains the variables:
* name: the name of the docker image to build
* container_name: the name of the container that will run
* version: the version (tag) of the docker image to build
* rootdir: the root directory containing the Dockerfile for building

###[run.dmake](#run -dmake)
The run.dmake file contains the command line arguments for a run command:
* run_options: The options used in the run command (ports, envirnments, volumes, etc)

Since the run.dmake files is sourced, it can be used to do other things like start dependent containers, etc. See the Advanced usage section for examples.

##[Simple usage](#simple-usage)
Current User: test_user
Current Directory: /some/path/to/tmp

* Create the dmake files
```
$ dmake init
$ cat user.dmake
name="test_user/tmp"
container_name="tmp"
version="latest"
rootdir="/some/path/to/tmp"
$ cat run.dmake
run_options=''
#run_options='-p 8081:80 -v some/path:/container/some/path'
```
* Create a Docker file and build it
```
$ echo FROM ubuntu > Dockerfile
$ dmake ## or dmake build
Sending build context to Docker daemon 3.072 kB
Step 1 : FROM ubuntu
 ---> e9ae3c220b23
Successfully built e9ae3c220b23
```
* quickrun: removes the container when exited (run --rm)
```
$ dmake quickrun
root@d44436304af3:/# exit
exit
```
* run the container
```
$ dmake run
6d0e5f6d52133c06becc14d2cd692745547454f9254d434a76e1382f06cb6432
```
* execute a bash shell on the running container
```
$ dmake exec
root@6d0e5f6d5213:/# exit
exit
```
* remove the running container
```
$ dmake rm
tmp
```

##[Advanced usage](#advanced-usage)
* Start a mysql instance for my app

```
$ git clone git@some/repo.git
...
$ cd repo && cat run.dmake
mysql_pw=changeme
mysql_container=app_mysql
host_port=1235
run_options='-p '$host_port':1235 --link '$mysql_container':mysql -e DPORT='$host_port' -e DHOST='$HOSTNAME' -e MYSQL_PW='$mysql_pw

docker ps | grep app_mysql > /dev/null 2>&1
if test $? -ne 0
then
    docker run -d -e MYSQL_DATABASE=$mysql_container -e MYSQL_ROOT_PASSWORD="$mysql_pw" --name app_mysql mysql
fi

$ dmake init
Writing user.dmake file...
name="test_user/repo"
container_name="repo"
version="latest"
rootdir="/home/test_user/repo"

$dmake run
d20c3eb7d53c7d0bb09c3ea3e8e0051af315c00b4d6c8c388e75d80c03892b0a
f7e349abc539d4755881fa7897fe96663d398cfded67ba03c007134a743f5746
```
##[Forwarding docker commands](#forwarding-docker-commands)
any command that is not a dmake command will be forwarded directly to docker:
```
$dmake ps -aq
d20c3eb7d53
f7e349abc53

$dmake rm -f f7e349abc53
f7e349abc53

$dmake run
8cfded67ba0
```
