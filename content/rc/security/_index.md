---
Title: Security
description:
weight: 51
alwaysopen: false
categories: ["RC"]
aliases: /rc/administration/security
         /rc/administering/designing-production/security/
         /rc/security/admin-console-security/_index.md
         /rc/security/admin-console-security/
         /rc/security/admin-console-security.md
         /rc/security/database-security/_index.md
         /rc/security/database-security/
         /rc/security/database-security.md
---

Redis Enterprise Cloud provides a number of ways to secure subscriptions and databases.

As a Redis Cloud user, consider these security domains:

- The [admin console]({{<relref "/rc/security/admin-console-security/_index.md">}})
- [Your databases]({{<relref "/rc/security/database-security/_index.md">}})
- The [Redis Cloud API]({{<relref "/rc/api/get-started/enable-the-api.md">}})

First, you might want to review our [shared responsibility model]({{<relref "/rc/security/shared-responsibility-model.md">}}) for security.

## Admin console security

The admin console is the web application you use to manage your Redis Cloud deployments. 

Secure access to the admin console by:

- Assigning appropriate roles to [team members with access]({{<relref "/rc/security/access-management#team-management-roles">}}).

- Enabling [multi-factor authentication]({{<relref "/rc/security/multi-factor-authentication">}}).

## Database security

You have several options when it comes to securing your Redis Cloud databases. For more information, see [Cloud database security]({{<relref "/rc/security/database-security/_index.md">}}). Options include:

- [Encryption at rest]({{<relref "/rc/security/encryption-at-rest.md">}})
- [Role-based access control]({{<relref "/rc/security/database-security/passwords-users-roles.md">}})
- [Network security]({{<relref "/rc/security/database-security/network-security.md">}})
- [TLS]({{<relref "/rc/security/database-security/tls-ssl.md">}})
- [Network security]({{<relref "/rc/security/database-security/network-security.md">}}) using
[VPC peering]({{<relref "/rc/security/vpc-peering.md">}}) and [CIDR whitelist]({{<relref "/rc/security/cidr-whitelist.md">}})

## API security

The Redis Enterprise Cloud API allows you to programmatically administer your subscriptions and database deployments. This API is disabled by default. When you [enable the API]({{<relref "/rc/api/get-started/enable-the-api.md">}}), you can then [manage the API keys]({{<relref "/rc/api/get-started/manage-api-keys.md">}}) for all owners of your Redis Cloud account. For an overview of the security features of the API, see the [API authentication documentation]({{<relref "/rc/api/get-started/_index.md">}}).
