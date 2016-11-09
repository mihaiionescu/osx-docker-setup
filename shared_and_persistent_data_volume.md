# Shared & Persistent Data Volume

This should be done through the command line, as Kitematic doesn't have the functionality to manipulate Docker volumes. 

Useful Link:

* http://www.tricksofthetrades.net/2016/03/14/docker-data-volumes/ - see content copy below.

#### Creating Dedicated Data Volume Containers

Create a new Docker volume:

`docker create -v /Users/mihai/.docker/volumes/db_data:/db_data --name db-data-container training/postgres /bin/true`

This will create a container based on Postgres' official Docker container, with a data volume named `db_data` linked to the `/Users/mihai/.docker/volumes/db_data` folder.

***Note***: /bin/true - returns a 0 and does nothing if the command was successful.

##### Steps

create a named container with a dedicated volume attached: `docker create -v /dbdata --name db-data-container training/postgres /bin/true`. The attached volume will be the volume shared with the other containers.


`docker run -d --volumes-from data-store --name database-container-1 training/postgres`

`docker run -d --volumes-from data-store --name perfectward_pg-test_1 perfectward-db-v0.1 postgres`


##### Contents of http://www.tricksofthetrades.net/2016/03/14/docker-data-volumes/ article:

###### 0 - Preamble

Docker containers are a lot more scalable and modular once they have the links in place that allow them to share data. How these links are created and arranged depends upon the arranger, who will choose either to create a file-system data volume or a dedicated data volume container.

This post works thorough these two common choices; data volumes and data volume containers. With consideration of the commands involved in backing up, restoring, and migrating said data volumes.

This is post four on Docker following on from Docker - Daemon Administration and Networking (3). Go back and read the latter half of that post to see how to network containers together so they can properly communicate back and forth, if you need to.

###### 1 – Creating Data Volumes

A “data volume” is a marked directory inside of a container that exists to hold persistent or commonly shared data. Assigning these volumes is done when creating a new container.

Any data already present as part of the Docker image in a targeted volume directory is carried forward into the new container and not lost. This however is not true when mounting a local host directory (covered later) as the data is temporarily covered by the new volume.

You can add a data volume to a container using the -v flag in conjunction with the create or run command. You can use the -v multiple times to mount multiple data volumes.

This next command will create a data volume inside a new container in the /webapp directory.

`docker run -d -P --name test-container -v /webapp training/webapp python app.py`

Data volumes are very useful as once designated and created they can be shared and included as part of other containers. It’s also important to note that any changes to data volumes are not included when you update an image, but conversely data volumes will persist even if the container itself is deleted.

Note: The VOLUME instruction in a Dockerfile will add one or more new volumes to any containers created from the image.

This preservation is due to the fact that data volumes are meant to persist independent of a container’s life cycle. In turn this also means Docker never garbage collects volumes that are no longer in use by a container.

###### 2 – Creating Host Data Volumes

You can instead mount a directory from your Docker daemon’s host into a container; you may have seen this used once or twice in the previous posts.

Mounting a host directory can be useful for testing. For example, you can mount source code inside a container. Then, change the source code and see its effect on the application in real time. The directory on the host must be specified as an absolute path and if the directory doesn’t exist Docker will automatically create it for you. This auto-creation of the host path has been deprecated.

The next example command mounts the host directory /src/webapp into the container at the /opt/webapp directory.

`docker run -d -P --name test-container -v /src/webapp:/opt/webapp training/webapp python app.py`

Some internal rules and behaviours for this process are:

* The targeted container directory must always take an absolute full file-system path.

* The host source directory can be either an absolute path or a name value.

* If the targeted container path already exists inside the container’s image, the host directory mount overlays but does not remove the destination content. Once the mount is removed, the destination content is accessible again.

Docker volumes default to mounting as both a dual read-write mode, but you can set them to mount as read-only if you like.

Here the same /src/webapp directory is linked again but the extra :ro option makes the mount read-only.

`docker run -d -P --name web -v /src/webapp:/opt/webapp:ro training/webapp python app.py`

Note: It’s not possible to mount a host directory using a Dockerfile because by convention built images should be portable and flexible, and a specific host directory might not be available on all potential hosts.

###### 3 – Mounting Individual Host Files

The -v flag used so far can target a single file instead of entire directories from the host machine. This is done by mapping the specific file on each side of the container.

A great interactive example of this that creates a new container and drops you into a bash shell with your bash history from the host, is as follows:

`docker run --rm -it -v ~/.bash_history:/root/.bash_history ubuntu /bin/bash`

Furthermore when you exit the container, the host version of the file will have the the commands typed from the inside of the container - written to the the .bash_history file.

###### 4 – Creating Dedicated Data Volume Containers

A popular practice with Docker data sharing is to create a dedicated container that holds all of your persistent shareable data resources, mounting the data inside of it into other containers once created and setup.

