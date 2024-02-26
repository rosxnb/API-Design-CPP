# INTRODUCTION

A library lets you package the compiled code and data that implement your API.
The characteristics, usage, and supporting tools for libraries are inherently
platform specific. You need to handle a Dynamic Link Library (DLL) on Windows
differently than how you handle Dynamic Shared Object (DSO) on Unix.

**Contents:**

- [Static Vs Dynamic Libraries](#static-vs-dynamic-libraries)
- [Dynamic Libraries As Plugin](#dynamic-libraries-as-plugins)
- [Libraries On Windows](#libraries-on-windows)
    - [Importing and Exporting Functions](#importing-and-exporting-functions)
    - [The DLL Entry Point](#the-dll-entry-point)
    - [Loading Plugins on Windows](#loading-plugins-on-windows)
- [Libraries On Linux](#libraries-on-linux)
    - [Creating Static Libraries on Linux](#creating-static-libraries-on-linux)
    - [Creating Dynamic Libraries on Linux](#creating-dynamic-libraries-on-linux)
    - [Shared Library Entry Points](#shared-library-entry-points)
    - [Useful Linux Utilities](#useful-linux-utilities)
    - [Loading Plugins on Linux](#loading-plugins-on-linux)
    - [Finding Dynamic Libraries at Run Time](#finding-dynamic-libraries-at-run-time)
- [Libraries On MacOS](#libraries-on-macos)
    - [Creating Static Libraries on MacOS](#creating-static-libraries-on-macos)
    - [Creating Dynamic Libraries on MacOS](#creating-dynamic-libraries-on-macos)
    - [Useful MacOS Utilities](#useful-macos-utilities)
    - [Frameworks on MacOS](#frameworks-on-macos)
    - [Finding Dynamic Libraries at Run Time](#finding-dynamic-libraries-at-run-time-1)


## Static Vs Dynamic Libraries

Static and Shared (Dynamic) libraries are two main forms of libraries you can create. The decision on which one you
employ can have a significant impact on your clients' application in terms of tangible
factors such as *load time*, *executable size*, and *robustness* to different versions of you API.

> Distribute static library is library is sufficiently small and stable.
> Distribute dynamic library to give users greater flexibility.


<table>
    <tr>
        <th>Static Library</th>
        <th>Shared Library</th>
    </tr>
    <tr>
        <td>It contains object code that becomes part of end-user application.</td>
        <td>The shared object doesn't become part of end-user app.</td>
    </tr>
    <tr>
        <td>Only needed at link time and not on run time because library code is embedded inside the application.</td>
        <td>It must be distributed with app so it can be discovered at runtime of application.</td>
    </tr>
    <tr>
        <td>Clients can distribute their app without any additional run-time dependencies.</td>
        <td>Client app will not run if shared object is deleted, moved, or upgraded, or downgraded.</td>
    </tr>
    <tr>
        <td>Multiple executables need separate copy of static library which results in total size increase.</td>
        <td>
        It is efficient in disk space utilization when more that one app needs to use the library because library 
        code is stored in a single shared folder.
        </td>
    </tr>
    <tr>
        <td>No concerns of finding an incompatible library version on the end-user's machine.</td>
        <td>
        It is efficient in memory because most modern OS will attempt to only load the shared object code
        into memory once and share it across all app that depend on it.
        </td>
    </tr>
    <tr>
        <td>
        These are represented as a collection of object files that can be copied individually into an executable 
        as needed.
        </td>
        <td>
        Here all code is essentially flattened into a single object file. So loading a DLL involves loading all the 
        code defined in there.
        </td>
    </tr>
    <tr>
        <td>
        To update version of library in end-user app, they must replace entire executable to achieve this. This 
        can lead to long wait if update is internet-based.
        </td>
        <td>
        To update version of shared library, user can simply drop in the replacement library and all of apps that 
        depend will use new version without having to recompile and relink.
        </td>
    </tr>
</table>

![Static library linking](../pics/Staitc%20library%20linking.png "linking a static library into an application causes the library code to be embedded in the resulting executable")
*fig: Static library linking*


![Dynamic library linking](../pics/Dynamic%20library%20linking.png "A dynamic library is used to link an application and is then distributed with that application so that the library can be loaded at run time")
*fig: Dynamic library linking*


## Dynamic Libraries As Plugins

It is possible for an application to load a dynamic library on demand without the application having been compiled and
linked against that library. This gives you the capability to create extensible APIs that allow your clients to drop in
new functionality that your API will then load and execute. The Netscape Plugin API is an example of this: it is the 
API that you develop against in order to create a plugin (i.e., dynamic library) that browsers such as Firefox, Safari,
Opera, and Chrome can then load and run. I discussed the use of plugins to create extensible APIs in Chapter 12.

![Plugin library](../pics/plugin%20library.png)
*fig: A plugin library is a dynamic library that can be compiled separately from the application and explicitly loaded by the application on demand*


## Libraries On Windows

- static libraries are represented with *.lib* files
- dynamic libraries are represented with *.dll* files

The *.dll* file **must be** accompanied with as **import library**, or a *.lib* file.
The import library is used to resolve references to symbols exported in DLL.

While static library and import library share same extension, the are **different files**.
If you plan to distribute both static and dynamic library versions of your API, 
then make sure to **avoid filename collision** either naming the static library differetly
or placing each in a separate directory.

---------------------------------------------------------------------------------
| **static:** foo_static.lib           | **static:** static/foo.lib             |
| **dynamic:** foo.dll                 | **dynamic:** dynamic/foo.dll           |
| **import:** foo.lib                  | **import:** dynamic/foo.lib            |
---------------------------------------------------------------------------------

### Importing and Exporting Functions

To make a function/class callable from a DLL on Windows, you must explicitly mark its declaration as following:
```C++
// to export
__declspec(dllexport) void MyFunction();
class __declspec(dllexport) MyClass;
```

To use an exported DLL function in an application, you must prefix the function prototype with `__declspec(dllimport)`.

It's common to employ preprocessor macros to use the export declaration when building an API but the import decoration
when using the same API in an application. It's also important because `__declspec` decorations may cause compile
errors on non-Window compilers. The following code provides a simple demonstration of this technique:

```C++
#ifdef _WIN32
    #ifdef _EXPORTING
        #define DECLSPEC __declspec(dllexport)
    #else
        #define DECLSPEC __declspec(dllimport)
    #endif
#else
    #define DECLSPEC
#endif
```

**Alternatively**, you can create a module definition *.def* file to specify the symbols to export.
A minimal DEF file contains a `LIBRARY` statement to specify the name of the DLL the file is associated with, 
and an `EXPORTS` statement followed by list of names to export.
The DEF file syntax also supports more powerful manipulations of your symbols, such as renaming symbols
or using an ordinal number as the export name to help minimize the size of the DLL.

```DEF
//MyLIB.def

LIBRARY "MyLIB"
EPORTS
    MyFunction1
    MyFunction2
    MyClass
```

### The DLL Entry Point

DLLs can provide an **optional entry point function** to initialize data structures when a thread or process
loads the DLL or to clean up memory when the DLL is unloaded. This is managed by a function called `DllMain()`
that you define and export within the DLL. If the entry point function returns FALSE, this is assumed to be a
fatal error and the application will fail to start.

```C++
BOOL APIENTRY DllMain(
    HANDLE dllHandle,
    DWORD reason,
    LPVOID lpReserved
) {
    switch (reason) {
        case DLL_PROCESS_ATTACHED:
            // A process is loading the DLL
            break;
        case DLL_PROCESS_DETACH:
            // A process unloads the DLL
            break;
        case DLL_THREAD_ATTACHED:
            // A process is creating a new thread
            break;
        case DLL_THREAD_DETACH:
            // A thread exits normally
            break;
    }

    return true;
}
```

### Loading Plugins on Windows

On Windows, `LoadLibrary()` or `LoadLibraryEx()` functions can be used to load a dynamic library into a process.
`GetProcAddress()` is used to obtain the address of an exported symbol in the DLL.
You don't need to an import library *.lib* file in order to load a dynamic library in this way.
For instance, consider following simple plugin interface used to create a *plugin.dll* library:

```C++
#ifndef PLUGIN_H
#define PLUGIN_H

#include <string>

extern "C"
__declspec(dllexport) void DoSomething(const std::string& str);

#endif // PLUGIN_H
```

To load this DLL on demand and call the `DoSomething()` method from that library:

```C++
// open DLL
HINSTANCE handle = LoadLibrary("plugin.dll");
if (!handle)
{
    std::cerr << "Cannot load plugin!" << std::endl;
    exit(1);
}

// get the DoSomething() functin from the plugin
FARPROC fptr = GetProcAddress(handle, "DoSomething");
if (fptr == (FARPROC)NULL)
{
    std::cerr << "Cannot find the function in plugin: " << std::endl;
    FreeLibrary(handle);
    exit(1);
}

// call the DLL function
(*fptr)("Hello There!");

// close the shared library
FreeLibrary(handle);
```


## Libraries On Linux

For deeper understanding on the matter, refer to [this paper](https://www.akkadia.org/drepper/dsohowto.pdf)


### Creating Static Libraries on Linux

On Linux, a static library is simply the archive of object *.o* files.

```bash
g++ -c file1.cpp
g++ -c file2.cpp

ar -crs libmyapi.a file1.o file2.o
```

Users can link against you library using `-l` option. `-L` linker option can be used to specify the directory
where the library can be found.
```bash
g++ main.cpp -L. -lmyapi
```

The `-static` compiler option is to be used by API consumer and not by the API developer. This flag instructs
the compiler to prefer linking the static versions of all dependent libraries into the executable so that it
depends on no dynamic libraries at run time.


### Creating Dynamic Libraries on Linux

It follows similar process to creating a static library. Couple distinct options used are
- `-shared`: to generate a *.so* file instead of an executable during API development.
- `-fpic`:  to instruct the compiler to emit position-independent code (PIC) because the code in a shared library may be loaded into a different memory location for different executables. This ensures that the user code doesn't depend on the absolute memory address of symbols.

```bash
g++ -c -fPIC file1.cpp
g++ -c -fPIC file2.cpp

g++ -shared -o libmyapi.so -fPIC file1.o file2.o
```

By default, all symbols in a DSO are exported publicly unless specified by the `-fvisibility=hidden` compiler option.
However, the GNU C++ compiler supports the concept of export maps, with `--version-script=<filename>` option, to define
explicitly the set of symbols in a dynamic library that will be visible to client programs.

Following *export.map* file specifies all symbols to be hidden except the `DoSomething()` function.
```txt
{
    global: DoSomething;
    local: *
};
```
```bash
g++ -shared -o libmyapi.so -fPIC file1.o file2.o -Wl,--version-script=export.map
```


### Shared Library Entry Points
We can define static constructors and destructors for shared library loading and unloading operations like library
initialization and cleanup without requiring your users to call explicit functions to perform this.
This works with all compilers and platforms.
Caveats:
- order of initialization of static constructors is not defined across translation units.
- meaning, don't depend on static variables in other *.cpp* files being initialized.

You can create a shared library entry point in one of you *.cpp* files as follows:
```C++
class APIInitManager
{
    public:
        APIInitManager()
        {
            std::cout << "APIInitManager initialize." << std::endl;
        }
        ~APIInitManager()
        {
            std::cout << "APIInitManager destroyed." << std::endl;
        }
};

static APIInitManager sInitManager;
```

In GNU compiler, we can use `__attribute__((constructor))` and `__attribute__((destructor))` decorations for
functions to achieve same behaviour. If this approach is used, shared library must not be compiled with the 
GNU GCC arguments `-nostartfiles` or `-nostdlib`.

```C++
static void __attribute__((constructor)) APIInitialize()
{
    std::cout << "API initialized." << std::endl;
}

static void __attribute__((destructor)) APICleanUp()
{
    std::cout << "API cleaned up." << std::endl;
}
```


### Useful Linux Utilities

The **libtool**:
Provides a consistent and portable interface for creating libraries on different UNIX platforms.
It is mostly used to create a static or dynamic library by passing list of object files with respective
`-static` or `-dynamic` option.
```bash
libtool -static -o libmyapi.a file1.o file2.o
```


The **nm**:
Used to display symbol names in an object file or library. This is useful to find out if a library defines
or uses a given symbol.

```bash
nm -g libmyapi.a
```
Above command shows all of the global(external) symbols in myapi static library.
**T** in second column of `nm` command result represents symbol defined in this library,
**U** represents symbol defined in other library and referenced in this library.
Third column represents the mangled symbol name.


The **c++filt**:
This can be used to unmangle the mangled name produced by C++ compiler.
```bash
c++filt __ZNSt8ios_base4InitD1Ev
std::ios_base::Init::~Init()
```


The **ldd**:
Used to display the list of dynamic libraries that an executable depends on.
This will display the full path that will be used for each library showing also the
version of a dynamic library and whether any dynamic libraries cannot be found by OS.


### Loading Plugins on Linux

`dlopen()` to load *.so* file into the current process.
`dlsym()` to access symbols within that library.

```C++
typedef void(*FuncPtrT)(const std::string&);
const char *error;

// open the dynamic library
void* handle = dlopen("libplugin.so", RTLD_LOCAL | RTLD_LAZY);
if (!handle)
{
    std::cerr << "Cannot load plugin!" << std::endl;
    exit(1);
}
dlerror();

// get the DoSomething() function
FuncPtrT fptr = (FuncPtrT) dlsym(handle, "DoSomething");
if ((error == dlerror()))
{
    std::cerr << "Cannot find function in plugin: " << error << std::endl;
    dlclose(handle);
    exit(1);
}

(*fptr)("Hello There!");
dlclose(handle);
```


### Finding Dynamic Libraries at Run Time

1. Standard library directories on end user machine, for example */usr/lib*.
2. `LD_LIBRARY_PATH` environment variable can be set to augment default library search path with colon-separated list.
3. `-rpath` linker option to burn preferred path to search for DLL into client executable.
`g++ usercode.cpp -o userapp -L. -lmyapi -Wl,-rpath,/usr/local/lib`


## Libraries On MacOS

MacOS is built on a version of BSD UNIX called Darwin. As such, many of the details for Linux apply equally well
to the Mac. However, there are a few differences.


### Creating Static Libraries on MacOS

Apple discourages use of `-static` compiler option to generate executables with all library dependencies
linked statically. This is because Apple wants to ensure that applications always pull in the latest system
libraries that they distribute.

Mac linker will scan through all paths looking or a dynamic library then only static library.
There is no way to favor linking against a static library over a dynamic library on Mac when both are located
in the same directory, not even with `-static` flag.


### Creating Dynamic Libraries on MacOS

Similar to Linux with the difference that you use flag `-dynamiclib` instead of `-shared`.
```bash
clang++ -dynamiclib -o libmyapi.so -fPIC file1.o file2.o \
    -headerpad_max_install_names
```


### Useful MacOS Utilities

Almost all utilities on Linux are also availabel on MacOS.

The **otool**:
    This is similar to **ldd** tool on Linux. It lists the collection of dynamic libraries used by an executable.
```bash
otool -L userapp
```

### Frameworks on MacOS

Frameworks are Apple's way to distribute all the files necessary to compile and link against an API
in a single package.
A framework is simply a directory with a *.framework* extension that can contain various resources such
as dynamic libraries, header files, and reference documentation.
Also, a framework can contain multiple versions of a library in the same bundle to make it easier to 
maintain backward compatibility for older applications.

The following directory listing gives an example layout for a framework bundle, where the -> symbol 
represents a symbol link.

```framework
MyAPI.framework/
    Headers -> Versions/Current/Headers
    MyAPI -> Versions/Current/MyAPI
    Versions/
        A/
            Headers/
                MyHeader.hpp
            MyAPI
        B/
            Headers/
                MyHeader.hpp
            MyAPI
        Current -> B
```

Apple distributes most of its APIs as frameworks, such as Cocoa, Foundation, and Core Services, 
located in the /Developer/SDKs directory. To make your API resemble a native Mac library, consider 
distributing it as a framework for the Mac platform. You can create your API as a framework using 
Apple's Xcode development environment. Simply select *File > New Project* and choose "Framework" in the 
left-hand panel. By default, you can opt to set up your project with either Carbon or Cocoa frameworks. 
If you're developing a pure C++ library and don't require these frameworks, you can remove them after 
Xcode creates the project for you.

Clients can link against your framework by supplying the `-framework` option to **g++** or **ld**.
They can also specify the `-F` option to specify the directory to find your framework bundle.


### Finding Dynamic Libraries at Run Time

Mac OS X does not support the Linux `-rpath` linker option. Instead, it provides the notion of install names.
An install name is a path that is burned into a Mach-O binary to specify the path to search for dependent 
dynamic libraries. This path can be specified relative to the executable program by starting the install name 
with the special string *@executable_path*.

Install name can be specified at building time of dynamic library, but the clients can change this path using
**install_name_tool** utility. if `-headerpad_max_install_names` option was not passed during build, the client
cannot specify a path that is longer that the original path in the *.dylib*.

Following commands demonstrate how a client could change the install name for your library
and change the install name for their executable:

```bash
install_name_tool -id@executable_path/../Libs/libmyapi.dylib \
    libmyapi.dylib

install_name_tool -change libmyapi.dylib \
    @executable_path/../Libs/libmyapi.dylib \
    UserApp.app/Contents/MacOS/executable
```
