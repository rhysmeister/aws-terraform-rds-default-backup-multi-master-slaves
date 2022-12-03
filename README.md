# aws-terraform-rds-default-backup-multi-master-slaves
An extension of [aws-terraform-rds-aws-backup-multi-master-slaves](https://github.com/rhysmeister/aws-terraform-rds-aws-backup-multi-master-slaves) but using the standard backup features instead of AWS Backup

# Initial Deployment

Ensure the identifier, snapshot_identifier and PITR local variables are as desired.

```bash
terraform apply
```

# Perform a restore without deleting the source RDS instance

First remove any slave instances from the state

```bash
terraform state rm aws_db_instance.rds_slave
```

Remove the master RDS instance

```bash
terraform state rm aws_db_instance.rds1
```

* Update the `identifier` value in the locals section, i.e. rds1 -> rds2
* Set `source_db_instance_identifier`to the source db, i.e. rds1
* Set `use_latest_restorable_time`to true

Perform the restore. This will create the rds2 instance using the backups of rds1...

```bash
terraform apply
```

The rds1 instance, slaves and any backups must be manually deleted as we have deleted the state from Terraform.

To cleanup the rds2 instance...

```bash
terraform destroy
```


# SSM Parameters

* endpoint - The master read/write endpoint. String.
* username - MariaDB root user. String.
* password - MariaDB root user password. String.
* slaves_endpoints - List of read only slaves endpoint. List of strings.

# Notes

* This version of the module introduces a locals variable for identifier so we can choose the name for our db instance - this is to enable restores without deleting our instance.
* Root password is hard-coded in this module. Fix this for any non-trivial usage.
* In order for the slaves to be created automatic backups must be enabled on the master.
* Disassociating a backup does not delete it but it might take some time to appear in RDS > Automated backups > Retained backups. Ensure that delete_automated_backups is set to false before deleting the RDS instance.
* We want to support PITR as well as snapshot restore in this module. The restore_to_point_in_time block needs to be dynamic for this to work properly. Creating the block with all null values causes terraform to crash. 