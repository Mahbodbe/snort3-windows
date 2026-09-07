# Snort 3 Windows Integration

[Persian documentation](README.fa.md)

A reproducible Windows integration environment for [Snort 3](https://github.com/snort3/snort3) and [LibDAQ](https://github.com/snort3/libdaq), including the Windows-specific compatibility work required to build Snort, load DAQ modules, capture traffic through Npcap, and process live packets.

The repository is intended for Windows-based IDS/IPS research and laboratory environments, including SCADA/ICS cybersecurity testbeds.

---

## Table of Contents

- [Overview](#overview)
- [Architecture](#architecture)
- [Test Environment](#test-environment)
- [Upstream Components](#upstream-components)
- [Exact Revisions](#exact-revisions)
- [Windows Compatibility Work](#windows-compatibility-work)
- [LibDAQ Windows Support](#libdaq-windows-support)
- [Windows Build Environment](#windows-build-environment)
- [Building LibDAQ](#building-libdaq)
- [Building Snort](#building-snort)
- [Npcap Setup](#npcap-setup)
- [Npcap Loopback Capture](#npcap-loopback-capture)
- [Snort DLT_NULL Codec](#snort-dlt_null-codec)
- [Running Snort on Windows](#running-snort-on-windows)
- [SCADA/DNP3 Loopback Test](#scadadnp3-loopback-test)
- [Validation Results](#validation-results)
- [Troubleshooting](#troubleshooting)
- [Current Limitations](#current-limitations)
- [Upstream Status](#upstream-status)
- [AI-Assisted Development](#ai-assisted-development)
- [Reproducibility](#reproducibility)
- [License](#license)

---

# Overview

Snort 3 is a network intrusion detection and prevention framework. LibDAQ provides the packet-acquisition abstraction used by Snort. On Unix-like systems, both projects rely on a number of POSIX-specific APIs and conventions that are not directly available on Windows.

This repository provides the Windows-specific compatibility work required to build and run Snort 3 with LibDAQ and Npcap on native Windows, without WSL or a virtual machine.

The integration was developed and tested on:

- Windows 10 Enterprise 64-bit
- MSYS2 UCRT64
- GCC 16.2.0
- GNU Make 4.4.1
- CMake
- Snort 3.12.2.0
- LibDAQ 3.0.27
- Npcap 1.88

The final system has been exercised through Snort startup, DAQ loading, Npcap interface access, live packet processing, and a Windows loopback capture path.

---

# Architecture

The normal Windows capture path is:

```text
Windows network interface
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
     Snort 3
        |
        v
Detection / Inspection
        |
        v
      Alerts
```

For local SCADA/ICS applications communicating over `127.0.0.1`, the path is slightly different because Windows loopback traffic is exposed by Npcap using the BSD/Npcap loopback data-link format:

```text
SCADA / DNP3 application
        |
        | TCP / IPv4
        v
  127.0.0.1 loopback
        |
        v
Npcap Loopback Adapter
        |
        | DLT_NULL (0)
        v
    LibDAQ / pcap
        |
        v
      Snort 3
        |
        v
   DLT_NULL codec
        |
        v
 IPv4 / TCP / application inspection
```

This loopback path required an additional Snort codec described below.

---

# Test Environment

| Component | Version / Configuration |
|---|---|
| Operating System | Windows 10 Enterprise 64-bit |
| Build Environment | MSYS2 UCRT64 |
| Compiler | GCC 16.2.0 |
| Build Tool | GNU Make 4.4.1 |
| Build System | CMake |
| Snort | 3.12.2.0 |
| LibDAQ | 3.0.27 |
| Npcap | 1.88 |
| Architecture | x86_64 |

The documentation intentionally avoids publishing machine-specific usernames, filesystem paths, IP addresses, and interface GUIDs.

---

# Upstream Components

The integration is based on two independent portability changes.

## Snort 3 Windows Support

The Snort Windows work addresses platform dependencies throughout the source tree, including build configuration, Windows networking, filesystem handling, time and error APIs, plugin handling, Unix-only connectors and transports, logging, and other POSIX-specific functionality.

## LibDAQ Windows Support

The LibDAQ Windows work focuses on dynamic DAQ module loading and discovery. Windows uses:

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

The two portability changes are maintained independently so that each can be reviewed and upstreamed separately.

---

# Exact Revisions

## Snort 3

```text
Version: 3.12.2.0
Windows-support commit: e976668958270f82e45b834e601ae28d868b7c31
Short form: e976668
Branch: windows-support
```

## LibDAQ

```text
Version: 3.0.27
Windows-support commit: c97c07e8207898c8292eef0db129788d5335df67
Short form: c97c07e
Branch: windows-support
```

---

# Windows Compatibility Work

The Snort Windows implementation contains platform-specific changes in several areas.

## Build and linker support

Windows builds use platform-specific CMake handling and link against:

```text
ws2_32
```

for Winsock functionality. Windows symbol/export handling is also configured so that the plugin architecture can operate correctly.

## Unix-only components

Components that depend directly on Unix-only functionality are excluded or adapted on Windows. Examples include Unix transports, Unix-domain connectors, Unix socket logging, and other platform-specific modules.

## Time and error APIs

Windows-compatible alternatives are used where required, including:

```text
ctime_s()   instead of ctime_r()
strerror_s() instead of strerror_r()
```

The existing POSIX paths remain available for non-Windows builds.

## Filesystem and wildcard handling

Windows-specific handling is provided for areas that previously depended on POSIX interfaces such as `fnmatch()` and Unix-specific file flags.

## RPC and other Unix dependencies

Build checks and optional functionality that depend on Unix RPC APIs are disabled or adapted when those APIs are not available on Windows.

The goal is to preserve the existing non-Windows behavior while allowing the core Snort packet-processing path to compile and run natively on Windows.

---

# LibDAQ Windows Support

LibDAQ provides the abstraction between Snort and packet acquisition.

The Windows implementation introduces platform-neutral dynamic-loader wrappers:

```text
daq_dlopen()
daq_dlsym()
daq_dlclose()
daq_dlerror()
```

On Windows these map to the Windows loader APIs. On POSIX systems they continue to use the corresponding POSIX APIs.

Windows DAQ modules are discovered as `.dll` files rather than Unix `.so` files. This allows modules such as:

```text
daq_pcap.dll
```

to be discovered and loaded by LibDAQ.

---

# Windows Build Environment

The build was performed natively on Windows using MSYS2 UCRT64.

The required development packages included:

```bash
pacman -S base-devel \
  mingw-w64-ucrt-x86_64-toolchain \
  mingw-w64-ucrt-x86_64-cmake \
  mingw-w64-ucrt-x86_64-pkgconf \
  git
```

The toolchain used during validation reported:

```text
GCC 16.2.0
GNU Make 4.4.1
```

No virtual machine or WSL environment is required for this integration.

---

# Building LibDAQ

The LibDAQ source was built using the Windows-support revision.

A typical source build follows this sequence:

```bash
./bootstrap
./configure --prefix=/path/to/snort3-windows/install
make -j$(nproc)
make install
```

The resulting DAQ modules include the pcap module required for Npcap capture.

The tested installation produced DAQ modules including:

```text
daq_file.dll
daq_hext.dll
daq_pcap.dll
daq_pcap_msyst2.dll
```

The installed DAQ directory is supplied to Snort explicitly when required:

```text
--daq-dir <installation-prefix>/lib/snort/daq
```

---

# Building Snort

The Snort 3 source was built with CMake from the Windows-support source tree.

The final build was performed with:

```bash
cmake -S . -B build
cmake --build build -j4
```

The resulting executable reports:

```text
Snort++ Version 3.12.2.0
Using DAQ version 3.0.27
Using Npcap version 1.88
```

The build completed successfully with:

```text
[.../...] Linking CXX executable src\\snort.exe
```

---

# Npcap Setup

Npcap 1.88 was installed on the Windows host.

The Npcap driver was verified to be running:

```text
SERVICE_NAME: npcap
STATE: 4 RUNNING
WIN32_EXIT_CODE: 0
```

For ordinary Ethernet capture, Snort can use an Npcap-backed network interface directly.

For the SCADA test environment, however, the relevant communication was initially local loopback traffic, so the Npcap Loopback Adapter was required.

Npcap provides `NPFInstall.exe` for managing the loopback adapter. The loopback adapter was installed with:

```cmd
"C:\Program Files\Npcap\NPFInstall.exe" -il
```

Successful installation returned:

```text
Npcap Loopback adapter has been successfully installed!
```

The resulting device was confirmed through Windows Plug and Play enumeration as:

```text
Npcap Loopback Adapter
Status: Started
Driver Name: netloop.inf
```

The Npcap registry configuration also showed loopback support enabled.

---

# Npcap Loopback Capture

The SCADA test applications communicate locally using TCP over `127.0.0.1`.

The observed connection was:

```text
TCP 0.0.0.0:22000       0.0.0.0:0       LISTENING
TCP 127.0.0.1:6901      127.0.0.1:22000 ESTABLISHED
TCP 127.0.0.1:22000     127.0.0.1:6901  ESTABLISHED
```

The test applications were identified as:

```text
tmwtest.exe
PAYA_DNP3.exe
```

The TMW DNP3 slave configuration used TCP/IP with the local peer at `127.0.0.1` and TCP port `22000`.

This means that the DNP3 communication was not traversing a physical Ethernet interface. It was entirely local loopback traffic.

Npcap exposes this traffic using `DLT_NULL`, whose value is:

```text
DLT_NULL = 0
```

This distinction was critical for the Snort integration.

---

# Snort DLT_NULL Codec

## Problem

The first attempt to capture the Npcap Loopback Adapter reached the DAQ layer successfully but Snort stopped with:

```text
No codec found for data link type 0
No codec for DAQ base protocol, stopping packet processing
```

This proved that:

1. Npcap was installed correctly.
2. The loopback adapter was available.
3. LibDAQ/pcap could open the adapter.
4. Snort received `DLT_NULL` as the DAQ base protocol.
5. The missing component was a Snort codec for DLT 0.

The issue was therefore not a network-interface or Npcap installation failure.

## Implementation

A native Snort codec was added at:

```text
src/codecs/root/cd_null.cc
```

The codec:

- registers itself as the `null` codec
- declares support for `DLT_NULL`
- consumes the 4-byte BSD/Npcap loopback family header
- recognizes IPv4 (`AF_INET`, family value `2`)
- recognizes IPv6 (`family value `24` in the Npcap loopback format)
- forwards the remaining payload to Snort's IPv4/IPv6 processing chain

The core behavior is conceptually:

```text
DLT_NULL packet
     |
     | 4-byte address-family header
     v
family == 2   -> IPv4
family == 24  -> IPv6
```

The codec was added to the static root codec target in:

```text
src/codecs/root/CMakeLists.txt
```

Specifically, `cd_null.cc` was included alongside the existing root codecs.

## Static plugin registration

A second issue appeared after the codec compiled successfully.

Snort's static codec architecture does not automatically discover every object file. The static plugin arrays are explicitly declared and loaded by `src/codecs/codec_api.cc`.

Therefore the following declaration was added:

```cpp
extern const BaseApi* cd_null[];
```

and the codec was explicitly loaded with:

```cpp
PluginManager::load_plugins(cd_null);
```

Without these two registrations, the codec object existed in the executable but `CodecManager` never instantiated it.

After registration, `CodecManager::thread_init()` could match:

```text
DAQ DLT = 0
```

with:

```text
cd_null -> DLT_NULL
```

This removed the `No codec found for data link type 0` failure.

## Result

The modified Snort executable was rebuilt successfully:

```bash
cmake --build build -j4
```

The loopback capture test then reached active packet processing:

```text
pcap DAQ configured to passive.
Commencing packet processing
Retry queue interval is: 200 ms
++ [0] \\Device\\NPF_{...}
```

Most importantly, the previous errors:

```text
No codec found for data link type 0
No codec for DAQ base protocol, stopping packet processing
```

were no longer produced.

This demonstrates that Snort can now initialize its packet-processing path on the Windows Npcap Loopback Adapter.

---

# Running Snort on Windows

The pcap DAQ module can be selected explicitly with:

```bash
./build/src/snort.exe \
  --daq-dir /path/to/snort/install/lib/snort/daq \
  --daq pcap \
  -i '<Npcap interface name>'
```

For the Npcap Loopback Adapter, the interface name has the form:

```text
\\Device\\NPF_{<interface-guid>}
```

The GUID is machine-specific and should not be committed to documentation.

Snort reports the selected DAQ interface with a line similar to:

```text
++ [0] \\Device\\NPF_{...}
```

---

# SCADA/DNP3 Loopback Test

The SCADA laboratory uses a TMW DNP3 test application and a local DNP3 application.

The observed architecture is:

```text
TMW Test Harness
    tmwtest.exe
        |
        | TCP / DNP3
        | 127.0.0.1:22000
        v
PAYA_DNP3.exe
```

The Windows networking state confirmed an established loopback TCP connection between the applications.

The Snort integration was then extended specifically to support capture of this loopback path.

### Current validation status

The following has been confirmed:

- Npcap Loopback Adapter installed successfully.
- Snort can open the Npcap Loopback Adapter.
- LibDAQ pcap starts successfully against the loopback interface.
- Snort recognizes and processes `DLT_NULL` after the custom codec is registered.
- The previous DLT 0 codec error is resolved.

The next validation step is to generate DNP3 traffic while Snort is running and confirm that the resulting packets are captured and decoded through the complete inspection path.

That final DNP3 live-traffic validation should be recorded separately from the already-confirmed loopback initialization result.

---

# Validation Results

| Test | Result |
|---|---|
| Native Windows build | PASS |
| Snort 3.12.2.0 executable | PASS |
| LibDAQ 3.0.27 build | PASS |
| DAQ pcap module available | PASS |
| Npcap 1.88 installed | PASS |
| Npcap driver running | PASS |
| Npcap Loopback Adapter installed | PASS |
| Snort opens loopback interface | PASS |
| DLT_NULL codec compiled | PASS |
| DLT_NULL codec statically registered | PASS |
| Snort packet-processing initialization on DLT 0 | PASS |
| SCADA/DNP3 loopback path identified | PASS |
| End-to-end DNP3 packet inspection | PENDING |

The distinction between `PASS` and `PENDING` is intentional. A successful interface initialization is not the same as proving that a specific DNP3 packet was captured, decoded, and matched by a Snort rule.

---

# Troubleshooting

## `No codec found for data link type 0`

This error occurs when Snort receives `DLT_NULL` but has no registered codec for it.

For the Windows Npcap Loopback Adapter, verify that:

1. `cd_null.cc` exists under `src/codecs/root/`.
2. `cd_null.cc` is included in `src/codecs/root/CMakeLists.txt`.
3. `cd_null[]` is declared in `src/codecs/codec_api.cc`.
4. `PluginManager::load_plugins(cd_null);` is present in `load_codecs()`.
5. Snort is rebuilt after the changes.

The required registration path is:

```text
cd_null.cc
    |
    v
root_codecs
    |
    v
Snort executable
    |
    v
codec_api.cc
    |
    v
PluginManager::load_plugins(cd_null)
    |
    v
CodecManager
    |
    v
DLT_NULL = 0
```

## `Error opening adapter ... (123)`

Do not use unsupported DAQ variables such as `show_interfaces` with the pcap DAQ. The interface should be passed through Snort's `-i` option using the Npcap interface name.

## Loopback versus Ethernet capture

Do not assume that loopback traffic has an Ethernet header. Npcap loopback capture uses `DLT_NULL`, so an Ethernet codec cannot decode the packet directly.

This is why adding a DLT_NULL codec was necessary for the local SCADA topology.

---

# Current Limitations

This project should be considered Windows integration and portability work rather than a claim of complete Windows feature parity with every Unix-like Snort deployment.

The current SCADA validation also has an explicit boundary: loopback capture initialization has been verified, but complete DNP3 packet detection still requires a live traffic test with the TMW/PAYA applications active.

Some functionality remains inherently platform-specific, including Unix-domain sockets, Unix-only transports, Unix socket logging, and other POSIX-dependent components.

The DLT_NULL codec currently targets the BSD/Npcap loopback format used by the tested Windows environment. It should not be treated as a universal replacement for every possible loopback data-link format without additional validation.

---

# Upstream Status

## Snort

Pull Request #478:

https://github.com/snort3/snort3/pull/478

Status:

```text
Open / not merged
```

Branch:

```text
Mahbodbe:windows-support
```

## LibDAQ

Pull Request #43:

https://github.com/snort3/libdaq/pull/43

Status:

```text
Open / not merged
```

Branch:

```text
Mahbodbe:windows-support
```

The DLT_NULL loopback codec described in this README is an additional integration change made in the local Snort source tree for the Windows/Npcap loopback SCADA test path.

It should therefore be kept conceptually separate from the broader Windows-support pull request until it is independently reviewed and, if appropriate, proposed upstream.

---

# AI-Assisted Development

Parts of the development and debugging process were performed with assistance from AI tools.

AI assistance was used for tasks including:

- analyzing Windows/POSIX portability issues
- interpreting compiler and linker errors
- investigating DAQ behavior
- tracing Snort's codec registration architecture
- diagnosing the Npcap loopback `DLT_NULL` failure
- designing and reviewing the DLT_NULL codec
- checking CMake/static-plugin integration
- documenting the reproducible build and validation process

The resulting changes were manually inspected, compiled, and tested in the target Windows environment. AI assistance was therefore a development aid rather than the sole basis for accepting the implementation.

---

# Reproducibility

The repository avoids publishing machine-specific information such as:

- private IP addresses
- interface GUIDs
- Windows usernames
- personal filesystem paths

Use placeholders such as:

```text
<installation-prefix>
<Npcap interface name>
<interface-guid>
```

The important software revisions are explicitly documented:

```text
Snort 3.12.2.0
LibDAQ 3.0.27
Npcap 1.88
GCC 16.2.0
MSYS2 UCRT64
```

The repository is intended to allow another Windows system to reproduce the build and then adapt the interface name and paths to its own environment.

---

# License

This repository contains references to and submodules from the upstream Snort and LibDAQ projects. Their respective licenses apply to their source code.

See the upstream repositories for authoritative licensing information:

- Snort 3: https://github.com/snort3/snort3
- LibDAQ: https://github.com/snort3/libdaq

Additional scripts and documentation introduced by this repository are subject to the repository's chosen licensing terms.

---

# References

- Snort 3: https://github.com/snort3/snort3
- LibDAQ: https://github.com/snort3/libdaq
- Snort Windows Support PR #478: https://github.com/snort3/snort3/pull/478
- LibDAQ Windows Support PR #43: https://github.com/snort3/libdaq/pull/43
- Integration repository: https://github.com/Mahbodbe/snort3-windows
- Npcap: https://npcap.com/
