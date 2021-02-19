---
title: Role Assignment
---
<ApiLifecycle access="beta" />

# Role Assignment
Role assignment to principals makes them administrators of your org. Principals can be users or groups of users.

## Standard Role Assignment
The following role types are provided and supported out of the box:
| Role type                               | Label                               | Optional targets                      |
| :-------------------------------------- | :---------------------------------- | :------------------------------------ |
| `API_ACCESS_MANAGEMENT_ADMIN`           | API Access Management Administrator |                                       |
| `APP_ADMIN`                             | Application Administrator           | Apps                                  |
| `GROUP_MEMBERSHIP_ADMIN`                | Group Membership Administrator      | [Groups](/docs/reference/api/groups/) |
| `HELP_DESK_ADMIN`                       | Help Desk Administrator             | [Groups](/docs/reference/api/groups/) |
| `MOBILE_ADMIN`                          | Mobile Administrator                |                                       |
| `ORG_ADMIN`                             | Organizational Administrator        |                                       |
| `READ_ONLY_ADMIN`                       | Read-Only Administrator             |                                       |
| `REPORT_ADMIN`                          | Report Administrator                |                                       |
| `SUPER_ADMIN`                           | Super Administrator                 |                                       |
| `USER_ADMIN`                            | Group Administrator                 | [Groups](/docs/reference/api/groups/) |

Standard role assignment is done in two steps:
1. Assign a role to a user or group. At this point the admin has the supported privileges of the role over all resources across organization.
2. (Optional) If the role supports targets, use one of the [target operations](/docs/reference/api/roles/#role-target-operations) to specify which specific resource the admin can manage.

Note that the entities involved in standard role assignment are:
* a role; identified either by type or id returned from [listing API](/docs/reference/api/roles/#list-roles)
* a principal; either a group or a user
* (Optional) a resource; when using target [target operations](/docs/reference/api/roles/#role-target-operations) this could be either an app or a group

## Custom Role Assignment
Custom roles can be built by piecing [permissions](/docs/reference/api/roles/#permission-types) together. Once a custom role is built, you can use its id or name to assign to admins. The process is:
1. Build a custom role
2. Build a set of resources
3. Bind the admin with the role from step 1 targeting the resource set from step 2

An assignment of a role to an admin is called a [binding](/docs/reference/api/roles/#binding-object). Conceptually, a binding is identified by a unique id, and represents a single unique combination of principal, resource-set, and custom role. A given resource-set can have multiple bindings (do we specify the limit here perhaps?), allowing for different combinations of principals and roles as necessary to provide permission to the encompassing resource.

Therefore, when dealing with custom roles, three entities are always in play:
* a role; identified by its name or id
* a principal; either a group or a user
* a resource-set; identified by its id

### Resource Sets
A resource set is simply a collection of resources. The following resources are currently supported:
* All users
* All groups
* A specific group
* All users within a specific group

### Identifiers
#### Resource Identifiers
In order to specify a resource targeted by a resource set, you will simply use the REST URL of the corresponding Okta API:
* [All users](/docs/reference/api/users/#list-users)
  ``` http
  https://${yourOktaDomain}/api/v1/users
  ```
* [All groups](/docs/reference/api/groups/#list-groups)
  ``` http
  https://${yourOktaDomain}/api/v1/groups
  ```
* [A specific group](/docs/reference/api/groups/#get-group)
  ``` http
  https://${yourOktaDomain}/api/v1/groups/${targetGroupId}
  ```
* [All users within a specific group](/docs/reference/api/groups/#list-group-members)
  ``` http
  https://${yourOktaDomain}/api/v1/groups/${targetGroupId}/users
  ```

> **Tip:** If you use a role with permissions that don't apply to the resources in the resource set, the admin role will have no effect. For example, the `okta.users.profile.manage` permission will give the admin no privileges if granted to a resource set that only included `https://${yourOktaDomain}/api/v1/groups/${targetGroupId}` resources. If you want the admin to be able to manage the users within the group the resource set must include the corresponding `https://${yourOktaDomain}/api/v1/groups/${targetGroupId}/users` resource.

#### Binding Member Identifiers
In order to specify binding members, you will simply use the REST URL of the corresponding Okta API:
* [A specific user](/docs/reference/api/users/#get-user)
  ``` http
  https://${yourOktaDomain}/api/v1/users/${memberUserId}
  ```
* [A specific group](/docs/reference/api/groups/#get-group)
  ``` http
  https://${yourOktaDomain}/api/v1/groups/${memberGroupId}
  ```

## Custom vs. Standard
1. An admin can have both standard role assignments and custom role bindings. Note that privileges granted to an admin are an aggregate of:
    * standard roles directly assigned
    * standard roles granted through group membership
    * custom roles directly assigned
    * custom roles granted through group membership

    As a result, if an admin was granted a standard role limited to a single group and at the same time received group management privileges on all groups in the org through a custom role, the ultimate outcome is group management on all groups.
2. During BETA, Custom Roles, Permissions and Resources are limited to User and Group related items. We'll be introducing additional Resources and Permissions over time and appreciate your feedback.
3. A custom role cannot be assigned without a resource set, hence always being applicable only to a subset of resources. Standard roles on the other hand are always initially granted at the entire org. They are only scoped to specific resources byt subsequent invoking of the [target operations](/docs/reference/api/roles/#role-target-operations).

### Permission types
| Permission type                     | Description                                                                                                                         | Applicable resource types                    |
| :---------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- | :------------------------------------------- |
| `okta.users.manage`                 | Allows the admin to create and manage users and read all profile and credential information for users                               | All users, all users within a specific group |
| `okta.users.create`                 | Allows the admin to create users. If admin also scoped to manage a group, can add the user to the group on creation and then manage | All groups, a specific group                 |
| `okta.users.read`                   | Allows the admin to read any user's profile and credential information                                                              | All users, all users within a specific group |
| `okta.users.credentials.manage`     | Allows the admin to manage only credential lifecycle operations for a user                                                          | All users, all users within a specific group |
| `okta.users.userprofile.manage`     | Allows the admin to only do operations on the User Object, including hidden and sensitive attributes                                | All users, all users within a specific group |
| `okta.users.lifecycle.manage`       | Allows the admin to only take any user lifecycle operations                                                                         | All users, all users within a specific group |
| `okta.users.groupMembership.manage` | Allows the admin to manage a user's group membership (also need `okta.groups.members.manage` to assign to a specific group)         | All users, all users within a specific group |
| `okta.groups.manage`                | Allows the admin to fully manage groups in your Okta organization                                                                   | All groups, a specific group                 |
| `okta.groups.create`                | Allows the admin to create groups                                                                                                   | All groups                                   |
| `okta.groups.members.manage`        | Allows the admin to only take member operations in a group in your Okta org                                                         | All groups, a specific group                 |
| `okta.groups.read`                  | Allows the admin to only read information about groups and their members in your Okta organization                                  | All groups, a specific group                 |