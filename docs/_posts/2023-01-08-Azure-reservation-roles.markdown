---
layout: post
title:  "Azure reservation roles - Adding Reservation Administrator, reader, and purchaser"
date:   2023-01-08 08:47:29 +0000
---
Starting Feb 20, 2023 -  Azure EA enrollments will only be able to be managed within the Azure Portal

Feature announced for [General Availability] in August 2, 2022

Having had a bit of a play, here are my notes to summarise - as always please reach out to make any corrections

## New roles for Reservations

There are three granular roles to help manage access to Azure reservations:

- Reservation Administrator - Directory role assigned at Reservation -> Role Assignment
- Reservation Reader - Directory role assigned at Reservation -> Role Assignment
- Reservation Purchaser - Resource RBAC Permission assigned at resource group, subscription, or management group levels

The role of Reader and Administrator are notable as they allow a user to gain visibility and control over all existing reservations tied to the tenant, instead of needing to grant access to each order retroactively.

## Understanding where reservations sit within Azure

Reservations are a bit of a strange resource type, they don't follow the usual resource flow of sitting under a subscription.

Instead, the reservation service sits under the tenant as `providers/Microsoft.capacity` and do not inherit permissions from a subscription, as they don't sit under a subscription, RBAC roles of `Owner` or `Contributor` will not allow you to manage all reservations - though `Owners` for a subscription can manage reservations for that particular subscription for which they are an `Owner` - the above roles will unlock the ability to see across all reservations within the tenant once assigned.

## Permissions required

As the reservation resource is under the tenant instead of any subscriptions, you will need to be a global administrator and elevate user access administrator privileges to assign these roles, a global administrator can elevate the User Access Administrator through

`Azure Active Directory -> properties` and select "Yes" for `"Access management for Azure resources"`

As `Reservation Purchaser` is a resource level role, it can be assigned by an `Owner` of the resource group, subscription, or management group

## Assigning the roles

In what is a bit of a confusing twist, reservation roles are actually directory roles, but are instead assigned under
`Reservations -> role assignment`

instead of
`Azure Active Directory -> roles and administrators`

Hoping this will eventually be unified.

It looks like resource assignments to mangement groups, subscriptions, and resource groups can assign the Reservation Purchaser role, it looks like the purchaser role is resource based and can be assigned by an Owner (so as to puchase scoped reservations), while the Administrator and Reader roles are directory roles only assignable currently under the Reservation tab by a global administrator with User Access Administrator elevation activated.

## How about Privileged Identity Management (PIM)?

As the resource is tied to the tenant, but the assignment is a directory role, there is no apparent direct way to assign a PIM control for Reservation Administrators or Readers as the PIM roles only allow assignment of roles that are grantable within the Azure AD roles page, and does not cover the roles assignable within the Reservation resource.

Working around the problem, I was able to create a Privileged Access Group that is assigned a `Reservation Administrator` for our users to temporarily obtain membership to do reservation mangement

## Existing Reservation Orders

As usual, any enterprise administrator may assign permissions to individual reservation orders to users, and the user will have visibility on that individual item, but these new roles allow for control over a wider scope.

## Microsoft Documentation

[Permissions to view and manage Azure reservations]

[Buying Reservations]

[EA Administration guide]

[Permissions to view and manage Azure reservations]: https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/view-reservations
[Buying Reservations]: https://learn.microsoft.com/en-us/azure/cost-management-billing/reservations/prepare-buy-reservation?toc=%2Fazure%2Fcost-management-billing%2Freservations%2Ftoc.json
[General Availability]: https://azure.microsoft.com/en-us/updates/general-availability-reservation-administrator-and-reader-roles-in-azure-portal/
[EA Administration guide]: https://learn.microsoft.com/en-us/azure/cost-management-billing/manage/direct-ea-administration