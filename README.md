# SHA-Intrinsics

This GitHub repository contains source code for SHA-1, SHA-224, SHA-256 and SHA-512 compress function using Intel SHA and ARMv8 SHA intrinsics, and Power8 built-ins. The source files should be portable across toolchains which support the Intel and ARMv8 SHA extensions.

Only the SHA-1, SHA-224, SHA-256 and SHA-512 compression functions are provided. The functions operate on full blocks. Users must set initial state, and users must pad the last block. The small sample program included with each source file does both on an empty message.

## Intel SHA

To compile the x86 sources on an Intel machine, be sure your CFLAGS include `-msse4 -msha`.

The x86 source files are based on code from Intel, and code by Sean Gulley for the miTLS project. You can find the miTLS GitHub at http://github.com/mitls.

If you want to test the programs but don't have a capable machine on hand, then you can use the Intel Software Development Emulator. You can find it at http://software.intel.com/en-us/articles/intel-software-development-emulator.

## ARM SHA

To compile the ARM sources on an ARMv8 machine, be sure your CFLAGS include `-march=armv8-a+crc+crypto`. Apple iOS CFLAGS should include `-arch arm64` and a system root like `-isysroot  /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS8.2.sdk`.

The ARM source files are based on code from ARM, and code by Johannes Schneiders, Skip Hovsmith and Barry O'Rourke for the mbedTLS project. You can find the mbedTLS GitHub at http://github.com/ARMmbed/mbedtls. Prior to ARM's implementation, Critical Blue provided the source code and pull request at http://github.com/CriticalBlue/mbedtls.

If you want to test the programs but don't have a capable machine on hand, then you can use the ARM  Fixed Virtual Platforms. You can find it at https://developer.arm.com/products/system-design/fixed-virtual-platforms.

## Power8 SHA

The Power8 source files are a work in progress. The problem at the moment is speed. The SHA-256 implementation using Power8 built-ins is 25% slower than C++ on big-endian so it is not suitable for production.

One of the problems we are having is IBM's documentation sucks. Namely, there is none. We kind of knew that going into things because we experienced the same problems when implementaing AES on Power8. Another problem is, IBM does not work with developers and answer questions. We emailed the IBM team about the performance issues and asked for guidance. The IBM team did not respond.

We found some problems with the code generated by GCC 4.8.5, GCC 7.2.0 and IBM XL C/C++ 13.1, but the bug reports were closed as "already fixed" in GCC 6.4 and above. Also see [Issue 84753 - GCC does not fold xxswapd followed by vperm](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=84753).

We can't switch to inline ASM because IBM XL/C++ does not support it well. Our research indicates it is available for XL C/C++ for Z/OS, but not other systems. Also see [Accelerating performance with inline assembly using IBM XL C/C++ compiler on IBM z Systems](http://www.ibm.com/developerworks/library/l-inline-assembly-using-ibm-xl-c-cpp/).

Below are the numbers we are witnessing for SHA-256. Performance testing of SHA-512 has not started. The numbers are not that impressive. Even OpenSSL's numbers are relatively bad. In contrast, Intel SHA-256 runs around 3.7 cpb, and ARM SHA-256 runs around 2.7 cpb.

### GCC112, ppc64-le, 3.2 GHz, SHA-256

|  Impl  |   MiB/s   |  Cyc/byte  |
| ------ | --------- | ---------- |
|   C++  |    138    |    23.43   |
| Power8 |    152    |    21.28   |

### GCC119, ppc64-be, 4.1 GHz, SHA-256

|  Impl  |   MiB/s   |  Cyc/byte  |
| ------ | --------- | ---------- |
|   C++  |    385    |    10.16   |
| Power8 |    297    |    13.15   |

# Benchmarks

The speedups can be tricky to measure, but concrete numbers are availble from Jack Lloyd's Botan. The relative speedups using a three second benchmark under the command `./botan speed --msec=3000 SHA-1 SHA-224 SHA-256` are as follows. The measurements were taken from a Intel Celeron J3455, and an ARMv8 LeMaker HiKey.

## Intel SHA

The following tests were run on a machine with a Celeron J3455 at 1.5 GHz (burst at 2.2 GHz). The machine has an ASUS J3455M-E motherboard, which was one of the first Goldmont's available.

### GCC 7.3.1

```
$ ./botan speed --msec=3000 SHA-1 SHA-224 SHA-256 SHA-384 SHA-512
SHA-160 hash buffer size 1024 bytes: 887.435 MiB/sec 1.61 cycles/byte (2662.31 MiB in 3000.00 ms)
SHA-224 hash buffer size 1024 bytes: 443.480 MiB/sec 3.22 cycles/byte (1330.44 MiB in 3000.00 ms)
SHA-256 hash buffer size 1024 bytes: 443.614 MiB/sec 3.22 cycles/byte (1330.84 MiB in 3000.00 ms)
SHA-384 hash buffer size 1024 bytes: 122.228 MiB/sec 11.69 cycles/byte (366.68 MiB in 3000.00 ms)
SHA-512 hash buffer size 1024 bytes: 119.733 MiB/sec 11.93 cycles/byte (359.20 MiB in 3000.00 ms)
```

### Clang 5.0.1

```
$ ./botan speed --msec=3000 SHA-1 SHA-224 SHA-256 SHA-384 SHA-512
SHA-160 hash buffer size 1024 bytes: 912.617 MiB/sec 1.57 cycles/byte (2737.85 MiB in 3000.00 ms)
SHA-224 hash buffer size 1024 bytes: 447.320 MiB/sec 3.20 cycles/byte (1341.96 MiB in 3000.00 ms)
SHA-256 hash buffer size 1024 bytes: 447.315 MiB/sec 3.20 cycles/byte (1341.94 MiB in 3000.00 ms)
SHA-384 hash buffer size 1024 bytes: 124.250 MiB/sec 11.50 cycles/byte (372.75 MiB in 3000.01 ms)
SHA-512 hash buffer size 1024 bytes: 124.248 MiB/sec 11.50 cycles/byte (372.74 MiB in 3000.00 ms)
```

## ARM SHA

The following tests were run on a AMD Opteron 1100 running at 2.0 GHz.

### GCC

```
> ./botan speed --msec=3000 SHA-1 SHA-224 SHA-256 SHA-384 SHA-512
SHA-160 hash buffer size 1024 bytes: 664.520 MiB/sec (1993.561 MiB in 3000.001 ms)
SHA-224 hash buffer size 1024 bytes: 599.788 MiB/sec (1799.363 MiB in 3000.000 ms)
SHA-256 hash buffer size 1024 bytes: 599.463 MiB/sec (1798.391 MiB in 3000.001 ms)
SHA-384 hash buffer size 1024 bytes: 188.142 MiB/sec (564.426 MiB in 3000.001 ms)
SHA-512 hash buffer size 1024 bytes: 188.017 MiB/sec (564.051 MiB in 3000.001 ms)
```

### Clang

To be determined.

## Power8 SHA

To be determined.