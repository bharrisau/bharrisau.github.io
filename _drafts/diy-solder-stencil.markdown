---
layout: post
title:  "DIY solder stencils in Kicad"
categories: pcb solder assembly reflow kicad
---

I saw this [great guide][low power stencils] on making your own aluminium solder stencils from soda cans a few months ago and decided to give it a go for my [operon][operon] board. I won't rehash all of the steps, Felix has done a great job of that. I'll just cover the stuff I learnt while giving it a go.

<!--excerpt-->

### Toner transfer
Finding a reliable medium for toner transfer was a little difficult. Felix suggests transparency film or shelving vinyl. After trying a few different things I ended up using book contact from Woolworths (I think it is very similar to the shelving vinyl).

I also tried using laminator sleeves and magazine paper. Neither really worked.

I used the contact to cover most of an A4 sheet of paper, and fed that through the printer. Unfortunately, when the contact is heated, it shrinks, curling the paper and also shrinking the stencil size. In the future I am going to try using a smaller amount of the contact, and preshrinking it a couple of times.

### Design preparation
I use Kicad for all my designs, so these are the steps I used to produce the stencil design.

#### Shrink the solder mask
The Low Power Labs article suggests shrinking the paste apertures, I used -10% but I might experiment with different values in the future. To change the global shrinkage go to 'Dimensions' -> 'Pads Mask Clearance' and set either 'Solder Paste Clearance' or 'Solder Paste Ratio Clearance'.

I used my tool [edalm][edalm] to produce all my footprints, and it splits large thermal pads into smaller ones by default. The standard shrinkage on these pads is -12.5%, but when I etched the design it became a single large pad. In the future I will be setting the thermal pad shrinkage to -25%.

#### Correct paste layer
You may need to go through and disable the paste layer on some pads; such as through hole components you will be soldering manually. To do this select the 'Edit' option for the pad and untick the F.Paste layer.

#### Create the SVG
I created the SVG by plotting to gerber and using gerbv to modify and export as SVG. My modifications were just the addition of [stencil8][stencil8] peg holes based on using my [stencil guides][stencil guide]. I'll be uploading the overlays once I've confirmed they all work.

#### Create the PDF
I use Inkscape to convert the SVG to a PDF. The steps are simple:

1. Draw black box covering the design (as we need the stencil to be a negative)
2. Send the box to the bottom: Object -> Lower to Bottom
3. Ungroup the stencil object: Object -> Ungroup
4. Convert the stencil to a single path: Path -> Union
5. Shift select the box so everything is selected, then: Path -> Difference
6. Flip the final object horizontally (this way the silver side is up)
7. File -> Save a copy. Save as PDF

### Preparing the aluminium
I had a hard time removing the inner coating on the can with just a paper towel and acetone. I moved up to using a green pot scrubber, but that left the aluminium with scratches. I'll have to try different cans and see if I can improve.

### Results
The rest of my process was the same as the Low Power Labs article. The stencil wasn't usable because of the shrinkage and the over etching of the thermal pads.

[low power stencils]:    http://lowpowerlab.com/blog/2013/02/11/diy-smd-metal-stencils-the-definitive-tutorial/
[operon]:   http://operon.bharr.is
[edalm]:    https://github.com/bharrisau/edalm
[stencil guide]: https://github.com/bharrisau/spacer_board
[stencil8]: http://www.hoektronics.com/2012/10/27/super-simple-smt-stencil8/