# Domain, Project, User & Role

## Domain

To mapping user with resource, OpenStack use terms Project, User and Roles:

"Traditionally, the resource mapping could be summed up by saying that “a user has a role in a project”. The user is typically an individual that is communicating with the cloud services, submitting requests to provision and destroy infrastructure. A role is an arbitrary bit of metadata that is used to influence that user’s authority within the environment. The project is a container used to group and isolate resources from one another. With the original mapping model, when the admin role was applied to a user, the user would become a cloud administrator, rather than just a project administrator, as intended.[1]"

In general, one OpenStack Cloud Enviroment often has only two roles: admin and user. With only two roles above, we have a problem: We can not distinguish between "cloud" administrator and "user" administrator. Any person who has admin role become cloud administrator.

With domain, we can solve above problem. Only user has admin role in specific domain become cloud administrator. Other user who has admin role but in a another domain become administrator of that domain:

```json
    "admin_required": "role:admin",
    "cloud_admin": "role:admin and (is_admin_project:True or domain_id:admin_domain_id)",
    "service_role": "role:service",
    "service_or_admin": "rule:admin_required or rule:service_role",
    "owner": "user_id:%(user_id)s or user_id:%(target.token.user_id)s",
    "admin_or_owner": "(rule:admin_required and domain_id:%(target.token.user.domain.id)s) or rule:owner",
    "admin_and_matching_domain_id": "rule:admin_required and domain_id:%(domain_id)s",
    "service_admin_or_owner": "rule:service_or_admin or rule:owner",
?
    "default": "rule:admin_required",

    "identity:get_region": "",
    "identity:list_regions": "",
    "identity:create_region": "rule:cloud_admin",
    "identity:update_region": "rule:cloud_admin",
    "identity:delete_region": "rule:cloud_admin",

    "identity:get_service": "rule:admin_required",
    "identity:list_services": "rule:admin_required",
    "identity:create_service": "rule:cloud_admin",
    "identity:update_service": "rule:cloud_admin",
    "identity:delete_service": "rule:cloud_admin",

    "identity:get_endpoint": "rule:admin_required",
    "identity:list_endpoints": "rule:admin_required",
    "identity:create_endpoint": "rule:cloud_admin",
    "identity:update_endpoint": "rule:cloud_admin",?
    "identity:delete_endpoint": "rule:cloud_admin",

    "identity:get_registered_limit": "",
    "identity:list_registered_limits": "",
    "identity:create_registered_limits": "rule:admin_required",
    "identity:update_registered_limits": "rule:admin_required",
    "identity:delete_registered_limit": "rule:admin_required",

    "identity:get_limit": "",
    "identity:list_limits": "",
    "identity:create_limits": "rule:admin_required",
    "identity:update_limits": "rule:admin_required",
    "identity:delete_limit": "rule:admin_required",

    "identity:get_domain": "rule:cloud_admin or rule:admin_and_matching_domain_id or token.project.domain.id:%(target.domain.id)s",
    "identity:list_domains": "rule:cloud_admin",
    "identity:create_domain": "rule:cloud_admin",
    "identity:update_domain": "rule:cloud_admin",
    "identity:delete_domain": "rule:cloud_admin",
```

With domain term, a user in Cloud environment must only in a specific domain and has only permissions in that domain. A user cannot exist in two different domains at the same time. Domain administrator has permission in only domain where he/she is in:

```json
    "admin_and_matching_target_project_domain_id": "rule:admin_required and domain_id:%(target.project.domain_id)s",
    "admin_and_matching_project_domain_id": "rule:admin_required and domain_id:%(project.domain_id)s",
    "identity:get_project": "rule:cloud_admin or rule:admin_and_matching_target_project_domain_id or project_id:%(target.project.id)s",
    "identity:list_projects": "rule:cloud_admin or rule:admin_and_matching_domain_id",
    "identity:list_user_projects": "rule:owner or rule:admin_and_matching_domain_id",
    "identity:create_project": "rule:cloud_admin or rule:admin_and_matching_project_domain_id",
    "identity:update_project": "rule:cloud_admin or rule:admin_and_matching_target_project_domain_id",
    "identity:delete_project": "rule:cloud_admin or rule:admin_and_matching_target_project_domain_id",
    "identity:create_project_tag": "rule:admin_required",
    "identity:delete_project_tag": "rule:admin_required",
    "identity:get_project_tag": "rule:admin_required",
    "identity:list_project_tags": "rule:admin_required",
    "identity:delete_project_tags": "rule:admin_required",
    "identity:update_project_tags": "rule:admin_required",

    "admin_and_matching_target_user_domain_id": "rule:admin_required and domain_id:%(target.user.domain_id)s",
    "admin_and_matching_user_domain_id": "rule:admin_required and domain_id:%(user.domain_id)s",
    "identity:get_user": "rule:cloud_admin or rule:admin_and_matching_target_user_domain_id or rule:owner",
    "identity:list_users": "rule:cloud_admin or rule:admin_and_matching_domain_id",
    "identity:create_user": "rule:cloud_admin or rule:admin_and_matching_user_domain_id",
    "identity:update_user": "rule:cloud_admin or rule:admin_and_matching_target_user_domain_id",
    "identity:delete_user": "rule:cloud_admin or rule:admin_and_matching_target_user_domain_id",
```

