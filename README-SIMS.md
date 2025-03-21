## This repo was forked from https://github.com/bcgov/biohubbc-backup-container.

Why? At the time, the original repo only supported Postgres version 16.x, and SIMS was using Postgres 17. The pg_dump, etc, tools are not backwards compatible.

When the original repo supports postgres 17, this fork could be decommissioned.

See the commit history for the changes made.

## Commands run to initialize the backup-container.

These follow the instructions from the original repo.

From the root of the biohubbc-backup-container project.

```bash
# Create the build config and image stream
oc -n af2668-tools process -f ./openshift/templates/backup/backup-build.yaml | oc -n af2668-tools create -f -

# Create the backup config
oc -n af2668-prod create configmap backup-conf --from-file=./config/backup.conf

# Label the backup config
oc -n af2668-prod label configmap backup-conf app=biohubbc-db-postgresql-backup

# Create the deployment
oc -n af2668-prod process -f ./openshift/templates/backup/backup-deploy.yaml | oc -n af2668-prod create -f -
```

## Helpful commands

These can also be found in the original README.

### Manually run the backup

In the backup pod, in the terminal, run:

```
./backup.sh -1
```

## Notes

- The backup PVC needs to be large enough to store several compressed copies of the database (based on the backup settings). The default backup settings is 11 copies (6 daily, 4 weekly, 1 monthly). The backup pvc also needs enough extra space to load the backup data initially uncompressed.
  - Total Backup PVC Size = `(11 x (compressed_size)) + (uncompressed_size)`

## TODO

1. There appears to be an issue with the S3 Upload. The backup file is created successfully, and kept in the backup PVC, but the S3 upload step fails. It fails to run the `mc` (minio) command, due to permission issues.
2. There appears to be an issue with the verification. It tries to run a command `run-postgresql` which does not exist. Possibly due to the change in the base docker image.
