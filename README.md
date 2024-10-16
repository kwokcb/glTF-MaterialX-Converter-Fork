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

#### API Example

The following is a simple example of using the API to convert from MaterialX to glTF

<pre>
# Import support modules
import MaterialX as mx
import json

# Import conversion module utilities
from gltf_materialx_converter import converter as MxGLTFPT
from gltf_materialx_converter import utilities as MxGLTFPTUtil

input_file = "my_file.mtlx" # Replace with desired file name

# Set up definitions and read in a sample file
stdlib, libFiles = MxGLTFPTUtil.load_standard_libraries()
mxdoc = MxGLTFPTUtil.create_working_document([stdlib])
mx.readFromXmlFile(mxdoc, input_file)

# Convert
converter = MxGLTFPT.glTFMaterialXConverter()
json_string, status = converter.materialX_to_glTF(mxdoc)

# Write result to disk
gltf_file = input_file.replace('.mtlx', '.gltf')
with open(gltf_file, 'w') as f:
    f.write(json_string)
</pre>

A sample `Jupyter` notebook is available <a href="./documents/notebook.ipynb">here</a>

### Documentation

#### API

API documentation can be found <a href="./documents/html/index.html">here</a>

Documentation can be generated by running `doxygen` from the "documents" folder.
It is assumed that `Doxygen` has been installed locally.

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

Any document level qualifiers must be pre-resolved when converting rom MaterialX. This includes any `fileprefix` qualifiers for image file names.

<table>
<tr>
<th>Description
<th>Documents
<th>Reference Image

<tr>
<td> 
A sample minimal graph routing a constant color to the downstream shader.
There are no inputs specified on the `nodegraph`.

```mermaid
graph TB
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
</td>
<td>
<a href=".//tests/data/minimal_graph.mtlx">MTLX</a>
<a href=".//tests/data/minimal_graph.gltf">GLTF</a>
<a href=".//tests/data/minimal_graph.usda">USD</a>
</td>
<td><img src=".//tests/data/minimal_graph.png">
</td>
</tr>
</table>

#### Test Data

The following is a set of example files used for unit testing. The term "Compound nodes" refers to nodes
which are implemented as node graphs themselves ("functional graphs" in MaterialX terminology)

For each `MaterialX` file the resulting `glTF` file is given, along with a diagram of how the graph looks and reference image.
A sample conversion from MaterialX to `USDShade` network is provided where applicable.

<details open><summary>Examples</summary>

<table>
<tr>
<th>Description
<th>Documents
<th>Reference Image

<tr>
<td> 
The following is a simple graph which adds two colors together.

- Graph count: single
- Graph inputs: multiple
- Graph outputs: single
- Stream inputs: no
- Compound nodes: none
- Downstream shader: glTF PBR

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
<a href=".//tests/data/add_graph.usda">USD</a>
</td>
<td><img src=".//tests/data/add_graph.png">
</td>
</tr>

<tr>
<td>

The following is a pattern graph that produces a checkerboard pattern. 
The two input colors, and a texture coordinate tiling option are exposed on the node graph. The output is a color which is routed to a downstream glTF PBR shading node (glTF material).

- Graph count: single
- Graph inputs: multiple
- Graph outputs: single
- Stream inputs: yes
- Compound nodes: none
- Downstream shader: glTF PBR

```mermaid
graph TB
    subgraph NG_main
    NG_main_uvtiling([input:vector2:8,8])
    style NG_main_uvtiling  fill:#09D, color:#FFF
    NG_main_color1([input:color3:1,0.094118,0.031373])
    style NG_main_color1  fill:#09D, color:#FFF
    NG_main_color2([input:color3:0.035294,0.090196,0.878431])
    style NG_main_color2  fill:#09D, color:#FFF
    NG_main_output_N_mtlxmix_out([output:color3])
    style NG_main_output_N_mtlxmix_out  fill:#09D, color:#FFF
    NG_main_N_mtlxmix[mix:color3]
    NG_main_N_mtlxdotproduct[dotproduct:float]
    NG_main_N_mtlxmult[multiply:vector2]
    NG_main_N_mtlxsubtract[subtract:vector2]
    NG_main_N_mtlxfloor[floor:vector2]
    NG_main_N_modulo[modulo:float]
    NG_main_Texcoord[texcoord:vector2:0]
    end
    Gltf_pbr[gltf_pbr:surfaceshader]
    MAT_Gltf_pbr([surfacematerial:material])
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
<a href="./tests/data/checkerboard_graph.usda">USD</a>
</td>
<td><img src="./tests/data/checkerboard_graph.png">
</td>
</tr>

