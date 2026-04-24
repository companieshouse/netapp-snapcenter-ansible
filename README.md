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


## Playbooks

`0-provision.yml` should be run against a newly created instance running [netapp-snapcenter-ami](https://github.com/companieshouse/netapp-snapcenter-ami)

`1-update-users.yml` can be run against an already provisioned instance to update SnapCenter users to mirror vault

`2-upgrade-version.yml` can be run against an already provisioned instance to upgrade (in place) the version of SnapCenter e.g. 6.1P2 -> 6.2


## Vault requirements

`0-provision.yml` requires the following keys in Hashicorp Vault:

```
/applications/{aws_account}-{aws_region}/netapp/snapcenter-linux/
└── accounts-admin-root     # Contains: root_password_temp, admin_password
```
Optionally:
└── accounts-users          # Contains: snapcenter_users (array)


`1-update-users.yml playbook` requires the following keys in Hashicorp Vault:

```
/applications/{aws_account}-{aws_region}/netapp/snapcenter-linux/
├── accounts-admin-root     # Contains: root_password_temp, admin_password
└── accounts-users          # Contains: snapcenter_users (array)
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
      "snapcenter_roles": [
        "SnapCenterAdmin"
      ]
    },
    {
      "username": "charliejones",
      "password": "987zyx",
      "snapcenter_roles": [
        "App Backup and Clone Admin"
      ]
    },
    {
      "username": "samwilliams",
      "password": "abc123",
      "snapcenter_roles": [
        "Backup and Clone Viewer"
      ]
    },
    {
      "username": "robinevans",
      "password": "789xyz",
      "snapcenter_roles": [
        "Infrastructure Admin",
        "Backup and Clone Viewer"
      ]
    }
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


## How To: Update SnapCenter Version

Download the latest files from NetApp Support:
[https://mysupport.netapp.com/site/products/all/details/snapcenter/downloads-tab](https://mysupport.netapp.com/site/products/all/details/snapcenter/downloads-tab)

You need the binary, signature, and pubkey e.g.
_snapcenter-linux-server.el9.bin_
_snapcenter-linux-server.el9.bin.sig_
_snapcenter_public_key.pub_

Upload these to a new version directory in Shared Services e.g.
_s3://shared-services.eu-west-2.resources.ch.gov.uk/netapp_snapcenter/**6.1P2/snapcenter-linux-server.el9.bin.sig**_

Update your pipeline (netapp-snapcenter) job with the appropriate params var e.g.
`ANSIBLE_EXTRA_VARS: "snapcenter_version=6.2P1"`

Optionally, update `ansible/group_vars/all/snapcenter.yml` to set a new default.
