# Level 1 Tutorial
# What is SST: <h1>
  To combat the ever-changing and multifaceted technologies of the HPC community, an effort was made to create a unified tool box used to evaluate these challenges. The Structural Simulation Toolkit (SST) an open, modular, parallel, multi-criteria, multi-scale simulation framework  was created to design and procure HPC systems. By providing a number of interfaces and utilizes for simulation models, SST, will demolish the need for simulations for individual system components. A now unified framework exists for parallel simulation of large machines at multiple levels. The SST has been used in variety of network, memory and applications.
## Components of SST: <h2> 
  To meet these requirements, the SST is comprised of a simple simulation core that contains a parallel discrete event simulator (DES) and support services for simulation. components, representing hardware systems such as processors, network switches, or memory devices, interface with the simulation core to communicate and operate with a common notion of timeframe. The simulation core also provides support services such as power and area estimation, checkpointing, configuration/initialization of the simulation, and statistics gathering. The SST’s modular interface eases the integration of existing simulators into a common framework and is licensed under a BSD-like license. In addition, the components of the SST model are compatible with external models such as Gem5 and DRAMSim2. Because of the open source core, the modular framework is easily extensible with new models. 
The SST uses a (DES) model layered on top of MPI.  The simulations are comprised of components or building blocks of the simulation model are connected by links. These components interact by sending events over links—with each link having a minimum latency. In addition, components can load subcomponents and modules for even more functionality and customization. To achieve better performance, the SST uses a conservative (i.e. no roll- back) distance-based optimization. At the start of the simulation, the system topology is represented by a graph with components as nodes and connections between them as edges, with each edge labeled with the minimum latency between the connected components. The Zoltan library is then used to partition components across the MPI ranks with the goal of balancing the load and partitioning across the highest latency links. Tests indicate that the algorithm is scalable and shows less than 25% overhead at 128 ranks (11,904 simulated components) compared to a single rank for detailed simulations. 

![image](https://user-images.githubusercontent.com/74792926/121704586-e5454680-caa1-11eb-8c1f-b33daad81498.png)
# Table of Contents <h2>
  Components (models)—pieces of a project 
Composition—how the components are arranged and connected.
Events—passed back and forth between components to progress the simulation

Services available from SST:
-Components
-Sub Components
-Links
-Events
-Clocks
-Configurable parameters
-Random number generator
-Statistic API
-Configurable statistics output
-Element library
	-Modular 
	-Pluggable
  
  
![image](https://user-images.githubusercontent.com/74792926/125094895-a3f48680-e0a1-11eb-80a0-3b9699636ace.png)
