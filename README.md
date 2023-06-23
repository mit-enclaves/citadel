# Citadel

We present Citadel, to our knowledge, the first side-channel-resistant enclave platform to run realistic secure programs on a speculative out-of-order multicore processor.

This repository contains pointers to the hardware and software infrastructure required to run our secure enclaves.
Our multicore processor runs on an FPGA and boots untrusted Linux from which users can securely launch and interact with enclaves.
We open-source our end-to-end hardware and software infrastructure, hoping to spark more research and bridge the gap between conceptual proposals and FPGA prototypes.

## Software Infrastructure

### [Security Monitor](https://github.com/mit-enclaves/security_monitor/)

The Security Monitor (SM) is a small (~9K LOC), trusted piece of software running at a higher privilege mode than the hypervisor or the OS.
Its role is to link low-level invariants exposed by the hardware (e.g., if a specific memory access is authorized), and the high-level security policies defined by the platform (e.g., which enclave is currently executing on which core and which memory region does it own).

### [Linux Building Tools And Kernel Module](https://github.com/mit-enclaves/linux)

This repository contains tools and scripts to compile Linux to run on Citadel. It also contains the code for the SM Kernel Module to enable Linux to interact with the Security Monitor and allow applications to launch secure enclaves.

### [RISC-V Toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)

To build the Security Monitor, compile Linux or enclaves, you will first need a functioning riscv toolchain.
Building the toolchain yourself will allow you to target the exact architecture rv64imafd used by the hardware (for instance, to avoid compiling with compressed instruction) and will ensure the generated binaries can run on the Riscy-OO processor.
The toolchain can be found on the [official repo](https://github.com/riscv-collab/riscv-gnu-toolchain).
Clone the repository with the submodules and install the dependencies (see the repository's `README` for more detailed instructions).
Run the following instruction to target the right architecture

```
$ ./configure --prefix=/opt/riscv_linux/ --with-arch=rv64imafd
```

And then build both the newlib and linux target by running (with optional -j flag to build on multiprocessor machines)

```
$ make
$ make linux
```

For easier debugging, add `GDB_TARGET_FLAGS_EXTRA := --enable-tui` to the `Makefile.in` file.

### [Citadel QEMU Target](https://github.com/mit-enclaves/qemu-sanctum/tree/riscy-ooo)

To debug software, tou might want to install QEMU with the special target that emulates the Riscy-OO processor. 
Building instructions can be found in the repository.

### [Secure Bootloader](https://github.com/mit-enclaves/secure_bootloader)

The Secure Bootloader is the first piece of code to run from the root-of-trust read-only-memory when the machine boots. 
It is trusted and will measure and attest the Security Monitor as part of the attestation mechanism.

## Enclave Repositories

We provide a series of example enclaves running on Citadel.

### [Crypto Enclave](https://github.com/mit-enclaves/crypto_enclave)

This enclave is a wrapper around an [ED2556 RISC-V library](https://github.com/mit-enclaves/ed25519).
The wrapper makes it possible to keep the keys used by the library inside of the enclave and for the library functions to be accessed in a Remote Procedure Call style (RPC).
We add a queue to shared memory so untrusted applications can send requests to the library to generate keys (that will stay in enclave memory), or to sign messages using previously generated keys.

### [Micropython Enclave](https://github.com/mit-enclaves/micropython)

We do a minimal port of [MicroPython](https://github.com/micropython/micropython) inside one of our enclaves.
This repository contains the port it-self while the [MicroPython Testbench](https://github.com/mit-enclaves/micropython-testbench) repository contains a testbench to test the enclave and to debug it.

### [CoreMark Enclave](https://github.com/mit-enclaves/coremark_enclave)

Simple port of the [CoreMark benchmark](https://github.com/eembc/coremark) to run inside of a Citadel enclave.

### [Enclave Skeleton](https://github.com/mit-enclaves/enclave_skeleton)

Simple infrastructure to write, run and debug an enclave. Great place to start if you want to try to run your own.

## Hardware Infrastructure

### [Riscy-OO and MI6](https://github.com/csail-csg/riscy-OOO)

Citadel is based on Riscy-OO and MI6, a fork of Riscy-OO that implement and evaluates several hardware security mechanisms.
Citadel introduces several new hardware mechanisms such as secure shared memory, DMA chanel partitioning, dynamic LLC partitioning and other mechanisms that relax the strict security policy of MI6

We also implement several mechanisms that were described in MI6 but only evaluated using models such as MSHR partitioning and LLC arbiter partitioning.

We will release the Citadel hardware changes soon.

### [Hardware Tests](https://github.com/mit-enclaves/hardware_tests)

This repository contains simple tests to verify the implementation correctness of the Citadel memory isolation mechanisms introduced by Sanctum.