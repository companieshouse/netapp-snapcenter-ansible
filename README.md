# NetApp SnapCenter Linux Installation

Ansible automation for deploying NetApp SnapCenter on Linux servers.
- Installs required packages and dependencies
- Creates linux users (root, admin, list of users) via Hashicorp Vault
- Configures MySQL directories and permissions
- Sets SELinux contexts for SnapCenter directories
- Downloads SnapCenter installation files from S3
- Runs silent installation of SnapCenter
- Sets up nginx reverse proxy and SSL certificates
- Registers SnapCenter users via Rest API
- Re-secures root user using random password

## Vault requirements

The playbook expects the following secrets in Hashicorp Vault:

```
/applications/{aws_account}-{aws_region}/netapp/snapcenter-linux/
├── root_password_temp      # The temporary root password used during setup
├── admin_password          # The SnapCenter admin user password
└── snapcenter_users        # A list of SnapCenter users
└── s3_resources_bucket     # S3 bucket containing installation files e.g. _bucket.eu-west-2.resources.com/snapcenter_installation_files_
```

## SnapCenter Users

Example contents of snapcenter_users in vault (as above):
```json
{
  "users": [
    {
      "username": "alexjones",
      "password": "securepassword123",
      "snapcenter_role": "AppBackupandCloneAdmin"
    }
  ]
}
```

Default roles available:
SnapCenterAdmin, AppBackupandCloneAdmin, BackupandCloneViewer, InfrastructureAdmin
