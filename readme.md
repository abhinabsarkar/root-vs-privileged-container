# Privileged container versus Root user in Docker
## Root user
By default, Docker daemon runs as root on the host machine, so by default all containers also run as root. Although containers are isolated using namespaces, a malicious user can escalate the privileges of a container & compromise the whole cluster. Always recommended to run as a non-root user.

### A simple example of an attack
The below example is run on docker desktop for windows installed on windows 10 running wsl2.
```bash
# Create a file as root user 
mkdir /mnt/d/temp 
sudo nano /mnt/d/temp/filebyroot.txt
# Modify the permissions so that no one other than the root has any access on it
sudo chmod 600 /mnt/d/temp/filebyroot.txt
# Check permission
ls -lrt /mnt/d/temp/filebyroot.txt
```
Create a Dockerfile 
```bash
# Sample dockerfile to show root user privileges
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python python3-pip wget
RUN pip install Flask
```
Build a docker image with root user using the above Dockerfile
```bash
# Build an image
docker build -t image_with_root_user .
# Create a container mounting the whole root filesystem of the host
docker run -v /:/host -it --name container_with_root_user image_with_root_user 
```
Once in the container, by doing ls you can see that you have the whole host file system in the host directory. As you are root now, you can access root-owned files.
```bash
# For a WSL based system, access the path /host/mnt
cd /host/mnt/d/temp
# You can access the file on the host system
cat filebyroot.txt
```
When you exit the container, you can do all the damage you want to the host machine because you have the root privilege even though you’re not root.
### Mitigation
create a non root user in the container and set it as the current user.
```bash
# Sample dockerfile to show root user privileges
FROM ubuntu:latest
RUN apt-get update
RUN apt-get install -y python python3-pip wget
RUN pip install Flask
# Add a nonroot user named as nonroot
RUN useradd -ms /bin/bash nonroot
#Run Container as nonroot
USER nonroot
```
Build a docker image with nonroot user using the above Dockerfile
```bash
# Build an image with non root user
docker build -t image_with_nonroot_user .
# Create a non root user container 
docker run -v /:/host -it --name container_with_nonroot_user image_with_nonroot_user
```
Once in the container, by doing ls you can see that you have the whole host file system in the host directory. As you are nonroot user now, you can't access root-owned files.
```bash
# For a WSL based system, access the path /host/mnt
cd /host/mnt/d/temp
# You can access the file on the host system
cat filebyroot.txt

cat: filebyroot.txt: Permission denied
```

## Privileged container
Privileged is a special flag you can set at runtime specifically to allow a Docker container to break free from its namespaces and access the entire system directly. Always recommended to set the privileged flag as false.

### Docker Privileged Example
To run an Ubuntu container (interactively) in privileged mode
```bash
docker run -it --privileged --name privileged_container image_with_root_user
```
To test whether the container has access to the host, you can try to create a temporary file system (tmpfs) and mount it to /mnt
```bash
mount -t tmpfs none /mnt
# list the disk space statistics (in human readable format)
df -h
# The newly created file system should appear on the list
root@193174929cf5:/# df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay         251G  6.0G  233G   3% /
tmpfs            64M     0   64M   0% /dev
tmpfs           488M     0  488M   0% /sys/fs/cgroup
shm              64M     0   64M   0% /dev/shm
/dev/sdc        251G  6.0G  233G   3% /etc/hosts
none            488M     0  488M   0% /mnt
# cleanup - remove the mount
umount none
```

## References
* [Privileged vs Root in docker](https://www.cloudsavvyit.com/5211/privileged-vs-root-in-docker-whats-the-difference/)
* [Privileged container](https://phoenixnap.com/kb/docker-privileged)