<tr>
<td>Pattern graph only without any materials.

- Graph count: single
- Graph inputs: multiple
- Graph outputs: single
- Stream inputs: no
- Compound nodes: yes
- Downstream shader: none

```mermaid
graph TB
    subgraph splittb_graph
    splittb_graph_output_color4([output_color4])
    style splittb_graph_output_color4  fill:#09D, color:#FFF
    splittb_graph_splittb_color4[splittb_color4]
    splittb_graph_texcoord_vector[texcoord_vector:0]
    end
    subgraph checker_graph
    checker_graph_output_color5([output_color5])
    style checker_graph_output_color5  fill:#09D, color:#FFF
    checker_graph_checkerboard_color4[checkerboard_color4]
    checker_graph_texcoord_vector3[texcoord_vector3:0]
    end
    splittb_graph_texcoord_vector --"texcoord"--> splittb_graph_splittb_color4
    splittb_graph_splittb_color4 --> splittb_graph_output_color4
    checker_graph_texcoord_vector3 --"texcoord"--> checker_graph_checkerboard_color4
    checker_graph_checkerboard_color4 --> checker_graph_output_color5
```

</td>
<td><a href="./tests/data/no_material.mtlx">MTLX</a>
<a href="./tests/data/no_material.gltf">GLTF</a>
<td><img src="./tests/data/no_material.png">
</tr>

<tr>
<td>
Pattern connected to an unsupported (non-glTF) PBR downstream shader.

- Graph count: single
- Graph inputs: multiple
- Graph outputs: single
- Stream inputs: no
- Compound nodes: yes
- Downstream shader: "standard surface"

```mermaid
graph TB
    subgraph splittb_graph
    splittb_graph_output_color4([output_color4])
    style splittb_graph_output_color4  fill:#09D, color:#FFF
    splittb_graph_splittb_color4[splittb_color4]
    splittb_graph_texcoord_vector3[texcoord_vector3:0]
    end
    subgraph checker_graph
    checker_graph_output_color5([output_color5])
    style checker_graph_output_color5  fill:#09D, color:#FFF
    checker_graph_checkerboard_color4[checkerboard_color4]
    checker_graph_texcoord_vector3[texcoord_vector3:0]
    end
    standard_surface_surfaceshader1[standard_surface_surfaceshader1]
    surfacematerial_material1([surfacematerial_material1])
    style surfacematerial_material1   fill:#090, color:#FFF
    surfacematerial_material2([surfacematerial_material2])
    style surfacematerial_material2   fill:#090, color:#FFF
    standard_surface_surfaceshader2[standard_surface_surfaceshader2]
    splittb_graph_splittb_color4 --> splittb_graph_output_color4
    splittb_graph_texcoord_vector3 --"texcoord"--> splittb_graph_splittb_color4
    checker_graph_checkerboard_color4 --> checker_graph_output_color5
    checker_graph_texcoord_vector3 --"texcoord"--> checker_graph_checkerboard_color4
    splittb_graph_output_color4 --"base_color"--> standard_surface_surfaceshader1
    standard_surface_surfaceshader1 --"surfaceshader"--> surfacematerial_material1
    standard_surface_surfaceshader2 --"surfaceshader"--> surfacematerial_material2
    checker_graph_output_color5 --"base_color"--> standard_surface_surfaceshader2
```

</td>
<td><a href="./tests/data/unsupported_stdsurf.mtlx">MTLX</a>
<a href="./tests/data/unsupported_stdsurf.gltf">GLTF</a>
<a href="./tests/data/unsupported_stdsurf.usda">USD</a>
</td>
<td><img src="./tests/data/unsupported_stdsurf.png"> (original)
<img src="./tests/data/no_material.png"> (skipping material )
</td>

<tr>
<td>Pattern graph using a file texture

- Graph count: single
- Graph inputs: none
- Graph outputs: single
- Stream inputs: yes
- File inputs: single. Non-default filtering.
- Compound nodes: yes. UV placement.
- Downstream shader: gltf PBR

