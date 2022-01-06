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
$> tar xzvf sstcore-X.Y.Z.tar.gz
$> cd sstcore-X.Y.Z
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
$> tar xzvf sstelements-X.Y.Z.tar.gz
$> cd sstelements-X.Y.Z
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
appropriately utilized.  However, all of the major SST command line tools 
provide a `--help` option to print useful command line options 
and their associated parameters.  

### SST Execution and Utility Functions

* _Executing a Simulation_: The most common command line tool is the `sst` command.  
This is utilized to build and execute a simulation run using the SST Core and associated 
Elements libraries.  Using the `sst` command line tool and passing a loadable 
simulation graph using Python or JSON can be performed as follows:
```
$> sst /path/to/simulation.py
$> sst /path/to/simulation.json
```

* _Executing with Verbosity_: Similar to the command above, users may enable 
verbose output of the graph construction and simulation by passing the `-v`
or `--verbose=LEVEL` option.  The higher the `LEVEL` specified, the more verbose
the output will be.  This is often useful for debugging new simulation configurations.  
```
$> sst -v /path/to/simulation.py
$> sst --verbose=4 /path/to/simulation.py
```

* _Testing Simulation Graph Syntax_: One of the common issues that users encounter 
when constructing complex simulation graphs is basic syntactic issues in their 
respective input files that prevent SST from constructing a valid simulation.  It is 
often useful to utilize the SST graph loader(s) to validate a graph using the 
internal initialization functions without actually executing the simulation.  
This can be performed using the following command.  Note that SST will report 
that the simulation completed, but the reported simulation time will be 0s.
```
$> sst --run-mode=init /path/to/simulation.py
Simulation is complete, simulated time: 0 s
```

* _Outputting DOT Configuration Graphs_: It is also often useful for debugging 
or publication purposes to direct SST to output a GraphViz DOT file that contains 
the configuration graph of the candidate simulation.  This can be performed 
with or without actually executing the simulation.  We show examples for both 
as follows:
```
$> sst --output-dot=foo.dot /path/to/simulation.py
$> sst --run-mode=init --output-dot=foo.dot /path/to/simulation.dot
$> dot -Tpdf foo.dot > foo.pdf
```

* _Converting Between Simulation File Types_: One of the latest features 
provided by SST is the ability to develop simulation inputs using Python 
or JSON syntax.  Python is often useful when developing large simulations 
that require redundant components generated (multiple CPUs with 
memory hierarchys).  However, JSON inputs are often 
easier to discern the respective graph 
connectivity given the inherent syntax of JSON.  SST provides an internal 
feature that permits users to specify in a graph in one format and convert 
it to another format (with or without executing the simulation).  Further, 
users may also specify large, complex graph inputs and direct SST to 
output the same graph in a more expanded form.  This can also be useful 
for debugging complex simulations as the graph output will be the raw 
form of what SST utilized for the actual graph connectivity.  Various 
forms of inputting and outputting graphs are shown below (all without 
actually executing the simulation).
```
$> sst --run-mode=init --output-json=foo.json /path/to/simulation.py
$> sst --run-mode=init --output-config=foo.py /path/to/simulation.json
$> sst --run-mode=init --output-config=foo.py /path/to/complex/simulation.py
```

### SST Information
The SST infrastructure may potentially contain a large number of 
available components for a target installation.  Each of the available 
components may have a large number of available configuration options 
and/or contain nested sets of subcomponent configuration options.  In order 
to assist users to discern the required or optional configuration parameters 
for components or subcomponents, SST provides the `sst-info` command.  This 
command outputs the relevant component and subcomponent parameters, options 
and statistics.  There are a number of potential methods for querying 
the configured components.  We details these as follows:

* _Querying All Components_: The first method is to simply query all known 
configured components.  By simply executing the `sst-info` command, 
the SST-Core will output all the configuration parameters, statitics and nested 
subcomponents for all the known components.  This includes any component built 
within the installed SST package as well as any user-registered components.
```
$> sst-info
```

* _Querying Specific Components_: If you seek to query the information for 
specific components, `sst-info` provides a command line option to specify 
the name of the component or subcomponent to narrow the search results.  This 
can be done be specifying the component and/or the component/subcomponent 
combination.  Examples of doing this are as follows using the `miranda` 
component.
```
$> sst-info miranda
$> sst-info miranda.CopyGenerator
```

