# NetApp SnapCenter Ansible

Ansible to apply environment-specific settings on servers using [netapp-snapcenter-ami](https://github.com/companieshouse/netapp-snapcenter-ami) and [netapp-snapcenter-terraform](https://github.com/companieshouse/netapp-snapcenter-terraform)
- Pulls required information from Hashicorp Vault (see requirements below)
- Sets hostname using tag from AWS
- Mounts the data volume (which is a snapshot from the ami build)
- Reboots the instance
- Creates a linux `admin` user with admin_password from Vault
- Adds other required users as specified in Vault (snapcenter_users)
- Authenticates with SnapCenter's Rest API using root_password_temp
- Registers admin and other users with SnapCenter via the API
- Verifies the environment-specific admin account works and has full permissions
- Secures root with a random password; `admin` can be used going forward

## Vault requirements

The playbook expects the following secrets in Hashicorp Vault:

```
/applications/{aws_account}-{aws_region}/netapp/snapcenter-linux/
├── accounts-admin-root/
│   ├── admin_password       # SnapCenter admin user password
│   └── root_password_temp   # Temporary root password used during setup
└── accounts-users/
    └── snapcenter_users     # Array of SnapCenter users

```

Example secrets:

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
      "username": "charliejones",
      "password": "987zyx",
      "snapcenter_role": "App Backup and Clone Admin"
    },
    {
      "username": "samwilliams",
      "password": "abc123",
      "snapcenter_role": "Backup and Clone Viewer"
    },
    {
      "username": "robinevans",
      "password": "789xyz",
      "snapcenter_role": "Infrastructure Admin"
    },
  ]
}
```

## SnapCenter Roles

SnapCenter has the following pre-defined roles:
- **SnapCenterAdmin** - Full administrative access to SnapCenter
- **App Backup and Clone Admin** - Manage application backups and clones (cannot manage hosts, storage connections, or install plug-ins)
- **Backup and Clone Viewer** - Read-only access to backups, clones, and reports
- **Infrastructure Admin** - Manage hosts, storage, provisioning, and plug-in installation (cannot perform backups)

More info: [https://docs.netapp.com/us-en/snapcenter/get-started/rbac-snapcenter.html#permissions-assigned-to-the-pre-defined-snapcenter-roles](https://docs.netapp.com/us-en/snapcenter/get-started/rbac-snapcenter.html#permissions-assigned-to-the-pre-defined-snapcenter-roles)