```mermaid
graph TB
    subgraph nodegraph1
    nodegraph1_output_color3([output_color3:color3])
    style nodegraph1_output_color3  fill:#09D, color:#FFF
    nodegraph1_image_color3[image_color3:color3]
    nodegraph1_place2d_vector2[place2d_vector2:vector2]
    nodegraph1_texcoord_vector2[texcoord_vector2:vector2:1]
    end
    gltf_pbr_surfaceshader[gltf_pbr_surfaceshader:surfaceshader]
    surfacematerial([surfacematerial:material])
    style surfacematerial   fill:#090, color:#FFF
    nodegraph1_image_color3 --> nodegraph1_output_color3
    nodegraph1_place2d_vector2 --"texcoord"--> nodegraph1_image_color3
    nodegraph1_texcoord_vector2 --"texcoord"--> nodegraph1_place2d_vector2
    nodegraph1_output_color3 --"base_color"--> gltf_pbr_surfaceshader
    gltf_pbr_surfaceshader --"surfaceshader"--> surfacematerial
```

</td>
<td><a href="./tests/data/bindings/gltf_simple_filetexture.mtlx">MTLX</a>
<a href="./tests/data/bindings/gltf_simple_filetexture.gltf">GLTF</a>
<a href="./tests/data/bindings/gltf_simple_filetexture.usda">USD</a>
<td><img src="./tests/data/bindings/gltf_simple_filetexture.png">
</tr>

<tr>
<td>Pattern graph using using multiple file textures with different texture placements and a shared input stream.

- Graph count: single
- Graph inputs: none
- Graph outputs: single
- Stream inputs: yes 
- File inputs: multiple 
- Compound nodes: yes.
- Downstream shader: gltf PBR

```mermaid
graph TB
    gltf_pbr_surfaceshader[gltf_pbr:surfaceshader]
    surfacematerial([surfacematerial:material])
    style surfacematerial   fill:#090, color:#FFF
    subgraph nodegraph1
    nodegraph1_output_color3([output:color3])
    style nodegraph1_output_color3  fill:#09D, color:#FFF
    nodegraph1_texcoord_vector2[texcoord:vector2:0]
    nodegraph1_place2d_vector3[place2d:vector2]
    nodegraph1_multiply_color4[multiply:color3]
    nodegraph1_image_color4[image:color3]
    nodegraph1_image_color3[image:color3]
    nodegraph1_place2d_vector2[place2d:vector2]
    end
    nodegraph1_output_color3 --"base_color"--> gltf_pbr_surfaceshader
    gltf_pbr_surfaceshader --"surfaceshader"--> surfacematerial
    nodegraph1_multiply_color4 --> nodegraph1_output_color3
    nodegraph1_texcoord_vector2 --"texcoord"--> nodegraph1_place2d_vector3
    nodegraph1_image_color3 --"in1"--> nodegraph1_multiply_color4
    nodegraph1_image_color4 --"in2"--> nodegraph1_multiply_color4
    nodegraph1_place2d_vector3 --"texcoord"--> nodegraph1_image_color4
    nodegraph1_place2d_vector2 --"texcoord"--> nodegraph1_image_color3
    nodegraph1_texcoord_vector2 --"texcoord"--> nodegraph1_place2d_vector2
```

</td>
<td><a href="./tests/data/bindings/gltf_shared_filetexture.mtlx">MTLX</a>
<a href="./tests/data/bindings/gltf_shared_filetexture.gltf">GLTF</a>
<a href="./tests/data/bindings/gltf_shared_filetexture.usda">USD</a>
<td><img src="./tests/data/bindings/gltf_shared_filetexture.png">
</tr>

<tr>
<td>Pattern graph using using multiple file textures routed to different outputs. Each output
is connected to a different downstream shader.

- Graph count: single
- Graph inputs: none
- Graph outputs: single
- Stream inputs: yes 
- File inputs: yes. 
- Compound nodes: yes.
- Downstream shader: gltf PBR