"In Keystone v3 (Grizzly release), Domains encapsulates users and projects into logical entities that can represent accounts, organizations, etc. Currently there is no capability or mechanism to manage or enforce quotas at domain level. Assigning or updating quota values or limits to a domain will allow the cloud administrator to evaluate domain lists and consumption. In order to achieve these capabilities it will be required to implement quota management for Keystone domains. The goal of this blueprint is to support quotas at the OpenStack Domain level. The design of the feature models, as far as possible, the style of project quotas. This blueprint is a contribution from CERN, BARC and Hewlett-Packard."[3]

When we create a user, we need to specify which domain this user will be in:

```bash
    openstack user create
        [--domain <domain>]
        [--project <project> [--project-domain <project-domain>]]
        [--password <password>]
        [--password-prompt]
        [--email <email-address>]
        [--description <description>]
        [--enable | --disable]
        [--or-show]
        <user-name>
```

In general, a domain administrator can create project, user, up?date user role, update project quota... in target domain, when cloud administrator has fulll permission in all domain in cloud, adddition create domain, delete domain, create role, create endpoint...

With domain term, we have another advantages:

- Support for overlapping resource names such as usernames
- The ability for separate organizations to leverage different backends. For example, one can be SQL based, while another can be LDAP based
- ...

The advantage of having user groups is that by assigning roles to a user group, all users in the group get permissions of the roles.

**Summary: Identity Concept in OpenStack** [5]:

* Authentication
    * The process of confirming the identity of a user. To confirm an incoming request, OpenStack Identity validates a set of credentials users supply. Initially, these credentials are a user name and password, or a user name and API key. When OpenStack Identity validates user credentials, it issues an authentication token. Users provide the token in subsequent requests.
* Credentials
    * Data that confirms the identity of the user. For example, user name and password, user name and API key, or an authentication token that the Identity service provides.
* Domain
    * An Identity service API v3 entity. Domains are a collection of projects and users that define administrative boundaries for managing Identity entities. Domains can represent an individual, company, or operator-owned space. They expose administrative activities directly to system users. Users can be granted the administrator role for a domain. A domain administrator can create projects, users, and groups in a domain and assign roles to users and groups in a domain.
* Endpoint
    * A network-accessible address, usually a URL, through which you can access a service. If you are using an extension for templates, you can create an endpoint template that represents the templates of all consumable services that are available across the regions.
* Group
    * An Identity service API v3 entity. Groups are a collection of users owned by a domain. A group role, granted to a domain or project, applies to all users in the group. Adding or removing users to or from a group grants or revokes their role and authentication to the associated domain or project.
* OpenStackClient
    * A command-line interface for several OpenStack services including the Identity API. For example, a user can run the openstack service create and openstack endpoint create commands to register services in their OpenStack installation.
* Project
    * A container that groups or isolates resources or identity objects. Depending on the service operator, a project might map to a customer, account, organization, or tenant.
* Region
    * An Identity service API v3 entity. Represents a general division in an OpenStack deployment. You can associate zero or more sub-regions with a region to make a tree-like structured hierarchy. Although a region does not have a geographical connotation, a deployment can use a geographical name for a region, such as us-east.
* Role
    * A personality with a defined set of user rights and privileges to perform a specific set of operations. The Identity service issues a token to a user that includes a list of roles. When a user calls a service, that service interprets the user role set, and determines to which operations or resources each role grants access.
* Service
    * An OpenStack service, such as Compute (nova), Object Storage (swift), or Image service (glance), that provides one or more endpoints through which users can access resources and perform operations.
* Token
    * An alpha-numeric text string that enables access to OpenStack APIs and resources. A token may be revoked at any time and is valid for a finite duration. While OpenStack Identity supports token-based authentication in this release, it intends to support additional protocols in the future. OpenStack Identity is an integration service that does not aspire to be a full-fledged identity store and management solution.
* User
    * A digital representation of a person, system, or service that uses OpenStack cloud services. The Identity service validates that incoming requests are made by the user who claims to be making the call. Users have a login and can access resources by using assigned tokens. Users can be directly assigned to a particular project and behave as if they are contained in that project.

## Roles

