# VKD3D-Proton

The VKD3D-Proton is a VKD3D fork designed to implement the entire Direct3D 12 API at the top of Vulcan.
The project serves as a development effort to support Direct3D 12 in [Proton] (https://github.com/ValveSoftware/Proton).

## Upstream

The original project is available at [WineHQ] (https://source.winehq.org/git/vkd3d.git/).

## Priorities

Efficiency and compatibility are important goals at the expense of compatibility with older drivers and systems.
Modern Vulcan extensions and features are being used aggressively to improve performance and compatibility.
For the best experience, it is recommended to use the latest drivers that you get your hands on.
Compatibility with the vkd3d standalone API is not the goal of this project.

------

## Cloning the repo

To clone a turnip, run:
```
git clone --recursive https://github.com/HansKristian-Work/vkd3d-proton
```
to pull all the submodules needed for construction.

## Building VKD3D

### Requirements:
- [wine](https://www.winehq.org/) (for `widl`) [for native builds]
  - On Windows this may be substituted for [Strawberry Perl](http://strawberryperl.com/) as it sends a `widl` and is easy to find and install - although this dependency can be fixed in the future.
- [Meson](http://mesonbuild.com/) build system (at least version 0.51)
- [glslang](https://github.com/KhronosGroup/glslang) translator
- [Mingw-w64](http://mingw-w64.org/) compiler, headers and tools (at least version 7.0) [for cross-builds for d3d12.dll which are default]

### Building:
#### The simple way
Within the VKD3D directory, run:
```
./package-release.sh master /your/target/directory --no-package
```

This will create a `vkd3d-master` folder in `/your/target/directory`, which contains 32-bit and 64-bit versions of VKD3D, which you can set up in the same way as the release versions as mentioned above.

To build the original (eg for `libvkd3d.so`), enter` --native` in the build script. With this option, it will be built using your system's compilers.

To save build directories for development, download `--dev-build` in the script. This option means "--no-package". After changing the source code, you can do the following to restore VKD3D:
```
# change to build.86 for 32-bit
cd /your/target/directory/build.64
ninja install
```

#### Compiling manually (cross for d3d12.dll, default)
```
# 64-bit build.
meson --cross-file build-win64.txt -Denable_standalone_d3d12=True --buildtype release --prefix /your/vkd3d/directory build.64
cd build.64
ninja install

# 32-bit build
meson --cross-file build-win32.txt -Denable_standalone_d3d12=True  --buildtype release --prefix /your/vkd3d/directory build.86
cd build.86
ninja install
```

#### Compiling manually (native)
```
# 64-bit build.
meson --buildtype release --prefix /your/vkd3d/directory build.64
cd build.64
ninja install

# 32-bit build
meson --cross-file x86-linux-gnu --buildtype release --prefix /your/vkd3d/directory build.86
cd build.86
ninja install
```

## Using VKD3D

VKD3D can be used by projects targeting Direct3D 12 as an alternative
at the time of construction with some modest source changes.

If VKD3D is available when building Wine, Wine will use it for support
Direct3D Applications 12.

## Environment variables

Most of the environment variables used by VKD3D are debugged. The
environment variables are not part of the API and can be changed or
removed in future versions of VKD3D.

Some debug variables are item lists. Elements must be separated by
commas or semicolons.

 - `VKD3D_CONFIG` - a list of options that change the behavior of libvkd3d.
    - vk_debug - enables Vulkan debug extensions and loads validation layer.
 - `VKD3D_DEBUG` - controls the debug level for log messages produced by
   libvkd3d. Accepts the following values: none, err, fixme, warn, trace.
 - `VKD3D_SHADER_DEBUG` - controls the debug level for log messages produced by
   libvkd3d-shader. See `VKD3D_DEBUG` for accepted values.
 - `VKD3D_LOG_FILE` - If set, redirects `VKD3D_DEBUG` logging output to a file instead.
 - `VKD3D_VULKAN_DEVICE` - a zero-based device index. Use to force the selected
   Vulkan device.
 - `VKD3D_DISABLE_EXTENSIONS` - a list of Vulkan extensions that libvkd3d should
   not use even if available.
 - `VKD3D_TEST_DEBUG` - enables additional debug messages in tests. Set to 0, 1
   or 2.
 - `VKD3D_TEST_FILTER` - a filter string. Only the tests whose names matches the
   filter string will be run, e.g. `VKD3D_TEST_FILTER=clear_render_target`.
   Useful for debugging or developing new tests.
 - `VKD3D_TEST_PLATFORM` - can be set to "wine", "windows" or "other". The test
   platform controls the behavior of todo(), todo_if(), bug_if() and broken()
   conditions in tests.
 - `VKD3D_TEST_BUG` - set to 0 to disable bug_if() conditions in tests.
 - `VKD3D_PROFILE_PATH` - If profiling is enabled in the build, a profiling block is
   emitted to `${VKD3D_PROFILE_PATH}.${pid}`.

## CPU profiling (development)

Pass `-Denable_profiling=true` to Meson to enable a profiled build. With a profiled build, use `VKD3D_PROFILE_PATH` environment variable.
The profiling dumps out a binary blob which can be analyzed with `programs/vkd3d-profile.py`.
The profile is a trivial system which records number of iterations and total ticks (ns) spent.
It is easy to instrument parts of code you are working on optimizing.

## Advanced shader debugging

These features are only meant to be used by vkd3d-proton developers. For any builtin RenderDoc related functionality
pass `-Denable_renderdoc=true` to Meson.

 - `VKD3D_SHADER_DUMP_PATH` - path where shader bytecode is dumped.
   Bytecode is dumped in format of `$hash.{spv,dxbc,dxil}`.
 - `VKD3D_SHADER_OVERRIDE` - path to where overridden shaders can be found.
   If application is creating a pipeline with `$hash` and `$VKD3D_SHADER_OVERRIDE/$hash.spv` exists,
   that SPIR-V file will be used instead.
 - `VKD3D_AUTO_CAPTURE_SHADER` - If this is set to a shader hash, and the RenderDoc layer is enabled,
 vkd3d-proton will automatically make a capture when a specific shader is encountered.
 - `VKD3D_AUTO_CAPTURE_COUNTS` - A comma-separated list of indices. This can be used to control which queue submissions to capture.
 E.g., use `VKD3D_AUTO_CAPTURE_COUNTS=0,4,10` to capture the 0th (first submission), 4th and 10th submissions which are candidates for capturing.

 If only `VKD3D_AUTO_CAPTURE_COUNTS` is set, any queue submission is considered for capturing.
 If only `VKD3D_AUTO_CAPTURE_SHADER` is set, `VKD3D_AUTO_CAPTURE_COUNTS` is considered to be equal to `"0"`, i.e. a capture is only
 made on first encounter with the target shader.
 If both are set, the capture counter is only incremented and considered when a submission contains the use of the target shader.

### Shader logging

It is possible to log the output of replaced shaders, essentially a custom shader printf. To enable this feature, `VK_KHR_buffer_device_address` must be supported.
First, use `VKD3D_SHADER_DEBUG_RING_SIZE_LOG2=28` for example to set up a 256 MiB ring buffer in host memory.
Since this buffer is allocated in host memory, feel free to make it as large as you want, as it does not consume VRAM.
A worker thread will read the data as it comes in and log it. There is potential here to emit more structured information later.
The main reason this is implemented instead of the validation layer printf system is run-time performance,
and avoids any possible accidental hiding of bugs by introducing validation layers which add locking, etc.
Using `debugPrintEXT` is also possible if that fits better with your debugging scenario.
With this shader replacement scheme, we're able to add shader logging as unintrusive as possible.

Replaced shaders will need to include `debug_channel.h` from `include/shader-debug`.
Use `glslc -I/path/to/vkd3d-proton/include/shader-debug --target-env=vulkan1.1` when compiling replaced shaders.

```
void DEBUG_CHANNEL_INIT(uvec3 ID);
```

is used somewhere in your replaced shader. This should be initialized with `gl_GlobalInvocationID` or similar.
This ID will show up in the log. For each subgroup which calls `DEBUG_CHANNEL_INIT`, an instance counter is generated.
This allows you to correlate several messages which all originate from the same instance counter, which is logged alongside the ID.
An invocation can be uniquely identified with the instance + `DEBUG_CHANNEL_INIT` id.
`DEBUG_CHANNEL_INIT` can be called from non-uniform control flow, as it does not use `barrier()` or similar constructs.
It can also be used in vertex and fragment shaders for this reason.

```
void DEBUG_CHANNEL_MSG();
void DEBUG_CHANNEL_MSG(uint v0);
void DEBUG_CHANNEL_MSG(uint v0, uint v1, ...); // Up to 4 components, can be expanded as needed up to 16.
void DEBUG_CHANNEL_MSG(int v0);
void DEBUG_CHANNEL_MSG(int v0, int v1, ...); // Up to 4 components, ...
void DEBUG_CHANNEL_MSG(float v0);
void DEBUG_CHANNEL_MSG(float v0, float v1, ...); // Up to 4 components, ...
```

These functions log, formatting is `#%x` for uint, `%d` for int and `%f` for float type.
