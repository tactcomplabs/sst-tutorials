# Container Tutorial

## Table of Contents
1. [Singularity](#singularity)
    1. [Introduction](#introduction)
    2. [Singularity Installation](#singularityinstallation)
    3. [Building Basic Singularity Containers](#buildingbasicsingularitycontainers)
        1. [Pre-Built Formulas](#basicprebuildformulas)
        2. [Advanced: Container Construction](#basiccontainerconstruction)
            1. [RedHat Linux](#basicredhat)
            2. [Ubuntu/Debian Linux](#basicubuntu)
    4. [Building Parallel Singularity Containers](#buildingparallelsingularitycontainers)
        1. [Singularity Networking](#singularitynetworking)
        2. [Pre-Built Formulas](#parallelprebuildformulas)
        3. [Executing Containers with MPI](#mpi)
    5. [Singularity References](#singularityreferences)
2. [Docker](#docker)
3. [Spack](#spack)

## Singularity <a name="singularity"></a>

### Introduction <a name="introduction"></a>
The following section covers the basic installation and configuration 
steps required to build and utilize SST within a Singularity container.  
This portion of the tutorial assumes that the user has sufficient 
admin privileges to install/configure singularity and/or has obtained 
permission from the appropriate administration staff to utilize `fakeroot`.  
In this tutorial, we will provide two main paths for building and utilizing 
Singularity containers for SST.  The first method provides users the ability 
to build and utilize basic SST containers without support for networking 
across containers.  These "monolithic" containers can be utilized for initial 
testing and developing of external modules.  The second method provides users 
the ability to build and utilize advanced SST containers that support networking 
and communication (MPI) across individual container instances.  Thus, the 
advanced (parallel) method provides much greater scalability of simulations.  

### Singularity Installation <a name="singularityinstallation"></a>

In order to install Singularity on your respective system, utilize 
the instructions provided by the Singularity installation guides.  For this 
tutorial, we assume the use of Singularity version 3.8. Pay special attention 
to the guidelines associated with installing and utilizing the `Go` infrastructure.

The installation guide for Singularity 3.8 can be found here: 

[Singularity Installation](https://sylabs.io/guides/3.8/user-guide/quick_start.html#quick-installation-steps)

### Building Basic Singularity Containers <a name="buildingbasicsingularitycontainers"></a>

#### Pre-Built Formulas <a name="basicprebuildformulas"></a>

We *HIGHLY* suggest that users utilize the pre-built singularity container 
formulas.  These formulas contain basic installations of SST-Core and/or SST-Elements
that can be utilized and/or modified to serve as the basis for a functional 
SST environment.  The pre-built SST container repository is found below.  

[SST Container Repository](https://github.com/tactcomplabs/sst-containers)

This repository contains two types of singularity containers.  The first includes the SST-Core
package and the second contains the SST-Core and SST-Elements packages.  The naming
convention for the singularity `def` files is as follows:

* sstcore-VERSION-OS.def : SST-Core only
* sstpackage-VERSION-OS.def : SST-Core + SST-Elements

The version numbers supported are listed as follows:
* 11.0.0 : SST 11.0.0 Release
* MASTER : SST github master branches

The supported operating systems are listed as follows:
* ubuntu-18.04
* ubuntu-20.04
* centos7
* centos8

Prior to building the provided SST containers, you must first bootstrap the 
environment.  The repository contains a `bootstrap.sh` script that can be 
utilized to initialize the environment, find available packages and download 
the appropriate SST source packages.  Bootstrapping the environment can be performed 
as follows:

```
$> git clone https://github.com/tactcomplabs/sst-containers.git
$> cd sst-containers
$> ./bootstrap.sh
```

Building one or more singularity containers can be performed using the following commands:
(edit the commands for your particular environment)

```
$> cd ./containers/singularity/
$> sudo singularity build sstcore-11.0.0-ubuntu-18.04.sif sstcore-11.0.0-ubuntu-18.04.def
```

To create a singularity container without using `sudo` privileges it is possible to use
singularity's `fakeroot` option.  Depending on your Linux kernel version, additional
system configuration steps must be taken.

* On CentOS7 you must first run:
```
$> sudo sh -c 'echo user.max_user_namespaces=15000 >/etc/sysctl.d/90-max_net_namespaces.conf'

$> sudo sysctl -p /etc/sysctl.d /etc/sysctl.d/90-max_net_namespaces.conf
```

Once your OS is properly configured (if necessary) the easiest way to enable `fakeroot`
on singularity >= v3.5 is:
```
sudo /usr/local/bin/singularity config fakeroot --add <your_user_name>
```
The above command will modify user mappings set in `/etc/subuid` and group mappings
in `/etc/subgid`.

Once built, the container functions as normal. The `.def` files in this repo work with
or without the `fakeroot` switch and can be built using a command similar to the example below:
```
singularity build --fakeroot sstcore-11.0.0-centos7.sif sstcore-11.0.0-centos7.def
```

Verifying that the container was build successfully can be performed using the following:
```
$> singularity exec sstcore-11.0.0-ubuntu-18.04.sif sst --help
```

#### Advanced: Container Construction <a name="basiccontainerconstruction"></a>

Users also have the ability build and/or modify custom SST container definitions.  
We *HIGHLY* suggest that you utilize the existing container definitions mentioned 
above as the basis for modified containers.  The container definitions Apache 
2.0 licensed such that they can be utilized/modified without the need to commit 
the source back to the parent repository.  Further, this portion of the tutorial 
assumes that Singularity has been installed and verified to function properly 
on the target platform.

In creating new container definitions, we must first decide what the target platform 
is for the container.  CentOS (and other RedHat-derived Linux distributions) 
will utilize a `yum` package infrastructure.  Ubuntu (and other Debian-derived 
Linux distributions) will utilize a `deb` or `dpkg` package infrastructure.  

##### RedHat-Based Linux <a name="basicredhat"></a>

RedHat-derived container definitions contain seven (7) individual sections.  
The first section defines the container prologue that governs the overall layout 
of the infrastructure.  This defines the OS version, core OS mirror URL and 
the target packaging mechanism (`yum`).  An example of a CentOS7 prologue is as follows:

```
Bootstrap: yum
OSVersion: 7
MirrorURL: http://mirror.centos.org/centos-%{OSVERSION}/%{OSVERSION}/os/$basearch/
Include: yum
```

The next section, `%setup`, initializes the installation directory for SST and 
its constituent packages.  An example of a CentOS7 `%setup` section is as follows:

```
%setup
    mkdir -p ${SINGULARITY_ROOTFS}/opt/SST/11.0.0/src
```

The next section, `%files`, places the SST source package (or packages) in the 
designated source location such that the forthcoming `%post` section can 
unpack and build the package.  An example of a CentOS7 `%files` section is as follows:

```
%files
    sstcore-11.0.0.tar.gz /opt/SST/11.0.0/src/
```

The `%environment` section ensures that the correct environment variables 
are initialized in the user's environment when the container is executed.  
Any potential environment variables (shell variables) can be initialized in 
this section.  AN example of an `%environment` section is as follows:

```
%environment
    PATH=$PATH:/opt/SST/11.0.0/bin
    LD_LIBRARY_PATH="/usr/local/lib:$LD_LIBRARY_PATH"
```

The `%runscript` section performs additional tasks during the execution of the 
container that encapsulate commands outside of basic environment/shell variables.  
For CentOS, we enable the `devtoolset-8` in order to initialize support for CXX-11+ 
compilers and toolchains.  An example of doing so is as follows:

```
%runscript
    scl enable devtoolset-8 bash
```

The next section, `%post` performs the bulk of the package installation, 
compilation and configuration for the target container.  The contents 
of the `%post` section is *ONLY* executed at build time during 
container construction.  Once constructed, the `%post` section is no 
longer utilized for execution of the container.  The post section 
contains four (4) subsections.  The first subsection installs 
all the package dependencies using Yum.  If the user desires 
to install additional packages to the container, this is the best 
place to add additional packages.  The next subsection installs 
all the Python dependencies.  Again, adding Python packages can 
performed using this stage.  The next subsection initializes 
the CentOS development environment such that CXX-11 toolchains 
can be found.  Finally, we unpack, build and install SST into the 
target container image.

```
%post
    # INSTALL ALL THE PACKAGE DEPENDENCIES USING YUM
    SINGULARITY_PREPEND_PATH=/opt/rh/devtoolset-8/root/bin/
    time yum clean all
    time yum update -y
    time yum install -y centos-release-scl \
                yum-utils
    time yum-config-manager --enable rhel-server-rhscl-7-rpms
    time yum install -y \
               devtoolset-8 \
               doxygen \
               graphviz \
               time \
               mpich \
               python3 \
               python3-devel \
               python3-pip \
               automake

    # INSTALL ALL THE PYTHON PACKAGES
    time yum clean all
    time python3 -m pip install --find-links=. \
          black \
          matplotlib \
          matplotlib-label-lines \
          networkx==2.4 \
          numpy==1.19.1 \
          pandas \
          setuptools \
          xmltodict \
          wheel

    # INITIALIZE THE ENVIRONMENT
    set -e
    source scl_source enable devtoolset-8 || true
    PATH=/opt/rh/devtoolset-8/root/bin:$PATH

    # UNPACK, BUILD AND INSTALL SST
    cd /opt/SST/11.0.0/src
    tar --no-same-owner -xvzf sstcore-11.0.0.tar.gz
    cd sstcore-11.0.0; ./configure --disable-mpi --prefix=/opt/SST/11.0.0; time make -j all; make install
```

The final section contains a help message describing the contents of the 
repsective container.  An example is as follows:

```
%help
    SST 11.0.0-Centos 7: Creates a basic SST Core image with the necessary software dependencies
```

##### Ubuntu/Debian-Based Linux <a name="basicubuntu"></a>

Ubuntu/Debian-derived container definitions contain seven (7) individual sections.  
The first section defines the container prologue that governs the overall layout 
of the infrastructure.  This defines the OS version, core OS mirror URL and 
the target packaging mechanism (`debootstrap`).  An example of an Ubuntu prologue is as follows:

```
Bootstrap: debootstrap
OSVersion: bionic
MirrorURL: http://us.archive.ubuntu.com/ubuntu/
Include: software-properties-common
```

The next section, `%setup`, initializes the installation directory for SST and 
its constituent packages.  An example of a Ubuntu `%setup` section is as follows:

```
%setup
  mkdir -p ${SINGULARITY_ROOTFS}/opt/SST/11.0.0/src
```

The next section, `%files`, places the SST source package (or packages) in the 
designated source location such that the forthcoming `%post` section can 
unpack and build the package.  An example of a Ubuntu `%files` section is as follows:

```
%files
  sstcore-11.0.0.tar.gz /opt/SST/11.0.0/src/
```

The `%environment` section ensures that the correct environment variables 
are initialized in the user's environment when the container is executed.  
Any potential environment variables (shell variables) can be initialized in 
this section.  AN example of an `%environment` section is as follows:

```
%environment
  PATH=$PATH:/opt/SST/11.0.0/bin
```

The `%runscript` section performs additional tasks during the execution of the 
container that encapsulate commands outside of basic environment/shell 
variables.  An example of doing so is as follows:

```
%runscript
  echo "SSTCore 11.0.0 using Ubuntu 18.04 created $NOW"
  echo "Arguments utilized: $*"
  exec echo "$@"
```

The next section, `%post` performs the bulk of the package installation, 
compilation and configuration for the target container.  The contents 
of the `%post` section is *ONLY* executed at build time during 
container construction.  Once constructed, the `%post` section is no 
longer utilized for execution of the container.  The post section 
contains three (3) subsections.  The first subsection installs 
all the package dependencies using Apt.  If the user desires 
to install additional packages to the container, this is the best 
place to add additional packages.  The next subsection installs 
all the Python dependencies.  Again, adding Python packages can 
performed using this stage. Finally, we unpack, build and install SST into the 
target container image.

```
  # INSTALL ALL THE PACKAGE DEPENDENCIES USING APT
  add-apt-repository 'deb http://archive.ubuntu.com/ubuntu bionic main universe'
  add-apt-repository 'deb http://archive.ubuntu.com/ubuntu bionic-security main universe'
  add-apt-repository 'deb http://archive.ubuntu.com/ubuntu bionic-updates main universe'
  apt-get update && apt-get install -y \
    build-essential \
    automake \
    openmpi-bin \
    libopenmpi-dev \
    python3 \
    python3-dev \
    python3-pip \
    doxygen

  # INSTALL ALL THE PYTHON PACKAGES
  python3 -m pip install --find-links=. \
    black \
    matplotlib \
    matplotlib-label-lines \
    networkx==2.4 \
    numpy==1.19.1 \
    pandas \
    setuptools \
    xmltodict \
    wheel

  # UNPACK, BUILD AND INSTALL SST
  cd /opt/SST/11.0.0/src
  tar --no-same-owner -xvzf sstcore-11.0.0.tar.gz
  cd sstcore-11.0.0; ./configure --disable-mpi --prefix=/opt/SST/11.0.0; make -j all; make install

  # OPTIONAL
  NOW=`date`
  echo "export NOW=\"${NOW}\"" >> $SINGULARITY_ENVIRONMENT
```

The final section contains a help message describing the contents of the 
respective container.  An example is as follows:

```
%help
  SST 11.0.0-Ubuntu 18.04: Creates a basic SST Core image with the necessary software dependencies
```

### Building Parallel Singularity Containers <a name="buildingparallelsingularitycontainers"></a>

#### Singularity Networking <a name="singularitynetworking"></a>

Singularity currently supports containerized network options with the [CNI](https://github.com/containernetworking/cni) 
infrastructure.  As a result, users have the ability to enable/disable host-native 
network access, change the DNS mechanisms, change the native container hostname and initiate 
port forwarding.  In general, the standard network options have been tested to work 
with MPI parallel execution across nodes.  However, in some cases, users may be required 
to enable/disable specific network interfaces in order to ensure that the correct high 
performance network is selected for MPI execution.  This can be done be utilizing 
the `network` command and explicitly enabling interfaces as follows:

```
singularity exec --net --network ptp /path/to/container.sif sst_info
```

For more information regarding specific network configuration parameters as well as the 
CNI infrastructure, refer to the singularity network options reference below.  

#### Pre-Built Formulas <a name="parallelprebuildformulas"></a>

We *HIGHLY* suggest that users utilize the pre-built singularity container 
formulas.  These formulas contain basic installations of SST-Core and/or SST-Elements
that can be utilized and/or modified to serve as the basis for a functional 
SST environment.  Each of the pre-built containers contains the necessary logic 
and mechanisms to build and execute parallel Singularity SST containers.  Note that the 
containers utilize native OpenMPI installs (using `apt`) for the Ubuntu containers and 
custom-built OpenMPI installs for the CentOS containers.  This ensures that the minimum 
version of the OpenMPI infrastructure is compatible with SST.  The CentOS containers are 
prebuilt with OpenMPI version 4.0.5 (release).  

Please note that the host execution system must also have the *same* version of OpenMPI 
installed.  Otherwise, the OpenMPI ORTD/ORTE launch mechanisms will not function correctly!
Finding the respective OpenMPI version installed in the container can be done via the following 
commands:

```
singuarity exec sstcore-11.0.0-ubuntu-18.04.sif ompi_info

Package: Open MPI buildd@lcy01-amd64-009 Distribution
                Open MPI: 2.1.1
  Open MPI repo revision: v2.1.0-100-ga2fdb5b
   Open MPI release date: May 10, 2017
                Open RTE: 2.1.1
  Open RTE repo revision: v2.1.0-100-ga2fdb5b
   Open RTE release date: May 10, 2017
                    OPAL: 2.1.1
      OPAL repo revision: v2.1.0-100-ga2fdb5b
       OPAL release date: May 10, 2017
                 MPI API: 3.1.0
            Ident string: 2.1.1
                  Prefix: /usr
 Configured architecture: x86_64-pc-linux-gnu

```

#### Executing Containers with MPI <a name="mpi"></a>

The best path to building and executing SST using MPI within Singularity containers 
is utilizing the host-based launch mechanism.  This utilizes the host infrastructure, 
including the host's batch scheduling infrastructure, to launch a series of singularity 
containers across the nodes in a compute cluster for parallel execution.  This is analogous 
to executing SST using MPI except the SST binaries and libraries reside within the container.  

Fundamentally speaking, this is done via running singularity under the standard `mpirun` command 
set.  The user specifies the number of MPI ranks to utilize and optionally a host file with the target 
nodes.  The SST commands are then `exec'd` inside the target singularity container as shown above.  We highly 
recommend utilize the system's standard batch scheduling infrastructure in order to ease the creation 
and management of proper host files for execution.  We provide an example batch script for the Slurm 
infrastructure to execute across 8 compute nodes below.  

```
#!/bin/bash
# Sample Slurm Script

#-- load the modules
module load openmpi/gcc/64/2.1.1 singularity/3.8.0

#-- Setup the hosts
srun hostname > output.txt

#-- Execute SST
mpirun --hostfile output.txt -N 8 singularity exec /path/to/container.sif sst -v /path/to/sst/test.py
```

You can submit the aforementioned Slurm script to the batch system using the following command:
```
sbatch -N8 slurm.sh
```

### Singularity References <a name="singularityreferences"></a>
- [Singularity Installation](https://sylabs.io/guides/3.8/user-guide/quick_start.html#quick-installation-steps)
- [Singularity Networking Options](https://sylabs.io/guides/3.0/user-guide/networking.html)
- [Singularity Container Definition Docs](https://sylabs.io/guides/3.0/user-guide/definition_files.html)
- [SST Container Repository](https://github.com/tactcomplabs/sst-containers)

## Docker <a name="docker"></a>

TBD

## Spack <a name="spack"></a>

Spack currently has standard package builds for SST.  You can install the standard builds as follows:
```
spack install sst-core sst-elements
```
