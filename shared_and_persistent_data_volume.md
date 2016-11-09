# Shared & Persistent Data Volume

This should be done through the command line, as Kitematic doesn't have the functionality to manipulate Docker volumes. 

Useful Link:

* http://www.tricksofthetrades.net/2016/03/14/docker-data-volumes/

#### Creating Dedicated Data Volume Containers

Create a new Docker volume:

`docker create -v /Users/mihai/.docker/volumes/db_data:/db_data --name db-data-container training/postgres /bin/true`

This will create a container based on Postgres' official Docker container, with a data volume named `db_data` linked to the `/Users/mihai/.docker/volumes/db_data` folder.

***Note***: /bin/true - returns a 0 and does nothing if the command was successful.

`docker run -d --volumes-from data-store --name database-container-1 training/postgres`

``