[4] Role Definitions basically creates a named role within the system that can then be assigned to one or more users. A Role Assignment maps subject (user/group) to a role, apart from role mapping it is also used to map subject (user/group) to a tenant. Role Definition Role definitions are available for use across all domains (global) or for a single domain. For example, Nova may may define a global role 'sysadmin' Alternatively, a domain administrator may create a custom role named 'FooBarAdmin' for use only within her domain. Role definitions are handled by Role Operations in the example API shown below. If the role object includes a domainId, then it means the definition is scoped to a specific domain.

- Created role modify access permisison by change service **policy.json** file:

[https://docs.openstack.org/keystone/pike/admin/identity-concepts.html#user-management](https://docs.openstack.org/keystone/pike/admin/identity-concepts.html#user-management)

```txt
The /etc/[SERVICE_CODENAME]/policy.json file controls the tasks that users can perform for a given service. For example, the /etc/nova/policy.json file specifies the access policy for the Compute service, the /etc/glance/policy.json file specifies the access policy for the Image service, and the /etc/keystone/policy.json file specifies the access policy for the Identity service.

The default policy.json files in the Compute, Identity, and Image services recognize only the admin role. Any user with any role in a project can access all operations that do not require the admin role.
```

**Example**:

To restrict users from performing operations in, for example, the Compute service, you must create a role in the Identity service and then modify the /etc/nova/policy.json file so that this role is required for Compute operations.

For example, the following line in the /etc/cinder/policy.json file does not restrict which users can create volumes:

```txt
"volume:create": "",
```?

If the user has any role in a project, he can create volumes in that project.

To restrict the creation of volumes to users who have the `compute-user` role in a particular project, you add `"role:compute-user"`:

```txt
"volume:create": "role:compute-user",
```

To restrict all Compute service requests to require this role, the resulting file looks like:

```txt
{
   "admin_or_owner": "role:admin or project_id:%(project_id)s",
   "default": "rule:admin_or_owner",
   "compute:create": "role:compute-user",
   "compute:create:attach_network": "role:compute-user",
   "compute:create:attach_volume": "role:compute-user",
   "compute:get_all": "role:compute-user",
   "compute:unlock_override": "rule:admin_api",
   "admin_api": "role:admin",
   "compute_extension:accounts": "rule:admin_api",
   "compute_extension:admin_actions": "rule:admin_api",
   "compute_extension:admin_actions:pause": "rule:admin_or_o?wner",
   "compute_extension:admin_actions:unpause": "rule:admin_or_owner",
   "compute_extension:admin_actions:suspend": "rule:admin_or_owner",
   "compute_extension:admin_actions:resume": "rule:admin_or_owner",
   "compute_extension:admin_actions:lock": "rule:admin_or_owner",
   "compute_extension:admin_actions:unlock": "rule:admin_or_owner",
   "compute_extension:admin_actions:resetNetwork": "rule:admin_api",
   "compute_extension:admin_actions:injectNetworkInfo": "rule:admin_api",
   "compute_extension:admin_actions:createBackup": "rule:admin_or_owner",
   "compute_extension:admin_actions:migrateLive": "rule:admin_api",
   "compute_extension:admin_actions:migrate": "rule:admin_api",
   "compute_extension:aggregates": "rule:admin_api",
   "compute_extension:certificates": "role:compute-user",
   "compute_extension:cloudpipe": "rule:admin_api",
   "compute_extension:console_output": "role:compute-user",
   "compute_extension:consoles": "role:compute-user",
   "compute_extension:createserverext": "role:compute-?user",
   "compute_extension:deferred_delete": "role:compute-user",
   "compute_extension:disk_config": "role:compute-user",
   "compute_extension:evacuate": "rule:admin_api",
   "compute_extension:extended_server_attributes": "rule:admin_api",
   "compute_extension:extended_status": "role:compute-user",
   "compute_extension:flavorextradata": "role:compute-user",
   "compute_extension:flavorextraspecs": "role:compute-user",
   "compute_extension:flavormanage": "rule:admin_api",
   "compute_extension:floating_ip_dns": "role:compute-user",
   "compute_extension:floating_ip_pools": "role:compute-user?",
   "compute_extension:floating_ips": "role:compute-user",
   "compute_extension:hosts": "rule:admin_api",
   "compute_extension:keypairs": "role:compute-user",
   "compute_extension:multinic": "role:compute-user",
   "compute_extension:networks": "rule:admin_api",
   "compute_extension:quotas": "role:compute-user",
   "compute_extension:rescue": "role:compute-user",
   "compute_extension:security_groups": "role:compute-user",
   "compute_extension:server_action_list": "rule:admin_api",
   "compute_extension:server_diagnostics": "rule:admin_api",
   "compute_extension:simple_tenant_usage:show": "rule:admin_or_owner",
   "compute_extension:simple_tenant_usage:list": "rule:admin_api",
   "compute_extension:users": "rule:admin_api",
   "compute_extension:virtual_interfaces": "role:compute-user",
   "compute_extension:virtual_storage_arrays": "role:compute-user",
   "compute_extension:volumes": "role:compute-user",
   "compute_extension:volume_attachments:index": "role:compute-user",
   "compute_extension:volume_attachments:show": "role:compute-user",
   "compute_extension:volume_attachments:create": "role:compute-user",
   "compute_extension:volume_attachments:delete": "role:compute-user",
   "compute_extension:volumetypes": "role:compute-user",
   "volume:create": "role:compute-user",
   "volume:get_all": "role:compute-user",
   "volume:get_volume_metadata": "role:compute-user",
   "volume:get_snapshot": "role:compute-user",
   "volume:get_all_snapshots": "role:compute-user",
   "network:get_all_networks": "role:compute-user",
   "network:get_network": "role:compute-user",
   "network:delete_network": "role:compute-user",
   "network:disassociate_network": "role:compute-user",
   "network:get_vifs_by_instance": "role:compute-user",
   "network:allocate_for_instance": "role:compute-user",
   "network:deallocate_for_instance": "role:compute-user",
   "network:validate_networks": "role:compute-user",
   "network:get_instance_uuids_by_ip_filter": "role:compute-user",
   "network:get_floating_ip": "role:compute-user",
   "network:get_floating_ip_pools": "role:compute-user",
   "network:get_floating_ip_by_address": "role:compute-user",
   "network:get_floating_ips_by_project": "role:compute-user",
   "network:get_floating_ips_by_fixed_address": "role:compute-user",
   "network:allocate_floating_ip": "role:compute-user",
   "network:deallocate_floating_ip": "role:compute-user",
   "network:associate_floating_ip": "role:compute-user",
   "network:disassociate_floating_ip": "role:compute-user",
   "network:get_fixed_ip": "role:compute-user",
   "network:add_fixed_ip_to_instance": "role:compute-user",
   "network:remove_fixed_ip_from_instance": "role:compute-user",
   "network:add_network_to_project": "role:compute-user",
   "network:get_instance_nw_info": "role:compute-user",
   "network:get_dns_domains": "role:compute-user",
   "network:add_dns_entry": "role:compute-user",
   "network:modify_dns_entry": "role:compute-user",
   "network:delete_dns_entry": "role:compute-user",
   "network:get_dns_entries_by_address": "role:compute-user",
   "network:get_dns_entries_by_name": "role:compute-user",
   "network:create_private_dns_domain": "role:compute-user",
   "network:create_public_dns_domain": "role:compute-user",
   "network:delete_dns_domain": "role:compute-user"
}
```

## User

- Users represent an individual API consumer. A user itself must be owned by a specific domain, and hence all user names are not globally unique, but only unique to their domain.
- Groups are a container representing a collection of users. A group itself must be owned by a specific domain, and hence all group names are not globally unique, but only unique to their domain.
- Due to their container architecture, domains may be used as a way to delegate management of OpenStack resources. A user in a domain may still access resources in another domain, if an appropriate assignment is granted.

## Problem

- We can set quota for a specific project, but how can we set quota for a specific domain, or a domain has unlimited quota ?
    - Answer: Domain can set quota.[7]
- A user belong only in one domain or a user can belong multi-domain?
    - Answer: A user only can exist in a maximum one domain (identified by ID, not by name). But a user can access to resource of other domain if it is granted permission.
- Use of roles in grant resource access permisisons ?
- Does every project (nova,cinder,swift,neutron...) has it own quota component?

## References

- [1] [https://www.mirantis.com/blog/mirantis-training-blog-openstack-keystone-domains/](https://www.mirantis.com/blog/mirantis-training-blog-openstack-keystone-domains/)
- [2] policy.v3cloudsample.json
- [3] [https://wiki.openstack.org/wiki/DomainQuotaManagementAndEnforcement](https://wiki.openstack.org/wiki/DomainQuotaManagementAndEnforcement)
- [4] [https://wiki.openstack.org/wiki/Domains](https://wiki.openstack.org/wiki/Domains)
- [5] [https://docs.openstack.org/keystone/pike/admin/identity-concepts.html](https://docs.openstack.org/keystone/pike/admin/identity-concepts.html)
- [6] [https://docs.openstack.org/keystone/latest/getting-started/architecture.html](https://docs.openstack.org/keystone/latest/getting-started/architecture.html)
- [7] [https://wiki.openstack.org/wiki/DomainQuotaDriver'](https://wiki.openstack.org/wiki/DomainQuotaDriver)