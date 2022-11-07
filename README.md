# External Services

This project includes files to set up all external services needed by the
[LumenVox Containers](https://github.com/lumenvox/helm-charts) software stack,
using docker compose.

## Quick Start
```shell
# Get compose file
git clone https://github.com/lumenvox/external-services.git
cd external-services

# Load NFS modules
sudo modprobe nfs
sudo modprobe nfsd

# Set up NFS directory
sudo mkdir /data
sudo chown -R $(whoami): /data

# Get user's UID
id -u

# Update exports.txt to use UID from previous step
nano exports.txt

# Start services
docker-compose up -d
```

## Prerequisites

This is intended to run on a Linux machine. The host machine should have Docker
installed, and it should have access to the Compose tool.

### NFS Modules
For the NFS server to work, the following modules must be loaded:
- nfs
- nfsd

You can find the status of these modules by checking the output of `lsmod`:
```shell
lsmod | grep nfs
```

If either module is not loaded, it should be loaded by running the following:
```shell
sudo modprobe <module>
```

To ensure that these modules are loaded upon reboot, you can create a file
`/etc/modules-load.d/nfs.conf` with the following content:
```text
nfs
nfsd
```

### NFS Port Conflict
The NFS container makes use of port 111, which may be used by other processes.
If this is the case, you may see an error when creating the NFS container:
```text
Error starting userland proxy: listen tcp4 0.0.0.0:111: bind: address already in use
```
If this error is encountered, you can check which process is causing the
conflict by running the following:
```shell
sudo netstat -tulpn | grep 111
```
One culprit found during testing was `rpcbind`. If this is true for you, this
can be solved by stopping and disabling the service:
```shell
sudo systemctl --now disable rpcbind
```

### NFS Volume Permissions
The NFS image will export a directory on your host machine. By default, this is
the `/data` directory, but this can be changed. Regardless of location, you
should verify the following:
- The directory exists.
  - If not, run `sudo mkdir /data`
- The directory is owned by the user that will run docker.
  - If not, run `sudo chown -R $(whoami): /data`

Once the directory has been set up, edit `exports.txt`, replacing `1002` with
your user's UID. This can be obtained by running `id -u`.

## Configuration

### Service Selection
While in the process of setting up dedicated servers for the external services,
you may want to run the stack without all services. If you would like to stop
running a service, you can comment out its section by placing a `#` at the start
of each line. If the service is not the NFS, you should also comment out the
relevant volume at the bottom of the file.

### Credentials
Passwords and other credentials can be managed in the `.env` file. The file is
provided with some defaults which should be changed before moving to production.

### Persistence Location
The NFS container will export the directory named in `docker-compose.yaml`. This
can be seen in the `volumes` section of the `nfs` entry. The default entry is:
```yaml
- '/data:/data:rw'
```
The leftmost `/data` determines the location on the host machine; this can be
configured at will. Be sure that the directory already exists, and be sure that
it is owned by the user that will run docker. This can be accomplished by
running the following:
```shell
sudo chown -R $(whoami): /data
```

The rightmost `/data` determines the mount point on the NFS image. There is
little reason to change this, but if you do change it, make sure to update
`exports.txt`.

The `rw` specifies read/write, which should not be changed.

### Further Configuration
Further configuration of the individual services may be possible, but that is
currently beyond the scope of this project. If you would like more information,
`docker-compose.yaml` includes links to the main pages for each image.

## Stopping and Starting
To start the services, navigate to the project root and run the following:
```shell
docker-compose up -d
```
To stop the services, navigate to the project root and run the following:
```shell
docker-compose down
```