```mermaid
graph TB
    gltf_pbr_surfaceshader[gltf_pbr:surfaceshader]
    surfacematerial([surfacematerial:material])
    style surfacematerial   fill:#090, color:#FFF
    surfacematerial1([surfacematerial:material])
    style surfacematerial1   fill:#090, color:#FFF
    gltf_pbr_surfaceshader1[gltf_pbr:surfaceshader]
    subgraph nodegraph1
    nodegraph1_output_color4([output:color3])
    style nodegraph1_output_color4  fill:#09D, color:#FFF
    nodegraph1_output_color3([output:color3])
    style nodegraph1_output_color3  fill:#09D, color:#FFF
    nodegraph1_texcoord_vector2[texcoord:vector2:0]
    nodegraph1_place2d_vector3[place2d:vector2]
    nodegraph1_place2d_vector2[place2d:vector2]
    nodegraph1_image_color4[image:color3]
    nodegraph1_image_color3[image:color3]
    end
    nodegraph1_output_color3 --"base_color"--> gltf_pbr_surfaceshader
    gltf_pbr_surfaceshader --"surfaceshader"--> surfacematerial
    gltf_pbr_surfaceshader1 --"surfaceshader"--> surfacematerial1
    nodegraph1_output_color4 --"base_color"--> gltf_pbr_surfaceshader1
    nodegraph1_texcoord_vector2 --"texcoord"--> nodegraph1_place2d_vector3
    nodegraph1_texcoord_vector2 --"texcoord"--> nodegraph1_place2d_vector2
    nodegraph1_place2d_vector3 --"texcoord"--> nodegraph1_image_color4
    nodegraph1_image_color4 --> nodegraph1_output_color4
    nodegraph1_image_color3 --> nodegraph1_output_color3
    nodegraph1_place2d_vector2 --"texcoord"--> nodegraph1_image_color3
```

</td>
<td><a href="./tests/data/bindings/gltf_shared_filetexture_2.mtlx">MTLX</a>
<a href="./tests/data/bindings/gltf_shared_filetexture_2.gltf">GLTF</a>
<a href="./tests/data/bindings/gltf_shared_filetexture_2.usda">USD</a>
<td><img src="./tests/data/bindings/gltf_shared_filetexture_2.png">
</tr>

<tr>
<td>Example MaterialX version of "boombox" example (from Khronos sample assets) that shows file name resolving.

- Graph count: single
- Graph inputs: none
- Graph outputs: one
- File inputs: yes. 
- File prefix: resolved during conversion (as is done for "UsdMtlx" resolve)
- Compound nodes: no
- Downstream shader: glTF PBR

```mermaid
graph LR
    subgraph boombox_graph
    boombox_graph_out_Image([out_Image])
    style boombox_graph_out_Image  fill:#09D, color:#FFF
    boombox_graph_texcoord1[texcoord1:0]
    boombox_graph_Image[Image]
    end
    SR_boombox[SR_boombox]
    Material_boombox([Material_boombox])
    style Material_boombox   fill:#090, color:#FFF
    boombox_graph_texcoord1 --"texcoord"--> boombox_graph_Image
    boombox_graph_Image --> boombox_graph_out_Image
    boombox_graph_out_Image --"base_color"--> SR_boombox
    SR_boombox --"surfaceshader"--> Material_boombox
```

</td>
<td><a href="./tests/data/gltf_examples/gltf_pbr_boombox.mtlx">MTLX</a>
<a href="./tests/data/gltf_examples/gltf_pbr_boombox.gltf">GLTF</a>
<a href="./tests/data/gltf_examples/gltf_pbr_boombox.usda">USD</a>
</td>
<td><img src="./tests/data/gltf_examples/gltf_pbr_boombox.png">
</td>
</tr>

<tr>
<td>Example with various port data types: integer, vec2, vec3, vec4, color3, color4, integer, matrix33, matrix44

- Graph count: single
- Graph inputs: none
- Graph outputs: 1 per type
- File inputs: no
- Compound nodes: no
- Downstream shader: glTF PBR

