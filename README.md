# dmake

dmake: A script to make running and managing docker containers. By setting some information in the dconfig file (container_name, name, etc), you can easily run, exec, rm, etc.
## dconfig
The dconfig file is a filed that is sourced and sets up some variables that dmake uses. To generate a default dconfig, run the command
```
dmake dconfig
```
The variables that are set in the dconfig include:
* name: the name of the docker image to build
* container_name: the name of the container that will run
* version: the version (tag) of the docker image to build
* rootdir: the root directory containing the Dockerfile for building
* run_options: The options used in the run command (ports, envirnments, volumes, etc)

Since the dconfig is sourced, it can be used to do other things like start dependent containers, etc. See the Advanced usage section for examples.

## Simple usage
Current User: test_user
Current Directory: /some/path/to/tmp

* Create a dconfig file
```
$ dmake dconfig
$ cat !$
name="test_user/tmp"
container_name="tmp"
version="latest"
rootdir="/some/path/to/tmp"
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

## Advanced usage
* Start a mysql instance for my app

```
$ git clone git@some/repo.git
...
$ cd repo && cat dconfig
mysql_pw=changeme
host_port=1235
run_options='-p '$host_port':1235 --link app_mysql:mysql -e DPORT='$host_port' -e DHOST='$HOSTNAME' -e MYSQL_PW='$mysql_pw

docker ps | grep app_mysql > /dev/null 2>&1
if test $? -ne 0
then
    docker run -d -e MYSQL_DATABASE=app_mysql -e MYSQL_ROOT_PASSWORD="$mysql_pw" --name app_mysql mysql
fi

$ dmake dconfig
/home/test_user/repo/dconfig already exists.
Would you like to prepend the default configuration [y/n]? y
name="test_user/repo"
container_name="repo"
version="latest"
rootdir="/home/test_user/repo"
run_options=''
#run_options='-p 8081:80 -v some/path:/container/some/path'
mysql_pw=changeme
host_port=1235
run_options='-p '$host_port':1235 --link app_mysql:mysql -e DPORT='$host_port' -e DHOST='$HOSTNAME' -e MYSQL_PW='$mysql_pw

docker ps | grep app_mysql > /dev/null 2>&1
if test $? -ne 0
then
    docker run -d -e MYSQL_DATABASE=app_mysql -e MYSQL_ROOT_PASSWORD="$mysql_pw" --name app_mysql mysql
fi
```

