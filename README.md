## MaterialX / glTF Procedurals Interop

[![Build Status](https://github.com/KhronosGroup/glTF-MaterialX-Converter/workflows/main/badge.svg)](https://github.com/KhronosGroup/glTF-MaterialX-Converter/actions?query=branch%3Amain)

### Introduction

This package supports the bi-directional translation between MaterialX material graphs and the glTF Procedural Textures extension.

- The Khronos extensions can be found here:
  - <a href="https://github.com/KhronosGroup/glTF/tree/KHR_texture_procedurals/extensions/2.0/Khronos/KHR_texture_procedurals">KHR_texture_procedurals</a>
  - <a href="https://github.com/KhronosGroup/glTF/tree/KHR_texture_procedurals/extensions/2.0/Vendor/EXT_texture_procedurals_mx_1_39">EXT_texture_procedurals_mx_1_39</a>
- The MaterialX specification documents can be found <a href="https://github.com/AcademySoftwareFoundation/MaterialX/tree/main/documents/Specification">here</a>

### Dependencies

- The 1.39 release (or patch releases) of MaterialX available on 
<a href="https://pypi.org/project/MaterialX/">PyPi</a> is required.
- The <code>jsonschema</code> package if Schema validation is desired

### Setup

The Github repository can be forked / cloned locally and the package built using `pip` as follows from the root folder:

`pip install .`

All dependencies listed will be installed if required. 

#### Command Line Interfaces

To convert from a MaterialX document to produce a glTF JSON document the
`materialx_to_gltf.py` utility script may be used.

The following is an example converting a sample file found in the test folder.
The results are saved to a file called `checkerboard_graph.gltf`.

<code>
python source/gltf_materialx_converter/materialx_to_gltf.py "tests/data/checkerboard_graph.mtlx"
</code>

### Documentation

#### API

API documentation can be found <a href="./documents/html/index.html">here</a>

Documentation can be generated by running `doxygen` from the "documents" folder.

### Tests

The following command can be used to run tests from the root folder:

<code>
python -m unittest discover -s tests -p "test_*.py"
</code>

### Supported MaterialX Configurations

Only specific configurations of MaterialX can be mapped to glTF Texture Procedurals.
1. There must be a `surfacematerial` material node
2. There must be a `glTF PBR` node connected to the surface shader input of the material.
3. A single `nodegraph` with a `color3` output node which is connected to the base color on the surface shader. The constant node can be replaced with the desired set of
nodes, and one or more inputs may be specified to route data into the `nodegraph`. 

Below is a diagram the minimal graph and it's corresponding MaterialX XML document 
from the test data area. There are no inputs specified on the `nodegraph`.

```mermaid
graph LR
    gltf_Material([surfacematerial:material])
    style gltf_Material   fill:#090, color:#FFF
    gltf_Shader[gltf_pbr:surfaceshader]
    subgraph gltf_procedural
    gltf_procedural_output_color4([output:color3])
    style gltf_procedural_output_color4  fill:#09D, color:#FFF
    gltf_procedural_constant_color4([constant:color3:1,1,1])
    style gltf_procedural_constant_color4  fill:#888, color:#000
    end
    gltf_Shader --"surfaceshader"--> gltf_Material
    gltf_procedural_constant_color4 --> gltf_procedural_output_color4
    gltf_procedural_output_color4 --"base_color"--> gltf_Shader
```

<a href=".//tests/data/minimal_graph.mtlx">Minimal MTLX Configuration File</a>


#### Sample Data

The following is a small set of example files used for unit testing. For each MaterialX file the resulting glTF file is given, along with a diagram of how the graph looks and reference image rendered
using the `MaterialXView` sample application which is available as part of the core MaterialX distribution.

<details open><summary>Examples</summary>

<table>
<tr>
<th>Description
<th>Documents
<th>Reference Image

<tr>
<td>The following is graph that adds 2 inputs and outputs the results to a glTF PBR shading node.

```mermaid
graph TB
    subgraph graph1
    graph1_myin1([input:1,0,0])
    style graph1_myin1  fill:#09D, color:#FFF
    graph1_myin2([input:0.94902, 0.768627, 0.109804])
    style graph1_myin2  fill:#09D, color:#FFF
    graph1_output_color4([output])
    style graph1_output_color4  fill:#09D, color:#FFF
    graph1_add_color4[add]
    end
    Default([surfacematerial])
    style Default   fill:#090, color:#FFF
    gltf_mat[gltf_pbr]
    graph1_myin2 --"in1"--> graph1_add_color4
    graph1_myin1 --"in2"--> graph1_add_color4
    graph1_add_color4 --> graph1_output_color4
    gltf_mat --"surfaceshader"--> Default
    graph1_output_color4 --"base_color"--> gltf_mat
```
</td>
<td>
<a href=".//tests/data/add_graph.mtlx">MTLX</a>
<a href=".//tests/data/add_graph.gltf">GLTF</a>
</td>
<td><img src=".//tests/data/add_graph.png">
</td>
</tr>

<tr>
<td>The following is a pattern graph that produces a checkerboard pattern. 
The two input colors, and a texture coordinate tiling option are exposed on the node graph. The output is a color which is routed to a downstream glTF PBR shading node (glTF material).

```mermaid
graph TB
    subgraph NG_main
    NG_main_uvtiling([uvtiling:8,8])
    style NG_main_uvtiling  fill:#09D, color:#FFF
    NG_main_color1([color1:1,0.094118,0.031373])
    style NG_main_color1  fill:#09D, color:#FFF
    NG_main_color2([color2:0.035294,0.090196,0.878431])
    style NG_main_color2  fill:#09D, color:#FFF
    NG_main_output_N_mtlxmix_out([output_N_mtlxmix_out])
    style NG_main_output_N_mtlxmix_out  fill:#09D, color:#FFF
    NG_main_N_mtlxmix[N_mtlxmix]
    NG_main_N_mtlxdotproduct[N_mtlxdotproduct]
    NG_main_N_mtlxmult[N_mtlxmult]
    NG_main_N_mtlxsubtract[N_mtlxsubtract]
    NG_main_N_mtlxfloor[N_mtlxfloor]
    NG_main_N_modulo[N_modulo]
    NG_main_Texcoord[Texcoord:0]
    end
    Gltf_pbr[Gltf_pbr]
    MAT_Gltf_pbr([MAT_Gltf_pbr])
    style MAT_Gltf_pbr   fill:#090, color:#FFF
    NG_main_N_mtlxmix --> NG_main_output_N_mtlxmix_out
    NG_main_color1 --"fg"--> NG_main_N_mtlxmix
    NG_main_color2 --"bg"--> NG_main_N_mtlxmix
    NG_main_N_modulo --"mix"--> NG_main_N_mtlxmix
    NG_main_N_mtlxfloor --"in1"--> NG_main_N_mtlxdotproduct
    NG_main_Texcoord --"in1"--> NG_main_N_mtlxmult
    NG_main_uvtiling --"in2"--> NG_main_N_mtlxmult
    NG_main_N_mtlxmult --"in1"--> NG_main_N_mtlxsubtract
    NG_main_N_mtlxsubtract --"in"--> NG_main_N_mtlxfloor
    NG_main_N_mtlxdotproduct --"in1"--> NG_main_N_modulo
    NG_main_output_N_mtlxmix_out --"base_color"--> Gltf_pbr
    Gltf_pbr --"surfaceshader"--> MAT_Gltf_pbr
```
</td>
<td>
<a href="./tests/data/checkerboard_graph.mtlx">MTLX</a>
<a href="./tests/data/checkerboard_graph.gltf">GLTF</a>
</td>
<td><img src="./tests/data/checkerboard_graph.png">
</td>
</tr>

</table>

</details>