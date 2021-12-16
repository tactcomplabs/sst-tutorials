# Level 2 Tutorial

## Table of Contents
1. [Parallel SST Runs](#ParallelSSTRuns)
2. [Advanced Python Scripting](#AdvPythonScripting)
3. [Advanced JSON Scripting](#AdvJSONScripting)
4. [Advanced Runtime Functions](#AdvRuntimeFunctions)
5. [Loading Parallel Graphs](#LoadingParallelGraphs)
6. [Links to External SST Components](#ExternSSTComp)

## Parallel SST runs: <a name="ParallelSSTRuns"></a>

## Advanced Python Scripting: <a name="AdvPythonScripting"></a>

## Advanced JSON Scripting: <a name="AdvJSONScripting"></a>

One of the unique features of the latest versions of SST is the ability 
to specify simulation models using Python or JSON input files.  The JSON 
simulation inputs provide users the ability to rapidly generate complex 
simulations using external tools that generate human-readable JSON files.
Currently, the SST JSON simulation loader is *automatically* invoked 
when a JSON file is specified on the command line.  This is analogous 
to the following:

```
$> sst /path/to/simulation.json
```

The SST JSON input files are organized in a very specific manner 
such that global program options, components, subcomponents and 
links are specified in unique blocks.  These blocks are subsequently 
linked together in order to form a simulation graph.  Currently, 
the SST JSON model loader accepts all of the standard program options, 
component attributes, recursively nested subcomponents (also with attributes) 
and link configuration data.  The SST JSON loader does not currently handle 
global statistics options.

In the following example, we have a sample JSON input file with a global
program options block that include `verbose`, `stop-at` and `timeVortex` 
parameters.  The syntax for the global parameters is identical to that 
of the Python scripting format.  We also have a `components` block 
that includes four components.  For each component, we define 
the `name`, `type` (specifying the component model) and a set of 
`params`.  Notice for `component.1` we also specify a set of 
`subcomponents`. Finally, we specify a `links` block that includes 
the `left` and `right` links, their respective ports, components and 
latency parameters.

```
{
  "program_options":{
    "verbose": "1",
    "stop-at": "25us",
    "timeVortex": "sst.timevortex.priority_queue"
  },
  "components":[
    {
      "name": "component.0",
      "type": "coreTestElement.coreTestComponent",
      "params": {
        "workPerCycle": "1000",
        "commSize": "100",
        "commFreq": "1000"
      }
    },
    {
      "name": "component.1",
      "type": "coreTestElement.coreTestComponent",
      "params": {
        "workPerCycle": "1000",
        "commSize": "100",
        "commFreq": "1000"
      },
      "subcomponents": [
        {
          "slot_name": "generator",
          "slot_number": 0,
          "type": "miranda.Stencil3DBenchGenerator",
          "params": {
            "verbose": "0",
            "nx": "30",
            "ny": "20",
            "nz": "10"
          }
        }
      ]
    },
    {
      "name": "component.2",
      "type": "coreTestElement.coreTestComponent",
      "params": {
        "workPerCycle": "1000",
        "commSize": "100",
        "commFreq": "1000"
      }
    },
    {
      "name": "component.3",
      "type": "coreTestElement.coreTestComponent",
      "params": {
        "workPerCycle": "1000",
        "commSize": "100",
        "commFreq": "1000"
      }
    }
  ],
  "links": [
    {
      "name": "link_s_0_10",
      "left": {
        "component": "component.0",
        "port": "Nlink",
        "latency": "10000ps"
      },
      "right": {
        "component": "component.1",
        "port": "Slink",
        "latency": "10000ps"
      }
    },
    {
      "name": "link_s_0_90",
      "left": {
        "component": "component.2",
        "port": "Slink",
        "latency": "10000ps"
      },
      "right": {
        "component": "component.3",
        "port": "Nlink",
        "latency": "10000ps"
      }
    }
  ]
}
```

In addition to writing files directly in the JSON model format, 
users may also utilize SST to create JSON input files with existing 
Python input files.  This offers a convenient migration path from 
existing simulations written in the original Python scripting 
form to a more human-readable JSON file.
This can be done using the following command:

```
$> sst --output-json=/path/to/simulation.json simulation.py
```

As an example of using this feature, we have generated a set of 
sample JSON inputs using existing SST test cases.  These are listed in 
the table below:

|  **Description**  | **Python Input** | **JSON Input** |
|:-|:-|:-|
| Test Component | [Test Component Python](samples/test_Component.py) | [Test Component JSON](samples/test_Component.json)|
| Miranda Stencil3D | [Miranda 3D Python](samples/stencil3dbench.py)| [Miranda3D JSON](samples/stencil3dbench.json)|
| Merlin 256 Node FatTree | [Merlin 256 Python](samples/fattree_256_test.py)| [Merlin 256 JSON](samples/fattree_256_test.json)|


## Advanced Runtime Functions: <a name="AdvRuntimeFunctions"></a>

## Loading Parallel Graphs: <a name="LoadingParallelGraphs"></a>

With the latest updates to SST, users now have the ability to build and execute 
simulation inputs in parallel.  Historically, SST would assign rank-0 to 
load, connect, validate and distribute the simulation graph data to all 
the subsequent MPI ranks.  While this path works well for reasonably sized 
graphs, very large graph inputs require a significant amount of memory 
during graph construction, thus placing a large burden on rank-0.  When 
using the parallel graph loader, users have the ability to split a simulation 
graph into `N` files for `N` ranks of a simulation whereby each `N-th` rank
independently loads only its portion of the simulation graph.  SST will handle 
the hard portion of wiring up the graph across MPI ranks.  However, the input 
files for each rank *must* contain the same `program_options`.  In this manner, 
each rank is configured to operate under the same global options.

Executing SST with the parallel graph loader requires that the user create 
a single input file for each rank of the parallel simulation.  As a result, 
if the user seeks to utilize `4` ranks, then `4` input files must be present.  
The files are named using a monotonically increasing integer in the following 
form (for `n` ranks):
```
FILE0.json
FILE1.json
...
FILE{n-1}.json
```

When executing using the parallel graph loader, we need additional options 
and file prefixes on the command line.  First, we must specify the 
`--parallel-load` option in order to trigger the SST core to load 
the per-rank simulation files.  Next, we must specify the base file 
prefix as the input to the simulation.  Using our example above (`FILE0.json`) 
would be specified as `FILE.json`.  An example of doing so resembles the following:

```
$> mpirun --hostfile output.txt -n2 sst --parallel-load FILE.json
```

In addition to the aforementioned parallel graph loading, the latest development 
source for SST also supports converting large simulation graphs into their 
parallel form.  This allows users to easily port their existing Python-based 
simulation inputs to parallel JSON inputs.  Users may accomplish this similar to the 
basic JSON convertor outlined above by adding the `--output-json=FILE.json` 
to the parallel run.  Note that the file name specified in the `--output-json` flag 
must contain a `.json` file extension.  SST will automatically split the input 
Python file into the appropriate `N` output JSON files in the same monotonically 
increasing form as outlined above.

```
$> mpirun --hostfile output.txt -n2 sst --parallel-load --output-json=FILE.json FILE.py
```

The following are several candidate examples of parallel JSON input files using 
standard SST tests.

|  **Description**  | **Ranks** | **Rank Files** |
|:-|:-:|:-|
| Miranda Stencil3D | 2 | [Rank0](samples/parallel/stencil3dbench0.json) [Rank1](samples/parallel/stencil3dbench1.json)|
| Merlin 256 Node FatTree | 4 | [Rank0](samples/parallel/fattree_256_test_parallel0.json) [Rank1](samples/parallel/fattree_256_test_parallel1.json) [Rank2](samples/parallel/fattree_256_test_parallel2.json) [Rank3](samples/parallel/fattree_256_test_parallel3.json)|

## Links to External SST Components: <a name="ExternSSTComp"></a>
