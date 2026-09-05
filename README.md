# Snort 3 Windows Integration

[نسخه فارسی](README.fa.md)

Windows build and integration environment for [Snort 3](https://github.com/snort3/snort3) and [LibDAQ](https://github.com/snort3/libdaq).

This repository provides a reproducible Windows integration of Snort 3 and LibDAQ, including the Windows-specific compatibility changes required to build Snort, load DAQ modules, capture live network traffic through Npcap, and execute Snort detection rules.

The repository is designed around two independent upstream changes:

- Snort 3 Windows support
- LibDAQ Windows dynamic-module loading support

The two changes are intentionally maintained as separate upstream pull requests and can be used independently.

---

## Table of Contents

- [Overview](#overview)
- [Why This Repository Exists](#why-this-repository-exists)
- [Architecture](#architecture)
- [Independent Upstream Components](#independent-upstream-components)
- [Upstream Pull Requests](#upstream-pull-requests)
- [Exact Revisions](#exact-revisions)
- [Test Environment](#test-environment)
- [Repository Layout](#repository-layout)
- [How the Integration Works](#how-the-integration-works)
- [Detailed Windows Changes](#detailed-windows-changes)
  - [Snort 3](#snort-3)
  - [LibDAQ](#libdaq)
- [Why Snort and LibDAQ Changes Are Separate](#why-snort-and-libdaq-changes-are-separate)
- [Clone and Initialize](#clone-and-initialize)
- [Build Requirements](#build-requirements)
- [Building LibDAQ](#building-libdaq)
- [Building the DAQ pcap Module](#building-the-daq-pcap-module)
- [Building Snort](#building-snort)
- [Npcap](#npcap)
- [Running Snort](#running-snort)
- [Live Capture](#live-capture)
- [Detection Rule Test](#detection-rule-test)
- [Validation and Testing](#validation-and-testing)
- [Validation Results](#validation-results)
- [Current Limitations](#current-limitations)
- [Upstream Status](#upstream-status)
- [AI-Assisted Development and Review](#ai-assisted-development-and-review)
- [Reproducibility](#reproducibility)
- [License](#license)

---

# Overview

Snort 3 is a network intrusion detection and prevention framework designed primarily around Unix-like environments.

LibDAQ is the Data Acquisition library used by Snort to abstract packet acquisition from the underlying capture mechanism.

On Unix-like systems, Snort and LibDAQ rely on a number of POSIX-specific APIs and assumptions, including:

- POSIX dynamic library loading
- Unix domain sockets
- Unix-specific filesystem functionality
- POSIX signal and synchronization APIs
- Unix-specific networking behavior
- POSIX error and time APIs
- Unix-only build targets
- `.so` dynamic modules

Windows provides different APIs and runtime behavior for several of these mechanisms.

The purpose of this repository is to provide the Windows-specific compatibility work required to make Snort 3 and its DAQ integration operate correctly in a Windows environment.

The integration has been tested on:

- Windows 10 Enterprise 64-bit
- MSYS2 UCRT64
- Snort 3.12.2.0
- LibDAQ 3.0.27
- Npcap 1.88

The resulting system was validated against live network traffic.

No private network addresses, interface GUIDs, usernames, or machine-specific filesystem paths are required by this documentation.

---

# Why This Repository Exists

This repository is not intended to replace the upstream Snort or LibDAQ projects.

Instead, it serves three purposes:

1. Provide a reproducible Windows build and integration environment.
2. Maintain the Windows-specific changes while upstream review is in progress.
3. Demonstrate that Snort 3 and LibDAQ can operate together on Windows using Npcap.

The project also separates the two portability problems.

Snort requires Windows compatibility across a relatively large portion of its source tree.

LibDAQ has a much narrower Windows-specific requirement: dynamic DAQ module discovery and loading.

Keeping these changes separate makes both projects easier to review and allows either change to be used independently.

---

# Architecture

The integration consists of three main components:

```text
                    Windows 10
                        |
                        |
                     Npcap
                        |
                        v
                 DAQ pcap module
                  daq_pcap.dll
                        |
                        v
                     LibDAQ
                  (DAQ 3.0.27)
                        |
                        v
                     Snort 3
                  (3.12.2.0)
                        |
                        v
                Detection Engine
                        |
                        v
                    Alerts
```

The responsibilities are separated as follows.

### Npcap

Npcap provides packet capture capabilities on Windows.

### LibDAQ

LibDAQ provides Snort with an abstraction layer for packet acquisition.

The pcap DAQ module interfaces with Npcap.

### Snort

Snort loads the DAQ module, receives packets, processes them through its inspection and detection engines, and generates alerts.

---

# Independent Upstream Components

The Snort and LibDAQ changes are intentionally independent.

## Snort 3 Windows Support

The Snort changes provide Windows compatibility throughout the Snort source tree.

They address areas including:

- CMake configuration
- Windows networking
- process management
- threading
- filesystem handling
- logging
- plugin loading
- platform-specific APIs
- Unix-only connectors
- Unix-only transports
- time handling
- error handling
- wildcard matching
- dynamic modules
- RPC-related build checks
- Windows linker requirements

This work is submitted independently to the Snort repository.

## LibDAQ Windows Support

The LibDAQ changes address dynamic DAQ module loading.

The Windows implementation uses:

```text
LoadLibraryA()
GetProcAddress()
FreeLibrary()
```

instead of the POSIX:

```text
dlopen()
dlsym()
dlclose()
```

The existing POSIX behavior remains available on non-Windows systems.

This work is submitted independently to the LibDAQ repository.

---

# Upstream Pull Requests

## Snort 3

**Pull Request #478**

Title:

```text
Add Windows support for Snort
```

Repository:

```text
snort3/snort3
```

Source branch:

```text
Mahbodbe:windows-support
```

Target branch:

```text
snort3:master
```

Pull request:

https://github.com/snort3/snort3/pull/478

Current status:

```text
Open
Not merged
```

The pull request contains:

```text
1 commit
54 changed files
1138 additions
115 deletions
```

The PR is specifically intended to add Windows platform support while preserving the existing non-Windows code paths.

---

## LibDAQ

**Pull Request #43**

Title:

```text
Add Windows support for LibDAQ
```

Repository:

```text
snort3/libdaq
```

Source branch:

```text
Mahbodbe:windows-support
```

Target branch:

```text
snort3:master
```

Pull request:

https://github.com/snort3/libdaq/pull/43

Current status:

```text
Open
Not merged
```

The pull request contains:

```text
1 commit
1 changed file
81 additions
8 deletions
```

The LibDAQ change is deliberately limited to:

```text
api/daq_base.c
```

---

# Exact Revisions

The integration repository pins the exact revisions used during validation.

## Snort 3

Version:

```text
3.12.2.0
```

Windows-support commit:

```text
e976668958270f82e45b834e601ae28d868b7c31
```

Short form:

```text
e976668
```

Branch:

```text
windows-support
```

---

## LibDAQ

Version:

```text
3.0.27
```

Windows-support commit:

```text
c97c07e8207898c8292eef0db129788d5335df67
```

Short form:

```text
c97c07e
```

Branch:

```text
windows-support
```

These exact revisions are used by the integration submodules.

---

# Test Environment

The integration was tested in the following environment.

| Component | Version / Configuration |
|---|---|
| Operating System | Windows 10 Enterprise 64-bit |
| Build Environment | MSYS2 UCRT64 |
| Compiler | GCC 16.2.0 |
| Build Tool | GNU Make 4.4.1 |
| Snort | 3.12.2.0 |
| LibDAQ | 3.0.27 |
| Packet Capture | Npcap 1.88 |
| Architecture | x86_64 |

The validation used a Windows network interface through Npcap.

Machine-specific network addresses and interface identifiers are intentionally omitted from this documentation.

---

# Repository Layout

```text
snort3-windows/
│
├── snort3/
│   └── Snort 3 source at the Windows-support revision
│
├── libdaq/
│   └── LibDAQ source at the Windows-support revision
│
├── patches/
│   └── Additional integration patches, if required
│
├── scripts/
│   └── Build and validation helper scripts
│
├── docs/
│   └── Additional documentation
│
├── .gitmodules
│
├── README.md        # English docs
├── README.fa.md     # Persian docs
└── LICENSE          # GPL-2.0 (upstream terms apply)
```

The `snort3` and `libdaq` directories are Git submodules.

This allows the integration repository to reference exact fork revisions instead of copying the entire source trees into the integration repository.

---

# How the Integration Works

The integration follows this dependency chain:

```text
Snort
  |
  | DAQ API
  v
LibDAQ
  |
  | pcap DAQ module
  v
daq_pcap.dll
  |
  | packet capture API
  v
Npcap
  |
  v
Windows network interface
```

Snort itself does not need to know the low-level details of the Windows packet capture implementation.

LibDAQ provides the abstraction.

The DAQ pcap module communicates with Npcap and supplies captured packets to Snort.

This separation is important because the Windows support is not a single monolithic modification.

---

# Detailed Windows Changes

# Snort 3

The Snort Windows-support commit modifies multiple platform-dependent areas.

The changes are grouped below by functionality.

---

## 1. Windows Build and Linker Support

Several Unix-specific build assumptions are not applicable to Windows.

The Windows build therefore adds platform-specific CMake handling.

For example, Snort links against:

```text
ws2_32
```

on Windows.

`ws2_32` provides the Windows Winsock networking APIs required by networking-related code.

The Windows executable also requires appropriate symbol/export handling for dynamic modules.

The Windows CMake configuration therefore adds:

```text
--export-all-symbols
```

to the linker configuration.

Unix-only object targets are also excluded from the Windows executable where they are not applicable.

This allows the Windows build to use the same general Snort architecture while avoiding dependencies that only exist on Unix-like systems.

---

## 2. Unix-Only Transport Handling

Snort contains transport components that depend on Unix-specific functionality.

For example:

```text
mp_unix_transport
```

is not appropriate for a native Windows build.

The Windows build therefore excludes this Unix-specific object from the executable.

This avoids forcing Windows builds to provide functionality that only exists on Unix-like systems.

---

## 3. Unix Domain Connectors

Unix domain connectors depend on Unix-specific IPC mechanisms.

Windows does not provide the same Unix-domain socket implementation expected by the existing Snort code.

The connector loader therefore avoids loading Unix-specific connector implementations on Windows.

This includes:

```text
tcp_connector
unixdomain_connector
```

where their existing implementations depend on Unix-specific functionality.

The associated platform-specific statistics objects are also handled appropriately for Windows compilation.

The objective is not to remove the connector framework, but to prevent unsupported Unix-specific components from breaking a Windows build.

---

## 4. Unix Socket Logger

The `alert_unixsock` logger depends on Unix-domain sockets.

It is therefore excluded from Windows builds.

The CMake configuration conditionally adds this logger only on non-Windows platforms.

This preserves the existing logger on Unix systems while preventing an unsupported dependency from breaking Windows builds.

---

## 5. Time API Compatibility

Snort uses:

```text
ctime_r()
```

on POSIX systems.

Windows provides:

```text
ctime_s()
```

instead.

The Windows code therefore uses:

```c
ctime_s(time_buf, sizeof(time_buf), &now);
```

while preserving:

```c
ctime_r(&now, time_buf);
```

for non-Windows platforms.

This is a portability adaptation rather than a behavioral redesign.

---

## 6. Error String Handling

Some POSIX code uses:

```text
strerror_r()
```

while the Microsoft runtime provides:

```text
strerror_s()
```

for the corresponding safe error-string operation.

Windows-specific branches therefore use `strerror_s()` where required.

The existing POSIX implementation remains unchanged.

This approach keeps the platform-specific API differences isolated behind conditional compilation.

---

## 7. File Access

Some file DAQ functionality used:

```text
O_NONBLOCK
```

when opening files.

This behavior is not directly portable to the Windows implementation.

The Windows path therefore uses:

```text
open(filename, O_RDONLY)
```

while the POSIX implementation retains:

```text
open(filename, O_RDONLY | O_NONBLOCK)
```

This keeps the original behavior for Unix-like systems while allowing the file DAQ component to compile and operate under Windows.

---

## 8. Filesystem Wildcard Matching

Snort previously relied on:

```text
fnmatch.h
```

for wildcard matching.

That header/API is not generally available in the same form on Windows.

A small internal wildcard implementation was therefore introduced.

The implementation supports the wildcard semantics required by the affected Snort code, including:

```text
*
?
```

The Windows-specific code replaces the dependency on `fnmatch()` with:

```text
wildcard_match()
```

This keeps wildcard processing within the Snort source tree and avoids introducing an additional Windows-specific external dependency.

---

## 9. Signal-Safe File Synchronization

Some Unix code uses:

```text
fsync()
```

to synchronize file descriptors.

The Windows implementation does not provide the same POSIX semantics through the same API.

The Windows path therefore avoids invoking the Unix `fsync()` implementation in the affected signal-safe logger path.

The existing POSIX implementation remains unchanged.

---

## 10. RPC Build Check

The Snort build system checks for an RPC program database implementation.

On Unix-like systems this can involve:

```text
getrpcent()
```

and potentially:

```text
TIRPC
```

Windows does not provide the same RPC program database interface.

The Windows build therefore disables the affected RPC service detector instead of treating the missing Unix RPC functionality as a fatal configuration error.

This allows the remainder of Snort to build successfully on Windows.

---

## 11. `ffs()` Compatibility

The code previously used:

```text
ffs()
```

for finding the first set bit.

The Windows-compatible implementation uses:

```text
__builtin_ffs()
```

which is provided by the GCC toolchain used by the MSYS2 UCRT64 environment.

This avoids depending on a POSIX implementation of `ffs()`.

---

## 12. Windows Plugin and Module Handling

Snort relies heavily on dynamically loaded modules.

The Windows build therefore requires appropriate handling of dynamic modules and symbol visibility.

The build configuration was adjusted to support the Windows dynamic-module environment while retaining the existing module architecture.

This is particularly important because the DAQ layer is itself dynamically loaded.

---

## 13. Thread-Local Statistics

Some connector modules declare statistics using thread-local storage.

The Windows compiler/toolchain requires the appropriate definitions to exist when the normal Unix implementation is not compiled.

Windows-specific definitions were therefore added for the affected connector statistics.

This allows the modules to compile without requiring the Unix-specific implementation files.

---

## 14. Windows Networking

Windows networking uses Winsock.

The Windows build therefore links against:

```text
ws2_32
```

This provides the networking symbols required by Snort and its Windows-compatible components.

The platform-specific code is guarded using Windows checks such as:

```c
#ifdef _WIN32
```

so the Windows implementation does not replace the Unix implementation.

---

## 15. Windows-Specific CMake Source Selection

A recurring pattern throughout the changes is conditional source selection.

Instead of attempting to compile every Unix source file on Windows, CMake selectively excludes components whose underlying platform APIs are unavailable.

Conceptually:

```text
                    Snort source tree
                           |
              +------------+------------+
              |                         |
           Windows                   POSIX
              |                         |
       Windows-compatible         Existing Unix
          components                components
              |                         |
              +------------+------------+
                           |
                    Snort executable
```

This approach minimizes behavioral changes to the existing non-Windows implementation.

---

# LibDAQ

The LibDAQ Windows change is intentionally much smaller.

The primary problem is dynamic module loading.

---

## 1. POSIX Dynamic Loading

On Unix-like systems LibDAQ uses:

```c
dlopen()
dlsym()
dlclose()
dlerror()
```

These APIs are provided through the traditional POSIX/Linux dynamic-loading model.

Windows does not provide these APIs in the same way.

---

## 2. Windows Dynamic Loading

Windows provides dynamic library loading through:

```text
LoadLibraryA()
GetProcAddress()
FreeLibrary()
```

The LibDAQ Windows implementation introduces an abstraction layer:

```text
daq_dlopen()
daq_dlsym()
daq_dlclose()
daq_dlerror()
```

On Windows these wrappers map to the Windows loader APIs.

On non-Windows platforms they map to:

```text
dlopen()
dlsym()
dlclose()
dlerror()
```

This creates a common interface for the rest of LibDAQ.

---

## 3. Why the Abstraction Is Important

Without this abstraction, the LibDAQ source would need to contain Windows-specific logic at every location where a dynamic module is opened or closed.

Instead, the platform-specific behavior is isolated.

Conceptually:

```text
                 LibDAQ
                    |
             daq_dlopen()
                    |
          +---------+---------+
          |                   |
       Windows             POSIX
          |                   |
   LoadLibraryA()          dlopen()
```

The same pattern is used for symbol lookup and module unloading.

This reduces platform-specific code throughout the rest of LibDAQ.

---

## 4. Dynamic Module Extension

Unix DAQ modules use:

```text
.so
```

Windows dynamic modules use:

```text
.dll
```

The DAQ module discovery logic therefore uses:

```text
.dll
```

on Windows and retains:

```text
.so
```

on POSIX systems.

This is required for LibDAQ to discover:

```text
daq_pcap.dll
```

in the Windows DAQ module directory.

---

## 5. Windows Error Reporting

When a Windows dynamic-loading operation fails, the implementation records the Windows error code obtained from:

```text
GetLastError()
```

The error is then exposed through the LibDAQ abstraction.

This provides useful diagnostic information while preserving the existing LibDAQ error-reporting structure.

---

## 6. Dynamic Module Lifecycle

The Windows-specific implementation covers the complete dynamic module lifecycle:

```text
Discover module
      |
      v
daq_dlopen()
      |
      v
LoadLibraryA()
      |
      v
Get module handle
      |
      v
daq_dlsym()
      |
      v
GetProcAddress()
      |
      v
Use DAQ module
      |
      v
daq_dlclose()
      |
      v
FreeLibrary()
```

This is important because simply compiling `daq_pcap.dll` is not sufficient.

LibDAQ must also be able to discover, load, resolve symbols from, and unload the module correctly.

---

# Why Snort and LibDAQ Changes Are Separate

Although Snort and LibDAQ are used together, they solve different portability problems.

Snort's Windows work covers a broad set of platform assumptions throughout the application.

LibDAQ's Windows work is primarily concerned with dynamic module loading and module discovery.

Therefore:

```text
Snort Windows Support
        |
        +---- independent change
        |
        v
Snort application
```

and:

```text
LibDAQ Windows Support
        |
        +---- independent change
        |
        v
DAQ module loading
```

The integrated stack combines both:

```text
Snort Windows Support
          +
LibDAQ Windows Support
          +
Npcap
          =
Working Windows integration
```

However, neither upstream change conceptually depends on the other being merged into the same commit or pull request.

This separation also makes upstream review easier.

---

# Clone and Initialize

Clone the integration repository:

```bash
git clone https://github.com/Mahbodbe/snort3-windows.git
cd snort3-windows
```

Initialize the Git submodules:

```bash
git submodule update --init --recursive
```

The repository contains two submodules:

```text
snort3
libdaq
```

To verify the exact revisions:

```bash
git submodule status
```

The expected revisions are:

```text
e976668  snort3
c97c07e  libdaq
```

The submodules point to the Windows-support branches of the corresponding forks.

---

# Build Requirements

The tested build environment uses:

- Windows 10 Enterprise 64-bit
- MSYS2
- UCRT64 environment
- GCC
- GNU Make
- CMake
- pkg-config
- Git
- Npcap

The MSYS2 UCRT64 environment is important because the build uses a native Windows-oriented toolchain rather than WSL.

---

# Building LibDAQ

Enter the LibDAQ source directory:

```bash
cd libdaq
```

Configure LibDAQ using the Windows-compatible configuration.

The following example uses a generic installation prefix:

```bash
./configure \
  --prefix=/path/to/snort3-windows/install \
  --disable-shared \
  --enable-static \
  --disable-afpacket-module \
  --disable-bpf-module \
  --disable-divert-module \
  --disable-dump-module \
  --disable-fst-module \
  --disable-netmap-module \
  --disable-nfq-module \
  --disable-savefile-module \
  --disable-trace-module \
  --disable-gwlb-module
```

Build the LibDAQ API:

```bash
make -C api
```

Build the DAQ modules:

```bash
make -C modules
```

Install the API:

```bash
make -C api install
```

Install the modules:

```bash
make -C modules install
```

The installation provides the LibDAQ headers and libraries required by Snort.

---

# Building the DAQ pcap Module

The Windows integration requires the pcap DAQ module to be available as a Windows dynamic library.

The resulting module is:

```text
daq_pcap.dll
```

The module is built against Npcap's packet capture library.

The required module directory is:

```text
install/lib/daq/
```

After building the pcap DAQ module, copy it into the DAQ module directory:

```bash
mkdir -p /path/to/snort3-windows/install/lib/daq

cp modules/pcap/.libs/daq_pcap.dll \
   /path/to/snort3-windows/install/lib/daq/
```

The resulting DLL requires the Npcap runtime, including:

```text
wpcap.dll
```

as well as the required Windows runtime libraries.

---

# Building Snort

Enter the Snort source directory:

```bash
cd ../snort3
```

Create a build directory:

```bash
mkdir -p build
cd build
```

Configure the project with CMake using the installed LibDAQ:

```bash
cmake .. \
  -G "MSYS Makefiles" \
  -DCMAKE_PREFIX_PATH=/path/to/snort3-windows/install
```

Build:

```bash
cmake --build . -j$(nproc)
```

Install:

```bash
cmake --install .
```

After installation, verify the Snort version:

```bash
snort.exe -V
```

The expected version is:

```text
Snort++ 3.12.2.0
```

The output should also report:

```text
DAQ 3.0.27
```

---

# Npcap

Npcap provides the packet capture functionality used by the pcap DAQ module.

The tested environment uses:

```text
Npcap 1.88
```

Npcap must be installed on the Windows host before live packet capture can be tested.

Snort accesses Windows network interfaces through Npcap's packet capture interface.

The DAQ layer therefore follows:

```text
Snort
  |
LibDAQ
  |
daq_pcap.dll
  |
Npcap
  |
Windows network interface
```

Machine-specific interface names and identifiers are intentionally omitted from this documentation.

---

# Running Snort

After installation, verify the DAQ modules.

Snort should be able to discover the Windows DAQ module:

```text
daq_pcap.dll
```

A successful module discovery test produced output equivalent to:

```text
Loading modules in:
<installation-prefix>/lib/daq

Registered daq module: pcap
Found module daq_pcap.dll
ret = 0
module: pcap
```

This confirms that:

1. The DAQ search path is correct.
2. The pcap DAQ module is visible.
3. LibDAQ can load the Windows DLL.
4. The DAQ module registers successfully.

A direct Windows loader test also confirmed:

```text
LoadLibrary OK
DAQ_MODULE_DATA OK
```

This verifies that the Windows dynamic loading path can load the DAQ DLL and resolve the expected module data.

---

# Live Capture

After the DAQ module has been installed and Npcap is running, Snort can open an Npcap network interface.

The exact interface identifier depends on the Windows machine and should not be hard-coded in public documentation.

A successful live-capture test demonstrated that:

```text
Snort
    |
    v
LibDAQ
    |
    v
daq_pcap.dll
    |
    v
Npcap
    |
    v
Live network traffic
```

was functioning correctly.

---

# Detection Rule Test

A simple ICMP rule was used to validate the complete packet-processing path.

Example rule:

```text
alert icmp any any -> any any
(
    msg:"ICMP ECHO REQUEST";
    itype:8;
    sid:1000001;
    rev:5;
)
```

The rule matches ICMP Echo Request packets.

An ICMP Echo Request was generated from the Windows test host toward the local network gateway.

Snort successfully processed the live packet and generated an alert equivalent to:

```text
[1:1000001:5] "ICMP ECHO REQUEST" {ICMP}
<test-host> -> <local-gateway>
```

The actual IP addresses are intentionally not included in this repository.

This test validates the entire path:

```text
Generated ICMP packet
        |
        v
Windows network stack
        |
        v
Npcap
        |
        v
daq_pcap.dll
        |
        v
LibDAQ
        |
        v
Snort
        |
        v
ICMP detection rule
        |
        v
Alert
```

---

# Validation and Testing

The integration was validated at multiple levels.

## 1. Snort Version

Verified:

```text
Snort++ 3.12.2.0
```

## 2. LibDAQ Version

Verified:

```text
DAQ 3.0.27
```

## 3. DAQ Module Discovery

Verified that:

```text
daq_pcap.dll
```

was discovered by LibDAQ.

## 4. Windows Dynamic Loading

A direct Windows loader test confirmed:

```text
LoadLibrary OK
DAQ_MODULE_DATA OK
```

## 5. Npcap Capture

Snort successfully opened an Npcap network interface.

## 6. Live Traffic

Snort successfully received live packets through the pcap DAQ module.

## 7. Detection

The ICMP Echo Request rule generated an alert from live traffic.

These tests collectively validate more than compilation alone.

---

# Validation Results

The final validation demonstrated the following:

| Test | Result |
|---|---|
| Snort Windows build | PASS |
| LibDAQ Windows build | PASS |
| Snort executable startup | PASS |
| DAQ module discovery | PASS |
| `daq_pcap.dll` loading | PASS |
| Windows `LoadLibrary` test | PASS |
| Npcap interface access | PASS |
| Live packet capture | PASS |
| ICMP detection rule | PASS |
| Alert generation | PASS |

The most important result is that the system was tested against live traffic rather than only being validated through compilation.

---

# Current Limitations

This project should be considered Windows platform support and integration work rather than a claim of complete feature parity between Windows and every Unix-like Snort environment.

Some Snort functionality is inherently platform-specific.

The Windows build therefore excludes or adapts components that depend on Unix-only functionality, including areas such as:

- Unix-domain socket functionality
- Unix-only transports
- Unix socket logging
- POSIX-specific APIs
- Unix-specific RPC functionality
- POSIX filesystem matching APIs

The purpose of these changes is to make the core Snort build and DAQ packet-capture path functional on Windows without unnecessarily changing the existing non-Windows implementation.

Additional testing is required before claiming complete Windows feature parity.

---

# Upstream Status

The Windows changes are maintained in separate upstream pull requests.

## Snort

PR:

https://github.com/snort3/snort3/pull/478

Status:

```text
Open
Not merged
```

Commit:

```text
e976668958270f82e45b834e601ae28d868b7c31
```

## LibDAQ

PR:

https://github.com/snort3/libdaq/pull/43

Status:

```text
Open
Not merged
```

Commit:

```text
c97c07e8207898c8292eef0db129788d5335df67
```

Until upstream maintainers merge these changes, this integration repository references the corresponding fork branches.

The integration repository should therefore be understood as a working Windows integration based on the submitted changes, not as an assertion that the changes are already part of upstream releases.

---

# AI-Assisted Development and Review

Parts of the development, debugging, portability analysis, build troubleshooting, documentation, and code review process were performed with assistance from AI tools.

AI assistance was used as a development aid, including:

- identifying likely POSIX-to-Windows incompatibilities
- analyzing compiler and linker errors
- suggesting platform-specific API mappings
- reviewing conditional compilation
- investigating build-system behavior
- checking dynamic-module loading logic
- helping structure the integration repository
- reviewing documentation and reproducibility steps

The resulting code was not accepted solely on the basis of AI-generated suggestions.

The implementation was manually inspected, built, tested, and reviewed in the target Windows environment.

The validation included actual execution of Snort, DAQ module discovery, dynamic DLL loading, Npcap live capture, and a live ICMP detection test.

Therefore, AI assistance was part of the development workflow, while the final implementation and test results were reviewed and validated in the target environment.

---

# Reproducibility

The integration is designed so that another developer can reproduce the environment without relying on the original machine.

The repository avoids publishing:

- local IP addresses
- gateway addresses
- network-interface GUIDs
- Windows usernames
- personal filesystem paths
- machine-specific configuration

Instead, the documentation uses placeholders such as:

```text
/path/to/snort3-windows
```

and:

```text
<test-host>
<local-gateway>
```

The exact software revisions are explicitly documented:

```text
Snort 3.12.2.0
Commit: e976668958270f82e45b834e601ae28d868b7c31

LibDAQ 3.0.27
Commit: c97c07e8207898c8292eef0db129788d5335df67

Npcap 1.88
```

This makes the project reproducible without exposing details of the original test network.

---

# License

This integration repository contains references to and submodules from the upstream Snort and LibDAQ projects.

The licensing terms of those projects apply to their respective source code.

Refer to the upstream repositories for the authoritative license information:

- Snort 3: https://github.com/snort3/snort3
- LibDAQ: https://github.com/snort3/libdaq

Any additional scripts or documentation introduced specifically by this integration repository should be considered under the license selected for those files by the repository maintainers.

---

# Conclusion

This repository demonstrates a working Windows integration of Snort 3, LibDAQ, and Npcap.

The work is divided into two independent portability changes:

```text
Snort Windows Support
        +
LibDAQ Windows Support
        +
Npcap
        |
        v
Snort 3 running on Windows
```

The Snort changes address the broader set of Windows compatibility issues throughout the Snort source tree.

The LibDAQ changes provide the Windows dynamic-library loading required to discover and load DAQ modules such as:

```text
daq_pcap.dll
```

The complete integration was built and tested on Windows 10 Enterprise 64-bit using MSYS2 UCRT64 and Npcap.

The final validation confirmed:

- successful Snort compilation
- successful LibDAQ compilation
- successful Windows DAQ module loading
- successful Npcap interface access
- successful live packet capture
- successful ICMP rule processing
- successful alert generation

The corresponding Windows-support changes have also been submitted as separate upstream pull requests so that the work can be reviewed independently by the respective maintainers.

---

# References

- Snort 3:
  https://github.com/snort3/snort3

- LibDAQ:
  https://github.com/snort3/libdaq

- Snort Windows Support PR #478:
  https://github.com/snort3/snort3/pull/478

- LibDAQ Windows Support PR #43:
  https://github.com/snort3/libdaq/pull/43

- Integration Repository:
  https://github.com/Mahbodbe/snort3-windows

- Npcap:
  https://npcap.com/

