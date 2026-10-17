# Phase 7: Active Directory Organizational Structure and Security Groups

## Objective

Organize Active Directory objects into a logical OU hierarchy and establish security groups to support administrative delegation and role-based access control.

## 1. Active Directory Structure

The current OU structure within `lab.local` is organized as follows:

```text
lab.local
│
├── Admin
│   ├── Privileged Users
│   ├── Servers
│   └── Service Accounts
│
├── Corporate
│   ├── IT
│   ├── HR
│   ├── Finance
│   ├── Management
│   └── Users
│
├── Workstations
├── Groups
└── Disabled Objects
```

## 2. Security Groups

The following security groups have been established:

- `Domain Admins - Lab`
- `Helpdesk - Lab`
- `IT - Lab`
- `HR - Lab`
- `Finance - Lab`
- `Management - Lab`
- `Server Admins - Lab`
- `Workstation Admins - Lab`

These groups provide a foundation for organizing permissions by department and administrative responsibility.

## Phase Status

**Completed:** Active Directory OU structure and security groups established.

**Next steps:** Configure Group Policy, assign appropriate group memberships and permissions, and validate administrative separation and least-privilege access.
