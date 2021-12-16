# Level 1 Tutorial

## Table of Contents
1. [What is SST](#WhatIsSST)
2. [Components of SST](#ComponentsOfSST)
3. [Basic Installation](#BasicInstallation)
4. [Basic Commands](#BasicCommands)
5. [Basic Runtime Functions](#BasicRuntimeFunctions)
6. [Collecting Data from SST](#CollectingData)
7. [Links to Documentation](#LinksToDocumentation)

## What is SST: <a name="WhatIsSST"></a>
  To combat the ever-changing and multifaceted technologies of the HPC 
  community, an effort was made to create a unified tool box used to evaluate 
  these challenges. The Structural Simulation Toolkit (SST) an open, modular, 
  parallel, multi-criteria, multi-scale simulation framework  was created to 
  design and procure HPC systems. By providing a number of interfaces and 
  utilizes for simulation models, SST, will demolish the need for simulations 
  for individual system components. A now unified framework exists for parallel 
  simulation of large machines at multiple levels. The SST has been used in 
  variety of network, memory and applications.

## Components of SST: <a name="ComponentsOfSST"></a>
  To meet these requirements, the SST is comprised of a simple simulation core 
  that contains a parallel discrete event simulator (DES) and support services 
  for simulation. components, representing hardware systems such as processors, 
  network switches, or memory devices, interface with the simulation core to 
  communicate and operate with a common notion of timeframe. The simulation 
  core also provides support services such as power and area estimation, 
  checkpointing, configuration/initialization of the simulation, and statistics 
  gathering. The SST’s modular interface eases the integration of existing 
  simulators into a common framework and is licensed under a BSD-like license. 
  In addition, the components of the SST model are compatible with external 
  models such as Gem5 and DRAMSim2. Because of the open source core, the 
  modular framework is easily extensible with new models. 

  The SST uses a (DES) model layered on top of MPI.  The simulations are 
  comprised of components or building blocks of the simulation model are 
  connected by links. These components interact by sending events over 
  links—with each link having a minimum latency. In addition, components can 
  load subcomponents and modules for even more functionality and customization. 
  To achieve better performance, the SST uses a conservative (i.e. 
  no roll-back) distance-based optimization. At the start of the simulation, 
  the system topology is represented by a graph with components as nodes and 
  connections between them as edges, with each edge labeled with the minimum 
  latency between the connected components. The Zoltan library is then used to 
  partition components across the MPI ranks with the goal of balancing the load 
  and partitioning across the highest latency links. Tests indicate that the 
  algorithm is scalable and shows less than 25% overhead at 128 ranks (11,904 
  simulated components) compared to a single rank for detailed simulations. 

![image](https://user-images.githubusercontent.com/74792926/121704586-e5454680-caa1-11eb-8c1f-b33daad81498.png)

## Basic Installation <a name="BasicInstallation"></a>

### Software Prerequisites

SST can be built on a variety of Linux and OSX platforms.  Currently, Windows 
is not a supported platform.  For each of these systems, there are a number 
of prerequisite software packages that need to be installed prior to building 
SST from source.  This includes the following:

* C/C++ Compiler (GCC,Clang)
* GNU Autotools
* OpenMPI (4.0.5 is preferred)
* Python 3.X+
* Git (if cloning the source tree)

### SST-Core Installation

Once the required software packages have been installed, you can either download 
the SST-Core package from the SST [release page](http://sst-simulator.org/SSTPages/SSTMainDownloads/) 
or check out the source code from [Github](https://github.com/sstsimulator/sst-core).

If you download the release package, untar it as follows:
```
$> tar xzvf sstcore-XX.Y.Z.tar.gz
$> cd sstcore-XX.Y.Z
```

If you seek to clone the source repository directly, do the following.  Note 
the final autogen command to generate the required configuration scripts.  This 
is only required if you clone the source repository directly.
```
$> git clone https://github.com/sstsimulator/sst-core.git sst-core
$> cd sst-core
$> ./autogen.sh
```

Now that you have the SST-Core source, you should select an appropriate 
installation location.  For this example, we will utilize `/opt/sst` as our 
installation target.  At this point, we can configure, build and install 
the SST-Core infrastructure as follows:
```
$> mkdir /opt/sst
$> ./configure --prefix=/opt/sst
$> make -j
$> make install
$> export PATH=$PATH:/opt/sst/bin
$> sst --version
```

In addition to the aforementioned basic build instructions, there are a number 
of additional configuration options that can be specified.  A short summary 
of these additional options are listed as follows:

| **Option**  | **Description**  |
|:-|:-|
| `--with-hdf5=/path/to/HDF5` | Enables support for HDF5 statistics output |
| `--disable-mpi`             | Disables MPI (only support for single node |
| `--disable-mem-pools`       | Disables memory pools |
| `--enable-debug`            | Enables debug mode  |
| `--enable-event-tracking`   | Enables event tracking for debug mode  |
| `--enable-profile`          | Enables performance profiling of core features  |

For more information regarding these options and other options, do the following:
```
$> ./configure --help
```

### SST-Elements Installation

Now that the SST-Core has been installed, we can build the SST-Elements package.  
The SST-Elements package provides individual *components* that simulate specific 
pieces of hardware.  The SST-Elements package provides a number of pathological 
hardware elements such as memories, caches, processor cores and network interfaces.  
It also provides a number of examples on how to construct larger simulations of 
full systems.  The SST-Elements provides a number of components that are built 
by default as well as a number of components that require external libraries.  
For the purposes of this tutorial, we will build the SST-Elements package 
with the default options.  

Much in the same manner as the SST-Core package, we can select the latest release 
package of SST-Elements or build directly from the git source tree.  
The standard SST-Elements release packages are available on the 
[release page](http://sst-simulator.org/SSTPages/SSTMainDownloads/)
or check out the source code from [Github](https://github.com/sstsimulator/sst-elements).

If you download the release package, untar it as follows:
```
$> tar xzvf sstelements-XX.Y.Z.tar.gz
$> cd sstelements-XX.Y.Z
```

If you seek to clone the source repository directly, do the following.  Note 
the final autogen command to generate the required configuration scripts.  This 
is only required if you clone the source repository directly.
```
$> git clone https://github.com/sstsimulator/sst-elements.git sst-elements
$> cd sst-elements
$> ./autogen.sh
```

Now that you have the SST-Elements source, you should select an appropriate 
installation location.  For this example, we will utilize `/opt/sst/elements` as our 
installation target.  At this point, we can configure, build and install 
the SST-Elements infrastructure as follows:
```
$> mkdir /opt/sst/elements
$> ./configure --prefix=/opt/sst/elements --with-sst-core=/opt/sst
$> make -j
$> make install
$> sst-info -q
```

At this point in the tutorial, we have a functional SST environment 
with both the base SST-Core and the SST-Elements packages installed.

## Basic Commands <a name="BasicCommands"></a>

There are a number of basic commands that are commonly utilized when 
running and/or debugging an SST simulation.  We summarize the most 
commonly used command sets here and the situations where they are 
appropriately utilized.

```
sst /path/to/simulation.py
```

```
sst-info
```

```
sst-config
```

## Basic Runtime Functions <a name="BasicRuntimeFunctions"></a>

## Collecting Data From SST <a name="CollectingData"></a>

## Links to Documentation <a name="LinksToDocumentation"></a>

