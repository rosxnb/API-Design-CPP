# INTRODUCTION

An API (Application Programming Interface) provides abstraction exposing the available functionalities and hiding implementations.
An API is a well defined interface that provides specific service to other pieces of software.
An API can be built upon an existing API essentially to extend the functionalities.

> An API describes software used by other engineers to build their applications.
> As such, it must be well-designed, documented, regression tested, and stable between releases.

**Contents:**
- APIs In C++
- Advantages Of API
- When To Avoid API
- APIs And SDKs
- File Formats And Network Protocols


## APIs In C++

In C++, API is: header files + supporting documentations files.
Implementation of API can be provided by:
- Static library:  *.lib* on Windows or *.a* on MacOS/Linux
- Dynamic library: *.dll* on Windows or *.dylib* on MacOS or *.so* on Linux

More about [Static vs Dynamic Library](./Staic_Dynamic_Library/README.md)


## Advantages of API

- **Hiding implementation** details in your module enables future flexibility in altering implementations without disrupting users, avoiding restrictions on updates or forcing users to rewrite their code to adopt new versions, thus influencing the success of your business or project by facilitating easier adoption of updates and reducing maintenance burden for clients.
- **Increases longevity** by preventing systems from devolving into complex and fragile structures dependent on internal details, thereby reducing the need for extensive efforts to redesign or rewrite code, ultimately leading to lower maintenance costs and longer-term viability.
- **Promotes modularization** by offering a focused grouping of functionality for specific tasks or use cases, fostering loosely coupled and modular architectures in application development, where modules operate independently without reliance on internal details of other modules.
- **Reduces code duplication** by consolidating logic behind a single interface, facilitating updates in one place for all clients, thus eliminating redundant implementation code and promoting efficient codebase management.
- **Removes hardcoded assumptions** by offering access to constant values without the need for replication throughout the code base, exemplified by replacing hardcoded values like "myprogram.log" with a GetLogFilename() API call.
- **Easier optimization of API performance** without necessitating changes to client code, enabling enhancements like caching solutions in computationally intensive calculations, as all data access is routed through the API, simplifying management of cached results and recalculations.
- **Parallel development** as two or more developers can work together to implement several functionality offered by API.


## When To Avoid API

- **License restrictions** can affect API suitability; for instance, the GNU GPL may require releasing derived works under its terms, posing challenges for commercial applications, while alternatives like the LGPL or costly commercial API licenses may also present limitations or financial barriers unsuitable for certain projects.
- **Functionality mismatch**, such as requiring a 2D Fourier transform for image processing while the available options are primarily 1D algorithms, potentially limited by platform compatibility or performance criteria.
- **Lack of source code** in closed-source APIs can hinder bug diagnosis and resolution, as you're unable to inspect or modify the source to understand or fix issues, potentially affecting project schedules due to reliance on third-party API owners for bug fixes and patches.
- **Lack of documentation** in an API can lead to uncertainty about its usage and behavior, potentially prompting developers to seek alternative solutions due to difficulties in understanding the API's functionality or lack of trust in poorly documented code.
- **Unnecessary overhead** for internal modules not needing external communication, the extra overhead of establishing a stable public interface may not be warranted, although adhering to API design principles remains beneficial in the long term.


## APIs And SDKs
SDKs (Software Development Kit) are packages installed on computers to develop applications using one or more APIs, 
typically including header files for compilation, library files for linking, and additional resources 
like documentation, example code, and tools. For instance, Apple's iPhone SDK contains frameworks (headers and libraries)
implementing iPhone APIs along with documentation, sample code, IDE templates, and a simulator for app testing.


## File Formats And Network Protocols

Various forms of communication contracts exist in computer applications, with file formats being a common example, 
such as the JPEG File Interchange Format (JFIF) used for image data exchange. These formats define data layout for 
easy interchange between users and facilitate the development of compatible tools and viewers. Similarly, client/server 
applications and middleware services rely on established protocols for data transmission, requiring agreement between 
clients and servers on the format of transmitted data.

![JFIF file format header specification](./pics/JFIF%20file%20format%20header.png "JFIF file format header specification")
*fig: JFIF File Format Header Specification*

While these formats and protocols aren't technically APIs, they serve as standard interfaces for information exchange. 
Nonetheless, it's advisable to develop an associated API alongside these formats or protocols to manage specification 
changes effectively. This practice aids in centralizing knowledge, facilitates future format changes without rewriting 
code, and abstracts complexity, ensuring seamless transitions and compatibility across different versions. This principle 
applies to client/server applications as well, where a common protocol and API allow teams to work independently and accommodate 
major implementation changes with minimal disruption, exemplified by transitions from UDP to TCP in systems like 
[Second Life Viewer](https://secondlife.com/).


![Architectural diagram of Second Life Viewer](./pics/SLV%20architecture.png "APIs layer used by Second life Viewer")
*fig: Architectural diagram of Second Life Viewer*
