# ephemeral_disk in Nomad

Nomad offers something called [`ephemeral_disk`](https://www.nomadproject.io/docs/job-specification/ephemeral_disk) to
persist data. If it is marked as "sticky", Nomad will migrate the data (if needed) before scheduling the data on 
another node (during redeployments/upgrades).

While I don't think persisting large data to ephemeral disk is a good idea (migration may take a long time), it might
be suitable for persisting smaller files (like sqlite databases or text files).

## Where is data stored?
Nomad creates an `alloc` directory for each allocation on whichever node the allocation is scheduled. Since Nomad 
creates one allocation for all the tasks inside a `group`, this directory is shared by all the tasks in the group. 
This directory contains 3 sub directories - `data`, `logs` and `tmp`.

`alloc/data` is used as ephemeral disk for storing data.

## References
1. https://www.nomadproject.io/docs/job-specification/ephemeral_disk
2. https://www.nomadproject.io/docs/internals/filesystem

tags: #nomad
