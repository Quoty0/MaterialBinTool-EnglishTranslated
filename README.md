#MaterialBinTool
RenderDragon .material.bin file unpack/pack/compile tool

## use
1. Install Java 8 or higher
2. Unpack: `java -jar MaterialBinTool-0.5.1-all.jar -u material.bin file path`
3. Packaging: `java -jar MaterialBinTool-0.5.1-all.jar -r Unpack the output directory or path to the json file`
4. Compile: `java -jar MaterialBinTool-0.5.1-all.jar -c input directory or path to json file`

## command line arguments
````
java -jar MaterialBinTool-0.5.1-all.jar [options] <input file or directory>
All options:
  -u, --unpack unpack the input .material.bin file or all .material.bin files in the input directory
  -a, --add-flagmodes Add the Variant's FlagMode as a comment to the output shader file (only available on ESSL and GLSL platforms)
  -r, --repack pack the input directory or json file into a .material.bin file
  --raw output/input raw bgfx shader files instead of just shader code
  -c, --compile compile input directory or json file into .material.bin file
  -h, --help see help
  -s, --shaderc specifies the shaderc executable path (if not specified or the specified file does not exist/not executable, try to find it from the PATH environment variable)
  -i, --include specify additional include directories for shader compilation
  -o, --output specify the output directory (if not specified, the unpacking defaults to the same level directory of .material.bin, and the default output for packaging/compiling is to the input directory or the same level directory of the input json file)
````

## compile sc file
Currently supported platforms: ESSL(Android), Direct3D(Win10), Metal(iOS)
1. Execute `java -jar MaterialBinTool-0.5.1-all.jar -u material.bin file path` to unpack the .material.bin file to be compiled
2. Create a src directory in the unpacked output directory
3. Place the shader source file in the src directory. The vertex shader is named `filename.vertex.sc`, the fragment shader is named `filename.fragment.sc`, and the varyingDef file is named `filename.varying` .def`
4. (Optional) Create defines.json in the unpacked output directory and add macro definition rules
5. (Optional) Add the directory where shaderc.exe is located to the PATH environment variable
6. Execute `java -jar MaterialBinTool-0.5.1-all.jar -c unpack output directory` to start compiling

## Default macro definition rules
When compiling, some macro definitions will be automatically generated according to the Pass name and the FlagMode of the Variant. The default macro definition addition and naming rules are as follows:
1. Convert the current Pass name to uppercase and underscore (example: `DepthOnlyOpaque -> DEPTH_ONLY_OPAQUE`, `Transparent -> TRANSPARENT`)
2. The key name of FlagMode whose value is On is converted to uppercase and underscore form (for example: `RenderAsBillboards=On -> RENDER_AS_BILLBOARDS`, `Seasons=On -> SEASONS`, `Seasons=Off -> None`)

## defines.json
Some FlagMode values ​​are not limited to On and Off. In this case, macro definition rules can be added to these FlagModes by creating defines.json in the unpacked output directory.
The format of defines.json is as follows:
````json
{
    "passes": {
        "Pass Name 1": ["Macro Name 1", "Macro Name 2"],
        "PassName2": ["MacroName1", "MacroName2", "MacroName3"]
    },
    "flagModes": {
        "keyname1": {
            "value1": ["macroname1", "macroname2"],
            "value2": ["macroname1", "macroname2", "macroname3"]
        },
        "keyname2": {
            "value1": ["macroname1", "macroname2"],
            "value2": ["macroname1", "macroname2", "macroname3"],
            "value3": ["macroname1"]
        }
    }
}
````
Note: Pass and FlagMode declared in defines.json will no longer define macros according to the default macro definition rules

## compile example
Take RenderChunk.material.bin compiled 1.18.31 as an example:
1. Execute `java -jar MaterialBinTool-0.5.1-all.jar -u RenderChunk.material.bin file path`
The directory structure after execution should look like this:
````
RenderChunk/
    AlphaTest/
        0~5. Platform name. Shader type. Shader language
        AlphaTest.json
    DepthOnly/
        0~5. Platform name. Shader type. Shader language
        DepthOnly.json
    DepthOnlyOpaque/
        0~5. Platform name. Shader type. Shader language
        DepthOnlyOpaque.json
    Opaque/
        0~5. Platform name. Shader type. Shader language
        Opaque.json
    Transparent/
        0~5. Platform name. Shader type. Shader language
        Transparent.json
    RenderChunk.json
````
2. Create a `src` directory in the `RenderChunk` directory, and place the header files such as `RenderChunk.vertex.sc`, `RenderChunk.fragment.sc`, `RenderChunk.varying.def.sc` and `bgfx_shader.sh` (optional)
3. Because the values ​​of FlagMode in RenderChunk.material.bin are only On/Off, there is no need to create `defines.json` to add additional macro definition rules (other files are created as appropriate)
The directory structure should now be:
````
RenderChunk/
    AlphaTest/
        0~5. Platform name. Shader type. Shader language
        AlphaTest.json
    DepthOnly/
        0~5. Platform name. Shader type. Shader language
        DepthOnly.json
    DepthOnlyOpaque/
        0~5. Platform name. Shader type. Shader language
        DepthOnlyOpaque.json
    Opaque/
        0~5. Platform name. Shader type. Shader language
        Opaque.json
    Transparent/
        0~5. Platform name. Shader type. Shader language
        Transparent.json
    src/
        bgfx_shader.sc (optional)
        RenderChunk.vertex.sc
        RenderChunk.fragment.sc
        RenderChunk.varying.def.sc
    RenderChunk.json
````
4. Execute `java -jar MaterialBinTool-0.5.1-all.jar -s shaderc.exe path (optional) -i directory path containing bgfx_shader.sh (optional) -c RenderChunk directory path` to start compiling
If the directory where shaderc.exe is located has been added to the PATH environment variable, you can not specify the `-s` parameter
If you have copied bgfx_shader.sh to the src directory, you can not specify the `-i` parameter
5. After the execution is completed, the compiled `RenderChunk.material.bin` will be generated in the `RenderChunk` directory, which can be used by replacing the corresponding files in the installation package
   
Note: At present, the compiled files are still not universal for all platforms. The unpacked files can only be used on which platform after the compilation is completed.

## About sc language
sc is bgfx's GLSL-based cross-platform shader language, which can be compiled into shaders for various platforms through `shaderc`
Most of the syntax of sc is the same as GLSL, but there are some differences. You need to follow the bgfx standard when writing. The specific differences can be viewed in the bgfx documentation: [shader-compiler-shaderc](https://bkaradzic.github.io/bgfx/tools.html#shader-compiler-shaderc)

## sc source file acquisition
Some sc source files that have been sorted out by [OEOTYAN](https://github.com/OEOTYAN/) can be obtained in the [RenderDragonSorceCodeInv](https://github.com/OEOTYAN/RenderDragonSorceCodeInv) repository
Other source files (except RTX related) can be manually sorted out according to the glsl unpacked from the Android version of .material.bin
`bgfx_shader.sh` can be obtained in the bgfx repository (the version of bgfx used by Rendering Dragon is older, and the source code in the latest version of the repository is not guaranteed to be available. Here is an available version: [bgfx_shader.sh](https://github.com/bkaradzic/bgfx/blob/1ba107d156d1d28e86550df5d586ea259aec1020/src/bgfx_shader.sh))
