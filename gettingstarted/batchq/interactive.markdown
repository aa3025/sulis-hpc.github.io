---
layout: page
title: Interactive jobs
parent: Job submission
grand_parent: Getting Started
nav_order: 6
---

# Interactive jobs
{: .no_toc }

1. TOC
{:toc}

The primary focus of the Sulis service is non-interactive processing of scripts submitted to the batch system from the login node. Despite this there are occasions when interactive access to a compute node can be beneficial for debugging, testing and compiling.  

SLURM can be used to allocate resources for interactive work, but note the following caveats.

- There is no guarantee that the resources will be available at a convenient time if the cluster is busy. 

- Interactive jobs consume CPU and GPU time budgets in the same way as batch jobs. Leaving an interactive session idle when not in use can consume a significant fraction of a user's allocation.

Should you have difficulty obtaining an interactive session at a useful time then is may be possible to make an advance reservation. Contact your local [support representative](../../support) to request this.

## Interactive sessions for multi-task applications on a single node

From the login node, issue the following `srun` command to request an interactive session on a compute node. This assumes that any task parallel (e.g. MPI) operations launched within the session will use a single CPU per task, so we request {{site.data.slurm.cnode_cores_per_node}} "slots" available for processes in the session. This is the appropriate configuration for debugging a pure MPI code.

```bash
{{site.data.terminal.prompt}} srun --interactive  -N 1 -n 128 --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}} --time=8:00:00 --pty bash
```

You will see the message `srun: job xxxxxx queued and waiting for resources` until a compute node becomes available.
Once allocated the user will be droped into an interactive session on that node, in this case node046. 

```bash
[user@node046(sulis) ~]$
```

The interactive session will end after the 8 hours requested, or if the user ends the session with:

```bash
[user@node046(sulis) ~]$ exit
```

To launch MPI applications within this session use `srun` as you would within a SLURM job script. This will use all 128 slots unless overridden with the `-n` argument, for example:

```bash
[user@node046(sulis) ~]$ srun -n 16 ./my_mpi_prog
```

## Interactive sessions for single task applications on a single node

From the login node

```bash
{{site.data.terminal.prompt}} srun --interactive  -N 1 -n 1 -c 128 --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}} --time=8:00:00 --pty bash
```

will (as above) display `srun: job xxxxxx queued and waiting for resources` until a compute node becomes available. Once given a terminal prompt on the allocated node, `srun` can be used to launch a single process which has access to all {{site.data.slurm.cnode_cores_per_node}}. This is the appropriate configuration for debugging a threaded code or Python script that uses multiprocessing. For example the following will launch a Python script which can use all 128 CPUs in the node.

```bash
[user@node046(sulis) ~]$ srun -n 1 -c 128 python my_multiproc_script.py
```

See also the application notes on [Jupyter](../../appnotes/jupyter).

## Interactive sessions on a GPU node

To request an interactive session on a GPU-equipped node, specify the `gpu` partition and include a resource request for the A100 GPUs in that node.

```bash
{{site.data.terminal.prompt}} srun -p gpu --interactive  -N 1 -n 1 -c 128 --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}} --gres=gpu:ampere_a100:3 --time=8:00:00 --pty bash
```

In this case SLURM will allocate one "slot" for a process launched via `srun`  which will have access to all 128 CPUs and the 3 GPUs in the node. Once the resources are allocated an available then a new command prompt will appear. 

```bash
[user@gpu016(sulis) ~]$
```

This environment will have access to all 3 GPUs in the node.
```bash
[user@gpu016(sulis) ~]$ nvidia-smi  -L              
GPU 0: A100-PCIE-40GB (UUID: GPU-04a0384d-82ee-c4c5-2414-4ad1ab396f49)
GPU 1: A100-PCIE-40GB (UUID: GPU-f55b44e4-79e9-0ee0-c1ad-a4dbd693ad71)
GPU 2: A100-PCIE-40GB (UUID: GPU-23d2b9d2-42d7-3886-a921-5c4869914b0e)
```

## Interactive use of multiple nodes

This is possible by using `salloc` to request resource across multiple nodes, and then use `srun` _on the login node_ to launch processes which use that allocation. This can be useful for debugging problems with code/scripts 
that only manifest when using multiple nodes.

For example, the following requests an allocation of two nodes, 

```bash
{{site.data.terminal.prompt}} salloc --ntasks-per-node=1 --cpus-per-task=128 --mem-per-cpu={{site.data.slurm.cnode_ram_per_core}} --nodes=2 --time=00:15:00 
```

Once the allocation has been granted, the terminal session will return to the login node prompt. Using `srun` _on the login node_ will (in this case) execute 1 instance of the command specified on each of the two nodes allocated. For example:

```bash
{{site.data.terminal.prompt}} srun uname -n 
node139.sulis.hpc
node138.sulis.hpc
```

Use `exit` to relinquish the allocation when no longer needed.

```bash
{{site.data.terminal.prompt}} exit
exit
salloc: Relinquishing job allocation 223826
```