```mermaid
graph LR
    glTF_Material([surfacematerial:material])
    style glTF_Material   fill:#090, color:#FFF
    glTF_Shader[gltf_pbr:surfaceshader]
    subgraph mygraph
    mygraph_out([output:color3])
    style mygraph_out  fill:#09D, color:#FFF
    mygraph_out1([output:color3])
    style mygraph_out1  fill:#09D, color:#FFF
    mygraph_out2([output:color3])
    style mygraph_out2  fill:#09D, color:#FFF
    mygraph_out3([output:color3])
    style mygraph_out3  fill:#09D, color:#FFF
    mygraph_out4([output:color3])
    style mygraph_out4  fill:#09D, color:#FFF
    mygraph_out6([output:color3])
    style mygraph_out6  fill:#09D, color:#FFF
    mygraph_out7([output:color3])
    style mygraph_out7  fill:#09D, color:#FFF
    mygraph_out5([output:color3])
    style mygraph_out5  fill:#09D, color:#FFF
    mygraph_out8([output:color3])
    style mygraph_out8  fill:#09D, color:#FFF
    mygraph_convert_color4[convert:color3]
    mygraph_convert_color5[convert:color3]
    mygraph_convert_color6[convert:color3]
    mygraph_convert_color7[convert:color3]
    mygraph_determinant_float1[determinant:float]
    mygraph_convert_color8[convert:color3]
    mygraph_convert_color9[convert:color3]
    mygraph_determinant_float2[determinant:float]
    mygraph_convert_color10[convert:color3]
    mygraph_mix_color3[mix:color3]
    mygraph_mix_float1[mix:float]
    mygraph_mix_color5[mix:color4]
    mygraph_mix_vector2[mix:vector2]
    mygraph_multiply_matrix34[multiply:matrix33]
    mygraph_multiply_matrix45[multiply:matrix44]
    mygraph_constant_integer1([constant:integer:1])
    style mygraph_constant_integer1  fill:#888, color:#000
    mygraph_mix_vector3[mix:vector3]
    mygraph_convert_color11[convert:color3]
    mygraph_mix_vector5[mix:vector4]
    end
    glTF_Shader --"surfaceshader"--> glTF_Material
    mygraph_out --"base_color"--> glTF_Shader
    mygraph_mix_color3 --> mygraph_out
    mygraph_mix_float1 --"in"--> mygraph_convert_color4
    mygraph_convert_color4 --> mygraph_out1
    mygraph_mix_color5 --"in"--> mygraph_convert_color5
    mygraph_convert_color5 --> mygraph_out2
    mygraph_mix_vector3 --"in"--> mygraph_convert_color6
    mygraph_convert_color6 --> mygraph_out3
    mygraph_mix_vector2 --"in"--> mygraph_convert_color7
    mygraph_multiply_matrix34 --"in"--> mygraph_determinant_float1
    mygraph_convert_color7 --> mygraph_out4
    mygraph_determinant_float1 --"in"--> mygraph_convert_color8
    mygraph_determinant_float2 --"in"--> mygraph_convert_color9
    mygraph_multiply_matrix45 --"in"--> mygraph_determinant_float2
    mygraph_convert_color9 --> mygraph_out6
    mygraph_convert_color10 --> mygraph_out7
    mygraph_constant_integer1 --"in"--> mygraph_convert_color10
    mygraph_convert_color8 --> mygraph_out5
    mygraph_convert_color11 --> mygraph_out8
    mygraph_mix_vector5 --"in"--> mygraph_convert_color11
```

</td>
<td><a href="./tests/data/supported_types.mtlx">MTLX</a>
<a href="./tests/data/supported_types.gltf">GLTF</a>
<!-- <a href="./tests/data/supported_types.usda">USD</a> -->
</td>
<td><img src="./tests/data/supported_types.png">
</td>
</tr>

</table>

</details>

### Development Information

<details><summary><h4>Reference Rendering</h4></summary>

All reference images are rendered using the `MaterialXView` sample application which is available as part of  
<a href="https://github.com/AcademySoftwareFoundation/MaterialX/releases">MaterialX releases</a>. The release
version used matches the version requirement for this package.

A sample utility called `test_render` is provided which will scan all files MaterialX XML files in a given folder hierarchy and use the path to `MaterialXView` to
render into the same folder.

Example:

```
python utilities/test_render.py tests/data -r 512 -c <path to MaterialXView>
```
where `<path to MaterialXView>` is the path to the MaterialXView executable with
a the resolution set to 512 by 512.

</details>

<details><summary><h4>Build Scripts</h4></summary>

Sample build scripts are provided in the `utilities` folder as follows:

1. `build.sh` : Will pull from head of the repo, install dependencies, and build the package.
2. `build_docs.sh` : Will build only the documentation. Called from `build.sh`. Doxygen is assumed to be installed. The README files are generated from the template Markdown file: `utilitites/README_template.md.` This will install the top level as well as the API documentation versions of this file with appropriate formatting to support the `Mermaid` graphs used to node graph diagrams.
3. `build_tests.sh` : Will run unit tests as well as command line tests.

`build.sh` is called within the check-in workflow defined in `.github/workflows/main.yml`

</details>