An example of the output from the `miranda.CopyGenerator` command from above 
is as follows:
```
PROCESSED 1 .so (SST ELEMENT) FILES FOUND IN DIRECTORY(s) /path/to/sst-elements-library
Filtering output on Element = "miranda.CopyGenerator"
================================================================================
ELEMENT 0 = miranda ()
Num Components = 1
Num SubComponents = 11
      SubComponent 0: CopyGenerator
      Interface: SST::Miranda::RequestGenerator
         NUM STATISTICS = 0
         NUM PORTS = 0
         NUM SUBCOMPONENT SLOTS = 0
         NUM PARAMETERS = 5
            PARAMETER 0 = read_start_address (Sets the start read address for this generator) [0]
            PARAMETER 1 = write_start_address (Sets the start target address for writes for the generator) [1024]
            PARAMETER 2 = request_size (Sets the size of each request in bytes) [8]
            PARAMETER 3 = request_count (Sets the number of items to be copied) [128]
            PARAMETER 4 = verbose (Sets the verbosity of the output) [0]
    CopyGenerator: Creates a single copy of stream of reads/writes replicating an array copy pattern
    Using ELI version 0.9.0
    Compiled on: Dec 16 2021 11:12:05, using file: generators/copygen.h
Num Modules = 0
Num SSTPartitioners = 0
```

### SST Configuration
Finally, the `sst-config` tool can be utilized by system administrators and 
component developers to resolve the respective SST configuration and compilation 
environment.  The `sst-config` tool has the ability to report information 
regarding which components were built with the target installation, which 
compilers and libraries were utilized as well as the respective compilation 
options required to successfully build and link out-of-tree components against 
the target installation.  We describe several of the most common options as follows:

* _Print the Entire Configuration_: Utilizing the raw `sst-config` command with 
no options will print all data stored with the respective installation.  This includes 
locations of libraries, compiler utilized to build SST and any user-installed 
(out-of-tree) components.  This can be done as follows:
```
$> sst-config
```

* _Print the C++ Compiler_:  For those seeking to develop external components, 
it is often useful to derive the target compiler and arguments utilized to build 
the SST-Core.  This ensures that the internal SST loading functions have the 
ability to load and execute external components packaged as shared libraries.  This 
can be done using the `sst-config` tool as follows:
```
$> sst-config --CXX
$> sst-config --ELEMENT_CXXFLAGS
$> sst-config --ELEMENT_LDFLAGS
```

A useful example of integrating these commands into an external Makefile resembles 
the following:
```
ifeq (, $(shell which sst-config))
 $(error "No sst-config in $(PATH), add `sst-config` to your PATH")
endif

CXX=$(shell sst-config --CXX)
CXXFLAGS=$(shell sst-config --ELEMENT_CXXFLAGS)
LDFLAGS=$(shell sst-config --ELEMENT_LDFLAGS)
CPPFLAGS=-I./
OPTIMIZE_FLAGS=-O3

FOO_SOURCES := $(wildcard *.cc)

FOO_HEADERS := $(wildcard *.h)

FOO_OBJS := $(patsubst %.cc,%.o,$(wildcard *.cc))

all: libfoo.so

debug: CXXFLAGS += -DDEBUG -g
debug: libfoo.so

libfoo.so: $(FOO_OBJS)
        $(CXX) $(OPTIMIZE_FLAGS) $(CXXFLAGS) $(CPPFLAGS) $(LDFLAGS) -o $@ *.o
%.o:%.cc $(FOO_HEADERS)
        $(CXX) $(OPTIMIZE_FLAGS) $(CXXFLAGS) $(CPPFLAGS) -c $<
install: libfoo.so
        sst-register foo foo_LIBDIR=$(CURDIR)
clean:
        rm -Rf *.o libfoo.so
```

## Basic Runtime Functions <a name="BasicRuntimeFunctions"></a>

## Collecting Data From SST <a name="CollectingData"></a>

## Links to Documentation <a name="LinksToDocumentation"></a>
* [SST Release Page](http://sst-simulator.org/SSTPages/SSTMainDownloads/)
* [SST-Core Github](https://github.com/sstsimulator/sst-core)
* [SST-Elements Github](https://github.com/sstsimulator/sst-elements)
* [GraphViz](https://graphviz.org/)
