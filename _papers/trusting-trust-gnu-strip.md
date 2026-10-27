---
title: "Trusting-Trust Attack against an Entire Linux Distribution through Binary Manipulation"
date: 2026-10-23
authors: "Julien Malka, Aman Sharma, Martin Monperrus, Stefano Zacchiroli, Théo Zimmermann"
paper_url: "https://arxiv.org/pdf/2607.24888"
---

## Trusting-trust attack

- Proposed by Ken Thompson
- A compromised complier compiles the source code of a binary that is to become the compiler's successor, the source code has no malicious logic
- During compliation, the compiler injects malicious logic into the new binary, producing a copy of itself as well
- When the new binary compiles a new generation of compiler, it propogates the same malicious logic

## Strip utility

- Removes symbols and other information from object files like symbol table (.symtab)

## Attack

- Append malicious bytes to the program
- Repurpouse a PT_NOTE in the ELF header to become PT_LOAD R+X for the malicious bytes
- Change ELF e_entry (program entry point) to the start of the malicious bytes
- Redirect ELF e_shoff to a new section table, calling the new payload .payload as a section (allowed as the program header of PT_LOAD R + X allows execution for this)
- ![Diagram of the attack in three stages: seed tampering, propagation through the NixOS bootstrap, and compromise]({{ '/assets/papers/trusting-trust-gnu-strip/figure-1.png' | relative_url }})
- Each generation of strip implants the payload into the next, running strip on all other binaries infects the whole distro

## Payload

- The payload runs before libc _start routine
- It saves registers the kernel gave to the process, aligns the stack and restores registers, it jumps to the original entry point so the process keeps everything else the same
- The kernel enters the process at the new entry point, the payload saves the kern registers and aligns the stack, it checks for a nix build, reads its own basename and only implants the file arguments when teh host is strip, then jumps back to original entry point

## Results

- Out of 3799 binaries, 3790 managed to print the infection marker "you've been pwned"
