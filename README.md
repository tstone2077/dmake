# dmake

dmake: A script to make running and managing docker containers. By setting some information in the dconfig file (container_name, name, etc), you can easily run, exec, rm, etc.

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
