# Kolla Deployment Document

## Case 1: All-in-one

Environment: 2 VM, 3 network:

- ssh management net: 192.168.122.0/24
- internal net: 192.168.120.0/24
- external net: 192.168.100.0/24

deployment VM - deployment host VM: 192.168.122.10/24 3 NICs

ops VM - VM where all-in-one OpenStack deployment is installed: 192.168.122.100/24 - 3 NICs