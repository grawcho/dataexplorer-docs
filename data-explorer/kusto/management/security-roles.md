---
title: Security roles management - Azure Data Explorer
description: This article describes security roles management in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 04/25/2021
---
# Security roles management

> [!IMPORTANT]
> Before altering authorization rules on your Kusto cluster(s), read the following:
> [Kusto access control overview](../management/access-control/index.md) 
> [role based authorization](../management/access-control/role-based-authorization.md) 

This article describes the control commands used to manage security roles.
Security roles define which security principals (users and applications) have
permissions to operate on a secured resource such as a database or a table,
and what operations are permitted. For example, principals that have the
`database viewer` security role for a specific database can query and view all
entities of that database (with the exception of restricted tables).

The security role can be associated with security principals or security groups (which
can have other security principals or other security groups). When a security
principal attempts to make an operation on a secured resource, the system checks
that the principal is associated with at least one security role that grants
permissions to perform this operation on the resource. This is called an
**authorization check**. Failing the authorization check aborts the operation.

## Security roles management commands

### Syntax

*Verb* *SecurableObjectType* *SecurableObjectName* *Role* [`(` *ListOfPrincipals* `)` [*Description*]]

### Arguments

* *Verb* indicates the kind of action to perform: `.show`, `.add`, `.drop`, and `.set`.

    |*Verb* |Description                                  |
    |-------|---------------------------------------------|
    |`.add` |Adds one or more principals to the role.     |
    |`.drop`|Removes one or more principals from the role.|
    |`.set` |Sets the role to the specific list of principals, removing all previous ones (if any).|

* *SecurableObjectType* is the kind of object whose role is specified.

    |*SecurableObjectType*|Description|
    |---------------------|-----------|
    |`database`|The specified database|
    |`table`|The specified table|
    |`materialized-view`| The specified [materialized view](materialized-views/materialized-view-overview.md)| 

* *SecurableObjectName* is the name of the object.

* *Role* is the name of the relevant role.

    |*Role*      |Description|
    |------------|-----------|
    |`admins`    |Have control over the securable object, including the ability to view, modify it, and remove the object and all sub-objects.|
    |`users`     |Can view the securable object, and create new objects underneath it.|
    |`viewers`   |Can view the securable object.|
    |`unrestrictedviewers`|At the database level only, allows viewing of restricted tables (which are not exposed to "normal" `viewers` and `users`).|
    |`ingestors` |At the database level only, allow data ingestion into all tables.|
    |`monitors`  ||

* *ListOfPrincipals* is an optional, comma-delimited list of security principal
  identifiers (values of type `string`).

* *Description* is an optional value of type `string` that is stored alongside
  the association, for future audit purposes.

### .show command

The `.show` command lists the principals that are set on the securable object. A line is returned for each role assigned to the principal. 

#### Syntax

`.show` *SecurableObjectType* *SecurableObjectName* `principals`

#### Example

The following control command lists all security principals which have some
access to the table `StormEvents` in the database:

```kusto
.show table StormEvents principals
```

Here are potential results from this command:

|Role |PrincipalType |PrincipalDisplayName |PrincipalObjectId |PrincipalFQN 
|---|---|---|---|---
|Database Apsty Admin |Azure AD User |Mark Smith |cd709aed-a26c-e3953dec735e |aaduser=msmith@fabrikam.com|

## Managing database security roles

`.set` `database` *DatabaseName* *Role* `none` [`skip-results`]

`.set` `database` *DatabaseName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

`.add` `database` *DatabaseName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

`.drop` `database` *DatabaseName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

The first command removes all principals from the role. The second removes all
principals from the role, and sets a new set of principals. The third adds new
principals to the role without removing existing principals. The last removes
the indicated principals from the roles and keeps the others.

Where:

* *DatabaseName* is the name of the database whose security role is being modified.

* *Role* is: `admins`, `ingestors`, `monitors`, `unrestrictedviewers`, `users`, or `viewers`.

* *Principal* is one or more principals. See [principals and identity providers](./access-control/principals-and-identity-providers.md) for how to specify these principals.

* `skip-results`, if provided, requests that the command will not return the updated
  list of database principals.

* *Description*, if provided, is text that will be associated with the change
  and retrieved by the corresponding `.show` command.

## Managing table security roles

`.set` `table` *TableName* *Role* `none` [`skip-results`]

`.set` `table` *TableName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

`.add` `table` *TableName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

`.drop` `table` *TableName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

The first command removes all principals from the role. The second removes all
principals from the role, and sets a new set of principals. The third adds new
principals to the role without removing existing principals. The last removes
the indicated principals from the roles and keeps the others.

Where:

* *TableName* is the name of the table whose security role is being modified.

* *Role* is: `admins` or `ingestors`.

* *Principal* is one or more principals. See [principals and identity providers](./access-control/principals-and-identity-providers.md) for how to specify these principals.

* `skip-results`, if provided, requests that the command will not return the updated
  list of table principals.

* *Description*, if provided, is text that will be associated with the change
  and retrieved by the corresponding `.show` command.

### Example

```kusto
.add table Test admins ('aaduser=imike@fabrikam.com ')
```

## Managing materialized view security roles

`.show` `materialized-view` *MaterializedViewName* `principals`

`.set` `materialized-view` *MaterializedViewName* `admins` `(` *Principal* `,[` *Principal...* `])`

`.add` `materialized-view` *MaterializedViewName* `admins` `(` *Principal* `,[` *Principal...* `])`

`.drop` `materialized-view` *MaterializedViewName* `admins` `(` *Principal* `,[` *Principal...* `])`

Where:

* *MaterializedViewName* is the name of the materialized view whose security role is being modified
* *Principal* is one or more principals. See [principals and identity providers](./access-control/principals-and-identity-providers.md)

## Managing function security roles

`.set` `function` *FunctionName* *Role* `none` [`skip-results`]

`.set` `function` *FunctionName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

`.add` `function` *FunctionName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

`.drop` `function` *FunctionName* *Role* `(` *Principal* [`,` *Principal*...] `)` [`skip-results`] [*Description*]

The first command removes all principals from the role. The second removes all
principals from the role, and sets a new set of principals. The third adds new
principals to the role without removing existing principals. The last removes
the indicated principals from the roles and keeps the others.

Where:

* *FunctionName* is the name of the function whose security role is being modified.

* *Role* is always `admin`.

* *Principal* is one or more principals. See [principals and identity providers](./access-control/principals-and-identity-providers.md)
  for how to specify these principals.

* `skip-results`, if provided, requests that the command will not return the updated
  list of function principals.

* *Description*, if provided, is text that will be associated with the change
  and retrieved by the corresponding `.show` command.

### Example

```kusto
.add function MyFunction admins ('aaduser=imike@fabrikam.com') 'This user should have access'
```
