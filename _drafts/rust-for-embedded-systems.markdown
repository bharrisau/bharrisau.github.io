---
layout: post
title:  "Rust for embedded systems"
categories: rust toolchain embedded arm
description: "Steps to set up a Rust environment that builds binaries for ARM micro-controllers"
---

I'm still looking for a development toolkit I'm happy with for embedded systems. But I've stumbled upon [Rust][rust-lang] while looking at the upcoming LinuxConfAU sessions and it looks nice. It is ahead-of-time compiled with LLVM, has a nice type system, lightweight tasks (actor like), and can be used without the runtime/standard library. Rust is still under heavy development, but I was able to set up a build environment to produce ARM Cortex-M0 binaries.

<!--excerpt-->

### Setting up Rust
To get the basic Rust system going, I followed the [installation instructions][rust-install]. Following the lead of [Svetoslav Neykov][armboot], when building with the [gcc-arm-embedded toolchain][gcc-arm-embedded] the process is to use rustc to generate LLVM IR, then use llc to generate ARM thumb assembly, finally using arm-none-eabi-gcc to finish the build. On Ubuntu, I found that I needed a newer LLVM-3.4 than was in the Saucy repos to handle the rustc output, so I added the [llvm.org hosted repositories][llvm-apt].

{% highlight sh %}
sudo -s
wget -O - http://llvm.org/apt/llvm-snapshot.gpg.key | apt-key add -
echo deb http://llvm.org/apt/saucy/ llvm-toolchain-saucy main > /etc/apt/sources.list.d/llvm-saucy.list
echo deb-src http://llvm.org/apt/saucy/ llvm-toolchain-saucy main >> /etc/apt/sources.list.d/llvm-saucy.list
apt-get update
apt-get install llvm-3.4 clang-3.4
exit
{% endhighlight %}

With this done I could build a project with _start defined in Rust. The flash and RAM use of libc was a bit high, 4K and 2K respectively. To remedy this for small systems, use the -specs=nano.specs flag when linking, this drops flash to 1K and RAM to 100 bytes (depends on what libc functions you are using).

I'm using [rust-core][rust-core] to help using Rust in a standalone fashion. I needed to add ARM as a platform (at the moment I've just aliased it to x86).

### Basic example
I've got a basic project working [here][rustup-operon]. Next step is to modify it into a blinky example and check it actually works.

### Future work
In the near future I want to fix up _start so it has its own section definition. I want to be able to define its location in flash and at the moment it is just generic .data. I'll also need to tackle the best way to handle the micro-controller registers, Rust doesn't have great support for 'volatile' yet and I'll need to import the C #defines into Rust macros.

[rust-lang]:      http://www.rust-lang.org/
[rust-install]:   http://static.rust-lang.org/doc/master/tutorial.html#getting-started
[rust-core]:      https://github.com/thestinger/rust-core
[llvm-apt]:       http://llvm.org/apt/
[gcc-arm-embedded]: https://launchpad.net/gcc-arm-embedded
[armboot]:        https://github.com/neykov/armboot
[rustup-operon]:  https://github.com/bharrisau/rustup/tree/operon
