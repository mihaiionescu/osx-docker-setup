

| Command | Description |
| -- | -- |
| `docker volume ls` | list all Docker volumes in the virtual machine |
| `docker volume ls -f dangling=true` | list all unused volumes |
| `docker volume rm <volume_id>` | remove Docker volume by ID |
| `docker images` | list all local Docker images |
| `docker rmi <image_id>` | remove a local Docker image |
| `docker ps` | list Docker containers in use |
| `docker ps -a` | list all local Docker containers, including the inactive ones. |
| `docker rm -v <container_id>` | remove Docker container and it's associated volume |
| `docker rm <container_id>` | remove Docker container **without** it's associated volume |
| **`docker run`** | 1:11 |
| `docker run [OPTIONS] <image_name> ` | 1:12 |
| 0:13 | 1:13 |
| 0:14 | 1:14 |
| 0:15 | 1:15 |
| 0:16 | 1:16 |
| 0:17 | 1:17 |
| 0:18 | 1:18 |
| 0:19 | 1:19 |
| 0:20 | 1:20 |
| 0:21 | 1:21 |
| 0:22 | 1:22 |

`docker run --name perfectward_pg-9.4.10 -v "$PWD/Users/mihai/Projects/bolt_partners/perfectward_db_dumps/barnsley_data/"://barnsley_data perfectward-db-v0.1 -d postgres`

**TODO: Add the `docker run` options with a few examples!**