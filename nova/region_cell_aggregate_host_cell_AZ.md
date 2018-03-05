# Region, Cell, Aggregate Host, Availability Zone, Cell in Aggregate Host, Cell v2.0 Disavantage

## Region

## Host aggregates

Host aggregates differ with Availability Zone in purpose and how the system and users use them:

- Host aggregates use for filtering scheduler (A flavor may contains metadata filters for some requirements about host aggregates)
- Availability Zone visible with User.

Host aggregates are a mechanism for partitioning hosts in an OpenStack cloud, or a region of an OpenStack cloud, based on arbitrary characteristics. Examples where an administrator may want to do this include where a group of hosts have additional hardware or performance characteristics.[2]

Host aggregates are not explicitly exposed to users. Instead administrators map flavors to host aggregates. Administrators do this by setting metadata on a host aggregate, and matching flavor extra specifications. The scheduler then endeavors to match user requests for instance of the given flavor to a host aggregate with the same key-value pair in its metadata. Compute nodes can be in more than one host aggregate. [2]

Administrators are able to **optionally** expose a host aggregate as an availability zone. Availability zones are different from host aggregates in that they are explicitly exposed to the user, and hosts can only be in a single availability zone. Administrators can configure a default availability zone where instances will be scheduled when the user fails to specify one.

## Availability Zone

Availability Zones are the end-user visible logical abstraction for partitioning a cloud without knowing the physical infrastructure. That abstraction doesn’t come up in Nova with an actual database model since the availability zone is actually a specific metadata information attached to an aggregate. Adding that specific metadata to an aggregate makes the aggregate visible from an end-user perspective and consequently allows to schedule upon a specific set of hosts (the ones belonging to the aggregate).

That said, there are a few rules to know that diverge from an API perspective between aggregates and availability zones:

    * one host can be in multiple aggregates, but it can only be in one availability zone
    * by default a host is part of a default availability zone even if it doesn’t belong to an aggregate (the configuration option is named default_availability_zone)

Administrators are able to optionally expose a host aggregate as an availability zone. Availability zones are different from host aggregates in that they are explicitly exposed to the user, and hosts can only be in a single availability zone. Administrators can configure a default availability zone where instances will be scheduled when the user fails to specify one.[2]

```txt
nova aggregate-create <name> [<availability-zone>]
    Create a new aggregate named <name>, and optionally in availability zone [<availability-zone>] if specified. The command returns the ID of the newly created aggregate. Hosts can be made available to multiple host aggregates. Be careful when adding a host to an additional host aggregate when the host is also in an availability zone. Pay attention when using the nova aggregate-set-metadata and nova aggregate-update commands to avoid user confusion when they boot instances in different availability zones. An error occurs if you cannot add a particular host to an aggregate zone for which it is not intended.[2]
```

## Cell


## Problem

- Host Aggregate come up with flavor selection in host scheduling.
- Can we boot an instance without specify its Availability Zone ?
- Can a flavor can has more than one metadata (mapping with Host Aggregate)? (yes)
- Can we use **Availability Zone** and flavor with **Host Aggregate metadata** together to select host which boot our new VM ?
- Can a Host Aggregate has more than one metadata ?

## Theory - Work

- An instance can create successfully without specify Availability Zone.
- ~~Must be test to make sure that a flavor has only maximum one metadata or~~ **a flavor can has more than one metadata**.
- Availability Zone and **multi Host Aggregate** can be use together to filter host when create a new VM
- Test relations and influences between Host Aggregate and Availability Zone

## References

- [1]<https://www.openstack.org/assets/presentation-media/divideandconquer-2.pdf>
- [2]<https://docs.openstack.org/nova/latest/admin/configuration/schedulers.html#host-aggregates>
- [3]<https://docs.openstack.org/nova/latest/admin/configuration/schedulers.html#example-specify-compute-hosts-with-ssds>
- [4]<https://docs.openstack.org/nova/latest/user/aggregates.html>
- [5]<https://docs.openstack.org/nova/latest/user/aggregates.html#availability-zones-azs>