This example taken from the Docker documentation uses the postgres SQL training image as a base for the data volume container.

`docker create -v /data-store --name data-container training/postgres /bin/true`

Note: /bin/true - returns a 0 and does nothing if the command was successful.

The --volumes-from flag is then used to mount the /data-store volume inside of other containers:

`docker run -d --volumes-from data-store --name database-container-1 training/postgres`

This process is repeated for additional new containers:

`docker run -d --volumes-from data-store --name database-container-2  training/postgres`

Be aware that you can use multiple --volumes-from flags in one command to combine data volumes from multiple other dedicated data containers.

An alternative idea is to mount the volumes from each subsequent container to the next, instead of the original dedicated container linking to new ones.

This forms a chain that would begin by using:

`docker run -d --name database-container-3 --volumes-from database-container-2  training/postgres`

Remember that If you remove containers that mount volumes, the volume store and its data will not be deleted. Docker preserves it.

To fully delete a volume from the file-system you must run:

`docker rm -v <container name>`

Where <container name> is “the last container with a reference to the volume.”

Note: There is no cautionary Docker warning provided when removing a container without the -v option. So if a container has volumes mounted the -v must be passed to fully remov them.

**Dangling Volumes**

“Dangling volumes” refers to container volumes that are no longer referenced by a container.

Fortunately there is a command to list out all the stray volumes on a system.

`docker volume ls -f dangling=true`

To remove a volume that’s no longer needed use:

`docker volume rm <volume name>`

Where <volume name> is substituted for the dangling volume name shown in the previous ls output.

###### 5 – Backing Up and Restoring Data Volumes

How are data volumes maintained when it comes to things like backups, restoration, and migration? Well here is one solution that takes care of these necessities by showing how you can achieve this with a dedicated data container.

To backup a volume:

`docker run --rm --volumes-from data-container -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /data-store`

Here’s how the previous command works:

1. The --volumes-from flag creates a new nameless container that mounts the data volume inside data-container you wish to backup.
2. A localhost directory is mounted as /backup . Then tar archives the contents of the /data-store volume to a backup.tar file inside the local /backup directory.
3. The container will be --rm removed once it eventually ends and exits.
We are left with a backup of the /data-store volume on the localhost.

From here you could restore the volume in whatever way you wish.

To restore into a new container run:

`docker run -v /data-store --name data-container-2 ubuntu /bin/bash`

Then extract the backup file contents into the the new container’s data volume:

`docker run --rm --volumes-from data-container-2 -v $(pwd):/backup ubuntu bash -c "cd /data-store && tar -zxvf /backup/backup.tar"`

Now the new container is up and running with the files from the original /data-store volume.

###### 6 – Volume and Data Container Issues

* Orphan Volumes – Referred to as dangling volumes earlier on. These are the leftover untracked volumes that aren’t removed from the system once a container is removed/deleted.

* Security – Other than the usual Unix file permissions and the ability to set read-only or read-write privileges. Docker volumes or data containers have no additional security placed on them.

* Data Integrity – Sharing data using volumes and data containers provides no trsl level of data integrity protection. Data protection features are not yet built into Docker i.e. data snapshot, automatic data replication, automatic backups, etc. So data management has to be handled by the administrator or the container itself.

* External Storage – The current design does not take into account the ability to use a Docker volume spanning from one host to another. They must be on the same host although this needs verifying on my end.

It seems like a large amount of information has been covered here but really only two ideas have been explored. That of singular data volumes and that of the preferred independent data container. There are also new updates to Docker on the horizon as always so some of the issues raised here are hopefully soon to be resolved. The next post on Docker covers building images using Dockerfiles, and likewise with Docker Compose.

* [Docker - Installing and Running](http://www.tricksofthetrades.net/2015/12/23/installing-running-docker/)
* [Docker - Administration and Container Applications](http://www.tricksofthetrades.net/2016/01/07/docker-administration-applications/)
* [Docker - Further Administration and Networking](http://www.tricksofthetrades.net/2016/01/27/docker-further-administration-networking/)
* Docker - Data Volumes and Data Containers - current file

###### 7 - More Information

* [Official Docker Documentation – Manage Data in Containers – Main source material used for this post.](https://docs.docker.com/engine/tutorials/dockervolumes/)
* [Digital Ocean - How To Work with Docker Data Volumes on Ubuntu 14.04 – Breaks down the topic further and has some Nginx logging volume mount examples.](https://www.digitalocean.com/community/tutorials/how-to-work-with-docker-data-volumes-on-ubuntu-14-04)
* [Docker storage 101: How storage works in Docker – Article from April 2015 that goes over the general ideas and practices discussed here.](http://www.computerweekly.com/feature/Docker-storage-101-How-storage-works-in-Docker)