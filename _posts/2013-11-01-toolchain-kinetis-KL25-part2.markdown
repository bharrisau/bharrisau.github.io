---
layout: post
title:  "Toolchain: Kinetis KL25 development - Part 2"
categories: toolchain development kl25 kinetis
description: "Selecting and installing a development environment for Freescale's Kinetis KL25: part 2"
---

After heaps of messing about, I have a basic toolchain for the KL25 (and hopefully any Kinetis series chip). I couldn't find out the licence for the very hard to find Freescale headers, so I've set up Github repositories to hold the BSD licensed [ARM CMSIS Headers][cmsis] and my own BSD licensed [Kinetis CMSIS headers][freescale]. My own project, Operon, has an [example][operon] of how the two repositories are used. I've utilised Simon Schubert's [SWD interface][mchck] for programming and debugging.

<!--excerpt-->

The Kinetic CMSIS headers are obviously a work in progress. I hope Freescale have kept the peripheral modules mostly the same between devices so that I can use the same structs and just change the memory map. I also need to find a better linker script to position the code in the flash. Or maybe I'll modify Simon's scripts to read srec files instead of binaries.

Pull requests for new devices or filling the gaps in my peripherals would be great.

[cmsis]:    https://github.com/bharrisau/CMSIS
[freescale]: https://github.com/bharrisau/cmsis-freescale
[operon]:   https://github.com/bharrisau/operon/tree/master/firmware
[mchck]:    https://github.com/mchck/mchck



