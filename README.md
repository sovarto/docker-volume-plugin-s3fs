# Introduction

Docker Swarm has no inherent solution for sharing data in volumes across nodes. Especially for small-ish files that don't change at high frequency, a good solution is [S3FS](https://github.com/s3fs-fuse/s3fs-fuse), a FUSE-based filesystem backed by S3.  
Encapsulating this into a Docker Volume Plugin allows using S3 as the backing store of Docker Volumes.

This plugin is based on [prior art](https://github.com/marcelo-ochoa/docker-volume-plugins), but focuses only on the S3FS plugin and enhances it, mainly by allowing to specify the target bucket per volume instead of per plugin install.

# Usage

Install the plugin on every node of the Docker Swarm cluster:

    docker plugin install sovarto/s3fs-volume-plugin:3.0.0 --alias s3fs:3.0.0 --grant-all-permissions --disable
    docker plugin set s3fs:3.0.0 AWSACCESSKEYID=<Your AWS Access Key>
    docker plugin set s3fs:3.0.0 AWSSECRETACCESSKEY=<Your AWS Secret Access Key>
    docker plugin enable s3fs:3.0.0

To create a volume backed by S3, create the volume with the driver "s3fs:3.0.0" and the following two options:

- bucket: The S3 bucket to store the data in
- folder: The folder inside the S3 bucket in which the data should be placed

Example:

    docker volume create --driver s3fs:3.0.0 --opt bucket=replicated-storage-sovarto-cluster-prod --opt folder=test test-volume

Please note:  
If the volume is created on a manager node and used in a Docker Swarm Service, then it doesn't need to be created manually on every node.  
However, if it is not used in a service but only in individual containers, it needs to be created on each node it should be used. Otherwise, Docker will automatically create a new local volume with that name.