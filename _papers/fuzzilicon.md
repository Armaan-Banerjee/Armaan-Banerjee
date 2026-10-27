---
title: "Fuzzilicon: A Post-Silicon Microcode-Guided x86 CPU Fuzzer"
date: 2026-10-23
authors: "Johannes Lenzen, Mohamadreza Rostami, Lichao Wu, Ahmad-Reza Sadeghi"
paper_url: "https://www.ndss-symposium.org/wp-content/uploads/2026-s1486-paper.pdf"
---

## Abstract

- Modern CPUs are hard to understand and contain many microarchitecture flaws that evade most anaylsis
- They reverse engineered Intel's proprietary mircocode update interface, which allows them to get feedback directly from the processors' architecture
- They found 5 findings, including 2 speculative execution vulnerabilites, and were able to rediscover umicro Spectre vulnerabilities

## Introduction

- Flaws in CPU microachitecuttre are often used to leak data, bypass protections, or undermine integrity, the x86 family is particularly vulnerable being very complicated as well as its CISC->RISC architecture
- Hardware fuzzing has become an approach used to detect these flaws
- Pre-silicon fuzzing targets Register-Transfer Level (RTL) during development
- Post-Silicon fuzzing evaluates processors under execution
- Post-silicon fuzzing is much less explored, due to them targetting black box CPUS like Intel or AMD where interal microcode level behavious is often locked away.
- In this Fuzzilicon they use the microarchitectural feedback channel to guide test generation, and by running the x86 CPU in Red-unlocked mode and using undocumented debugging and instruction in intel cpus, they gained access to micrcode engine interface which they use to turn the CPU into a grey box
- They can view internal execution state without RTL or hadware using this technique
- They built a bare-metal hypervisor fuzzer which isolates the device, and controls its environment
- They introduced a serilisation oracle that generates equivalent instruction sequences

## Background

- The microcode engine is used for more complicated instructions, it expans the x86 instruction into a sequence of microoperations, and scheduled using a triad
- A triad contains 3 microoperations and a sequence word that controls branching, ordering and termination
- The ROM stores factory installled instruction handlers, wheras the RAM stores runtime patches, the micrcode engine first checks the RAm for patches before exeucting the ROM
- Intel CPUs support multiple debug access lelves, Green-locked is the default where all debugging features are disable, Orange-unnlocked its intended for OEMs and has a subset of debugging features, Red-unlocked is used internally by Intel and has unrestricted access to mircoarchitecture, 2 new instructions become availabe, udbgrd and bdbgwr which enable reading and writing internal CPU regions include microcode RAM
- This way, by writing microcde triads, and triggering as specific address, abritary mirccode can be executed
- Hardware fuzzing generates intruction sequences that are executed on the CPU and relies on 2 strategies for bug detection, assertion checking which is only possible when RTL is available, and differential testing which compares execution across different CPUs or configurations
- Post silicon testing is made harder by restricted visibility to CPU internal state

## Methodology

### Challenges

- Microarchitectural invisibility: x86 CPUs are closed source, so previous post-silicon fuzzers only had coarse feedback like register values or performance counters, which only show surface level behaviour
- No bug detection oracle: there is no formal specification of what the microarchitecture should do, and comparing outputs across different CPUs gives false positives (from legitimate architectural differences) and false negatives (when both CPUs have the same flaw)
- Fault containment: as all knowledge of Intel's microcode comes from reverse engineering, a bad microcode patch can hang or crash the CPU, which would take down the whole fuzzing setup
- Non-deterministic execution: instructions like RDRAND (random number), RDTSC (timestamp counter) and RDPMC (performance counter) give different results each run, and OS noise and leftover state from previous tests make results hard to reproduce

### Overview

- Split into 3 parts: the Fuzzer Controller which runs on a separate machine and generates and mutates test cases, the Fuzzer Agent which runs on the target CPU, installs the microcode instrumentation and runs the tests, and the Watchdog which hard resets the target if it crashes or hangs
- Each test case is run as is and as a serialised version in 2 separate VMs, if the final register state differs, a tracer replays the test one instruction at a time to find where they diverged
- The microcode coverage from each run is sent back to the controller to guide the next mutations
- Because the controller is on a different machine, crashes on the target do not lose any fuzzing state, so it can run for days or weeks without anyone touching it

### Microcode coverage

