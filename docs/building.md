# Building and Dependencies

This information is for how to build your own copy of the SDK binaries. Most users of 
the Azure Kinect DK should not need to re-build the SDK, if you just need to use the SDK
see [usage](usage.md) for more information. If you need to build from source, you have to clone the repository instead of downloading the Zip file to ensure all the submodule dependencies in place.

## Support Configurations

Compilers:
* MSVC (Windows)
* Clang (Linux)
* GCC (Linux)

Architectures: 
* amd64
* x86 (limited, no testing is performed)

Toolchain files in [cmake/toolchains](../cmake/toolchains) are mainly used for our
CI system but can be used to test specific flavors if needed.

## Dependencies

Building on Windows and Linux requires an active internet connection. Part of the
build will require downloading dependencies.

### Windows Dependencies

The following tools are required to build on Windows:

* [VS2017](https://visualstudio.microsoft.com/vs/). Contains the toolchains
  for building. Please be sure to install the following workloads:
  * Desktop development with C++
  * Linux development with C++ (Visual C++ tools for CMake and Linux)

  Also make sure the following components are selected:
  * Visual C++ Tools for CMake
  * Test adapter for Google Test

* [CMake](https://cmake.org/download/). Add CMake to the PATH. Must be
  version 3.10 or higher. Only necessary if your version of VS2017 is less than
   15.3 or you have not installed the "Linux development with C++" workload as 
   part of your VS installation.

* [ninja](https://github.com/ninja-build/ninja/releases). Add ninja to the
  PATH. Only necessary if your version of VS2017 is less than 15.3 or you have
  not installed the "Linux development with C++" workload as part of your VS
  installation.

* [tar](http://gnuwin32.sourceforge.net/packages/gtar.htm). Note, if you are
  running Windows 10 version 17063 or later, this comes installed by default.

* [python3](https://www.python.org/getit/). During the install make sure to add
  python to path.

* [Depth Engine](depthengine.md). The depth engine (DE) is a closed source binary shipped with the
  SDK installer. The DE binary needs to be copied to the host system and added 
  to the path. **NOTE:** *This step is not need for building, but is required 
  to run the SDK and tests*

The following tools are optional:

* [Doxygen](http://www.doxygen.nl/download.html). Add doxygen to the PATH.
  Required for building documentation. To use, pass the CMake parameter `-DK4A_BUILD_DOCS=1`

* [Clang-Format](http://releases.llvm.org/download.html). Please download clang
  v6.0.0 since that is what we are using to format our code. To invoke, call `ninja clangformat`

If you are building from a command prompt, it **must** be a **x64 Visual Studio
developer command prompt** in order for CMake to find the installed compilers.
We build both 32-bit and 64-bit binaries, but 64-bit binaries are the only
binaries that are tested. (The command prompt should be called something like
x64 Native Tools Command Prompt for VS 2017). Note: call the command line tool
with the option `-arch=amd64` for x64 builds i.e `VsDevCmd.bat
-arch=amd64`

**NOTE:** *You can run
[scripts/verify-windows.ps1](../scripts/verify-windows.ps1) to verify that your
Windows PC is setup properly.*

### Linux Dependencies

The list of libraries and tools that must be installed is maintained in a Dockerfile
used by our CI system. Extract and run the install list from the 
[Dockerfile](../scripts/Dockerfile) to ensure your machine has required dependencies.

The depth engine is needed as well. The depth engine (DE) is a closed source
binary shipped with the Linux Debian package. As an example, run `apt install
libk4a1.3` to install the Azure Kinect 1.3 and get the depth engine. See 
[using the depth engine](usage.md#debian-package) for information about
versioning and adding the Microsoft's Package Repository to your machine. **NOTE** *This step is not
need for building, but is required running the SDK*


## Building

### Building using a terminal (cross platform)

1. Create a folder named "build" in the root of the git repo and cd into that
    directory.

    ```
    mkdir build && cd build
    ```

2. Run CMake from that directory. The preferred build is ninja. All other
    generators are untested.
    
    Release Build:
    ```
    cmake .. -GNinja
    ```
    Debug Build:
    ```
    cmake .. -GNinja -DCMAKE_BUILD_TYPE=Debug
    ```

3. Run the build (ninja).

    ```
    ninja
    ```

### Building using Visual Studio

Visual Studio 2017 supports opening CMake based projects directly.
Use File / Open / CMake ... to open the root CMakeLists.txt in the project.

To cross compile for Linux on Windows you can run a pre-configured
[docker container](../docker/DOCKER.md) with the tools needed for Visual
Studio.

### C# Wrapper

The C# Wrapper is not included in the primary CMake build systems, it is built using the standard
Visual Studio MSBuild build system. The C# build depends on files generated by the CMake and Ninja
build systems. Therefore the CMake configuration steps generates files that will be consumed by
the C# Build and Test systems to find the needed components. Therefore, you need to run the CMake
configuration and build before loading the C# solution.

1. Build the project with the configuration for the k4a.dll you want to build against. (e.g. x64 RelWithDebInfo)
2. Open the [K4a.sln](../src/csharp/K4a.sln) solution
3. Build C# from within Visual Studio

**NOTE:** Running a configure step again, such as opening the folder in Visual Studio, will overwrite the settings and can break the C# build.
