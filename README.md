# NetApp SnapCenter Ansible

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
├── s3/
│   └── s3_resources_bucket  # S3 bucket containing SnapCenter installation files
├── accounts-admin-root/
│   ├── admin_password       # SnapCenter admin user password
│   └── root_password_temp   # Temporary root password used during setup
└── accounts-users/
    └── snapcenter_users     # Array of SnapCenter users

```

Example secrets:

**s3/**
```json
{
  "s3_resources_bucket": "bucket.eu-west-1.resources.com/snapcenter_installation_files"
}
```

**accounts-admin-root/**
```json
{
  "root_password_temp": "abc123",
  "admin_password": "xyz789"
}
```

**accounts-users/**
```json
{
  "snapcenter_users": [
    {
      "username": "alexsmith",
      "password": "cba321",
      "snapcenter_role": "SnapCenterAdmin"
    },
    {
      "username": "samjones",
      "password": "987zyx",
      "snapcenter_role": "AppBackupandCloneAdmin"
    }
  ]
}
```

## SnapCenter Roles

SnapCenter has the following pre-defined roles:
- **SnapCenterAdmin** - Full administrative access to SnapCenter
- **AppBackupandCloneAdmin** - Manage application backups and clones (cannot manage hosts, storage connections, or install plug-ins)
- **BackupandCloneViewer** - Read-only access to backups, clones, and reports
- **InfrastructureAdmin** - Manage hosts, storage, provisioning, and plug-in installation (cannot perform backups)

More info: [https://docs.netapp.com/us-en/snapcenter/get-started/rbac-snapcenter.html#permissions-assigned-to-the-pre-defined-snapcenter-roles](https://docs.netapp.com/us-en/snapcenter/get-started/rbac-snapcenter.html#permissions-assigned-to-the-pre-defined-snapcenter-roles)