- In software fuzzing (eg AFL), coverage-guided fuzzing keeps inputs that reach new branches (edges) of the program and mutates them further, the problem is finding an equivalent for a CPU
- Architectural coverage (which instructions ran and what registers they changed) saturates early, as one x86 instruction can take many different paths inside its microcode depending on internal state
- Eg RDRAND can return a random number, raise an invalid opcode exception (#UD) if used with a LOCK prefix, or exit to System Management Mode (SMM) or the hypervisor, but architectural coverage counts it as covered after the first time it runs
- Microcode coverage records which microcode addresses (ie which microoperations) are executed and how many times, so 2 inputs that use the same microoperations a different number of times (eg a loop running once vs 4 times) are counted as different, which previous work ignored
- This plays the role that edge coverage plays in software fuzzing

### Microcode instrumentation

- The CPU has a hook table (the patch registers) which maps a microcode address in ROM to a patch in RAM, so when the source address is fetched execution jumps to the patch instead
- Fuzzilicon uses this to insert a small probe at each address it wants to monitor, which increments a counter in normal memory and then jumps back to the original microcode
- To avoid changing the CPU's behaviour, the probe saves the few registers it uses to the CPU's staging buffer (the only known microoperation that can write a register to memory without needing another register), records the coverage, restores the registers, runs the original microoperation it replaced, then jumps to the next intended address
- They found that hooking an even address S to destination D also automatically redirects S+1 to D+1, so the patch starts with a jump at D that only runs when entered from S, letting them tell which of the 2 addresses triggered it
- Reconfiguring all 16 hooks would need 144 separate udbgwr writes, so they wrote a custom microcode routine that sets up the whole hook table in a single udbgwr call per test case

### Coverage collection

- The N3350 only has 16 hooks, each covering an even/odd pair, so only 32 addresses can be monitored at once
- The microcode address space is 0x7C00 (about 32k) addresses, so covering everything naively means running each test case 0x7C00 / 32 = 992 times
- Instead, they first only hook the entry points of x86 instructions, which are all below 0x1000 and aligned to 8, giving 512 entry points, so 512 / 16 = 32 runs
- They then use static analysis to split the microcode into basic blocks (a sequence of microoperations with one entry point where once the first one runs, all of them run), so hooking one address gives coverage for the whole block
- Only the blocks that can be reached by a conditional branch from a block that was actually hit are hooked in the next round, which cuts the overhead by up to 31 times

### Hypervisor

- A type-1 hypervisor runs directly on the hardware instead of on top of an OS, here it runs as a UEFI application using Intel VMX (Intel's hardware virtualisation)
- Each test case gets a fresh virtual CPU with the same starting registers, so every test starts from an identical state
- Using Extended Page Tables (hardware support for giving a VM its own view of memory), memory is split into an execute-only region for the test, a read-write region that is zeroed before every test, and a read-only region with structures x86 needs like the Global Descriptor Table (GDT) and Task State Segment (TSS)
- The VM exits on anything non-deterministic, any access to hardware, external interrupts, or when it runs out of time, which stops infinite loops from stalling the fuzzer
- Execute-only memory also stops tests from modifying their own code, which would break the serialisation oracle

### Input generation and mutation

- The starting corpus is either random bytes or valid instructions taken from libraries like libc++, libz and libzip and concatenated together
- Mutation uses either AFL's Havoc mutator (random bit flips, byte changes, insertions etc, from the LibAFL library) or their own genetic algorithm doing 1 to 8 byte mutations and crossover between the best samples, scored by microcode coverage and how much of the input actually ran
- Havoc performed better so it was used in the evaluation
- As the inputs can be random bytes, they can be invalid instructions too, which is still useful as the CPU's handling of invalid input is also microcode

### Serialisation oracle

- Inspired by Zenbleed, each test case P is compared to a version Q with a fence instruction (eg LFENCE, which waits for all previous instructions to finish before continuing) between every instruction
- Fences stop speculation and reordering, so on a correct CPU P and Q should end with the same architectural state, and any difference shows hidden microarchitectural behaviour without needing a reference model
- Inserting fences moves the code around and breaks instructions that use relative addressing, so the offsets of these are recalculated for the new layout
- As inputs are random bytes, jumps can land in the middle of other instructions, so they use superset disassembly (disassembling from every byte offset to find every possible instruction) to unroll the program into individual instructions which are relocated separately with fences between them

### Speculative microcode fuzzing

- Speculative execution is when the CPU runs instructions before it knows whether they should run (eg before a branch is resolved), and is supposed to throw away all their effects if it guessed wrong
- They built a microcode template using UJMPCC_DIRECT_NOTTAKEN_CONDNZ, a microcode branch that is always predicted not taken, so the microoperations after it always run speculatively before being discarded
- Each fuzzing iteration fills this speculative window with random or guided microoperations, and compares the architectural state before and after to find effects that survived the rollback

### Implementation

- Written in Rust and x86-64 assembly and open source
- Target is a Gigabyte mini PC with an Intel Celeron N3350 (Goldmont), which was Red-unlocked manually once using known procedures for this CPU family
- The controller and watchdog run on Raspberry Pi 4s, the watchdog acts as a remote keyboard and storage device for the target and can power cycle it using wires soldered to its motherboard
- The Fuzzer Agent runs as a standalone UEFI application with no OS to avoid interference, and talks to the controller over UDP

## Results

- F1: automatically rediscovered the µSpectre class, where statically predicted microcode branches let speculation continue across x86 instructions, which was previously only found manually, as the instrumentation itself created these branches and the fenced version diverged
- F2: speculative writes to the Control Register Bus (CRBUS) persist after rollback, eg writing to CRBUS address 0x692 can disable the hook table and so turn off all microcode patches including security fixes, and others cause CPU lockups (DoS)
- F3: speculative writes to the segment selector cache (eg setting the GDT base address) persist after rollback and bypass permission checks, which could allow privilege escalation to ring 0
- F4: speculation stops when it reaches an instruction that needs the microcode engine and the instruction cache is empty, which creates a timing difference an attacker could measure
- F5: some speculative microoperations still increment performance counters even though they are discarded, leaking internal state
- Coverage feedback reached the same coverage about 8 times faster than fuzzing without it, the valid corpus with feedback found the most unique addresses in total (2,528), and they reached 16.27% of hookable microcode addresses
- One fuzzing round took 18.274 ms compared to 566.490 ms for the previous instrumentation approach, about 31 times faster
