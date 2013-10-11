---
layout: post
title:  "Toolchain: Kinetis KL25 development"
categories: toolchain development kl25 kinetis
---

I've recently selected the Kinetis KL25 as my go-to microcontroller. I can buy it cheaply direct from Freescale, and it has an incredible collection of peripherals for the cost. This necessitated the search for a new toolchain. I have a bad habit of reinventing the wheel with most things, so I started looking for projects with peripheral libraries. After a relatively short search, I had found three potentials.

<!--excerpt-->

### Alternatives
#### mbed
[mbed][mbed] looks to have the biggest ecosystem of the three, and the standard library is designed to make everything very easy. It has a target for the 128kB version of the KL25, which was easy to modify for the 64kB version I'm using (see below). In the end I felt mbed was a bit too heavy, I might come back to it if the other alternatives don't pan out.

#### NuttX
[NuttX][nuttx] is the choice I'm trying first up. It has the least pretty homepage of the three, but the feature list is strong, and it has a large number of periphial libraries. Most importantly, it has a porting guide.

#### Contiki
[Contiki][contiki] has always been a platform that I've wanted to use. It is event driven with protothreads, which is how I would like to write my programs. In the end I didn't choose it because working out how to port it was more effort than NuttX and the library of peripherals looked to be focused for wireless sensor projects.

Clearly this was just from a quick review, I'm probably wrong on a number of points and I might have to come back and try a different project if NuttX doesn't work out.

### Setting up the NuttX toolchain
The instructions below are all for Ubuntu 13.04. I'd like to change my development to box to Arch, but, much like going to the dentist, I'm not looking forward to actually doing it.

#### Install compilers and dependencies
{% highlight sh %}
sudo add-apt-repository ppa:terry.guo/gcc-arm-embedded
sudo apt-get update
sudo apt-get install -y gcc-arm-none-eabi clang gperf libncurses5-dev
{% endhighlight %}

#### Prepare NuttX
I grabbed the NuttX git repo and had a search through. The first thing I had to do was modify the KL2x arch files to include the 64kB chip instead of just the 128kB one. I've put these changes in their own [branch][MKL25Z64]. With the basics out of the way, I created a folder for the board configuration using the freedom-kl25z as a template. As an example, these are the steps for my 'operon' config.

{% highlight sh %}
git clone -b operon git@github.com:bharrisau/NuttX.git
# Build mconf if you don't already have it
cd NuttX/misc/tools/kconfig-frontends
./configure --enable-mconf --disable-gconf --disable-qconf
make
sudo make install
sudo ldconfig
cd ../../..
# Once mconf is built and installed
cd nuttx/tools
./configure operon/default
cd ..
make menuconfig
# System Type -> Toolchain Selection -> Generic GNU EABI
# Check through the options, exit and save
make
{% endhighlight %}

#### Next steps
I need to confirm that the build files are correct. From first inspection, the vector 0 (initial stack pointer) is pointing to the wrong place. It should be the top of the RAM (0x2000 1800) but is at 0x2000 0234. The program counter start is at 0x0411, but I've set the start of flash at 0x0410 (there might just be 1 byte of dummy or something).

My target board should be arriving soon, so I'll have something to test with.

### P.S. building a custom mbed library
For anyone looking how to modify mbed for a smaller KL25 version, this is what I did.

Update MKL25Z4.ld for 64kB flash/8kB RAM. 1/4 of the RAM is before 0x20000000, and mbed wants me to not use the first 0xC0 of it.

{% highlight c %}
MEMORY
{
  VECTORS (rx) : ORIGIN = 0x00000000, LENGTH = 0x00000400
  FLASH_PROTECTION  (rx) : ORIGIN = 0x00000400, LENGTH = 0x00000010
  FLASH (rx) : ORIGIN = 0x00000410, LENGTH = 64K - 0x00000410
  RAM (rwx) : ORIGIN = 0x1FFFF8C0, LENGTH = 8K - 0xC0
}
{% endhighlight %}

Update system_MKL25Z4.c to use internal system clock.

{% highlight c %}
#define CLOCK_SETUP     0
{% endhighlight %}

Build the library.
{% highlight sh %}
python workspace_tools/build.py -m KL25Z -t GCC_ARM
{% endhighlight %}

[nuttx]:    http://nuttx.org/
[mbed]:     http://mbed.org/
[contiki]:  http://www.contiki-os.org/
[MKL25Z64]: https://github.com/bharrisau/NuttX/tree/MKL25Z64



