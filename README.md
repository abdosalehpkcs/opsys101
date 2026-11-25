# Memory Management – Part 2

**Theory Exercises V: Memory management. Memory allocation algorithms in multiprocess systems.**

This lab covers Problems 6–10. Problems 1–5 are in Part 1, on the branch `lab9-memory-allocation-1`.

## Introduction

In Part 1 we met paging. Memory is cut into pages and frames, and a page table says where each page is stored. Paging has no external fragmentation. But paging has its own price: the page table can be very long, and every address that a program uses must be translated.

In this lab we calculate this price. Again we do not write code. The problems come from the course slides [4].

| # | Question | Topic |
|---|----------|-------|
| 6 | How much CPU time does loading the page table take? | Page table in the hardware |
| 7 | How does a two-level page table split an address? | Two-level page tables |
| 8 | How much faster is a program with twice as many frames? | Page faults |
| 9 | What hit rate does the associative memory need? | Associative memory |
| 10 | How long does it take to read a program from disk? | Reading pages from a disk |

You need only basic maths: percentages, powers such as 2¹⁰, and a calculator.

### How each problem is organised

1. **The problem** – the text of the task.
2. **Understand the situation** – what is happening, in plain words and pictures.
3. **Solution, step by step** – the calculation with an explanation of every step.
4. **Answer** – the final result.

Numbers in square brackets, for example [1], point to the [References](#references) at the end.

## Key Terms

Read these terms first.

### Terms from Part 1

- **Logical address** – an address that the program uses. **Physical address** – the real place in the memory hardware [1, §13.3].
- **Page** – a block of the logical memory of a process. **Frame** – a block of the physical memory. A page and a frame have the same size, and each page is stored in one frame [1, ch. 18].
- **Page table** – a table that the OS keeps for every process. For each page it says in which frame the page is stored [1, §18.1].
- **Page number and offset** – an address has two parts. The page number says *which page*. The offset says *which place inside that page* [1, §18.1].

### New terms

- **32-bit address space** – every address has 32 bits. So a process can use 2³² different addresses [1, §18.2].
- **Page table entry** – one line of the page table. It holds the frame number of one page [1, §18.3]. The page table has one entry for every page.
- **Process switch** (also called *context switch*) – the OS stops running one process and starts running another one [1, §6.3].

### Units

| Unit | Meaning in this lab |
|------|---------------------|
| 1 kB | 1024 bytes = 2¹⁰ bytes |
| 1 ms (millisecond) | 0.001 s |
| 1 ns (nanosecond) | 0.000 000 001 s, so 1 ms = 1 000 000 ns |

Be careful with **kB**. In the official standard, 1 kB is 1000 bytes, and the name for 1024 bytes is *kibibyte* (KiB) [2], [3]. Our problems write kB for 1024 bytes.

### Powers of two

With **b** bits we can write 2ᵇ different numbers.

| Bits | 10 | 13 | 19 | 32 |
|------|----|----|----|----|
| Different numbers | 1024 | 8192 | 524 288 | 4 294 967 296 |

---

## References

1. R. H. Arpaci-Dusseau, A. C. Arpaci-Dusseau. *Operating Systems: Three Easy Pieces*, version 1.10. Arpaci-Dusseau Books. Free online: <https://pages.cs.wisc.edu/~remzi/OSTEP/>. Chapters used: [6 – Mechanism: Limited Direct Execution](https://pages.cs.wisc.edu/~remzi/OSTEP/cpu-mechanisms.pdf), [13 – The Abstraction: Address Spaces](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-intro.pdf), [18 – Paging: Introduction](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-paging.pdf), [19 – Paging: Faster Translations (TLBs)](https://pages.cs.wisc.edu/~remzi/OSTEP/vm-tlbs.pdf).
2. National Institute of Standards and Technology (NIST). *Prefixes for binary multiples*. <https://physics.nist.gov/cuu/Units/binary.html>.
3. National Institute of Standards and Technology (NIST). *Metric (SI) Prefixes*. <https://www.nist.gov/pml/owm/metric-si-prefixes>.
4. Course slides: *Operačné systémy – Teoretické cvičenia V. Správa pamäti. Algoritmy prideľovania pamäti v multiprocesových systémoch.* The problems and the solution methods in this lab come from these slides and their speaker notes.
