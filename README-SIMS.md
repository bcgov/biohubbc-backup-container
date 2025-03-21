## This repo was forked from https://github.com/bcgov/biohubbc-backup-container.

Why? At the time, the original repo only supported Postgres version 16.x, and SIMS was using Postgres 17. The pg_dump, etc, tools are not backwards compatible.

When the original repo supports postgres 17, this fork could be decommissioned.

## Custom modifications that diverge from the base repo:

- Modified the `Dockerfile` to pull from `postgres:17-bullseye`
- Because `postgres:17-bullseye` is not a fedora based image:
  - Added the line `RUN apt-get update && apt-get install -y curl && rm -rf /var/lib/apt/lists/*`
  - Replaced the line `RUN dnf install -y util-linux` with `RUN apt-get update && apt-get install -y util-linux`

## Commands run to initialize the backup-container. These follows the instructions from the original repo:

From the root of the biohubbc-backup-container project.

```bash
# Create the build config and image stream
oc -n af2668-tools process -f ./openshift/templates/backup/backup-build.yaml -p NAME=biohubbc-db-postgresql-backup OUTPUT_IMAGE_TAG=latest | oc -n af2668-tools create -f -

# Create the backup config
oc -n af2668-prod create configmap backup-conf --from-file=./config/backup.conf

# Label the backup config
oc -n af2668-prod label configmap backup-conf app=biohubbc-db-postgresql-backup

# Create the deployment
oc -n af2668-prod process -f ./openshift/templates/backup/backup-deploy.yaml -p NAME=backup-postgres -p IMAGE_NAMESPACE=af2668-tools -p SOURCE_IMAGE_NAME=biohubbc-db-postgresql-backup -p TAG_NAME=prod -p BACKUP_VOLUME_NAME=backup -p BACKUP_VOLUME_SIZE=20Gi -p VERIFICATION_VOLUME_SIZE=20Gi -p ENVIRONMENT_FRIENDLY_NAME='SIMS DB Backups' | oc -n af2668-prod create -f -
```
