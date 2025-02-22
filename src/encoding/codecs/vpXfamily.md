# Encoding for vp9/av1

The vpX/aom family of codecs (vp8, vp9, and av1) are the standard for
royalty-free video encoding and have been for the past 10 years.
The vpX/aom family of codecs is widely supported for playback
in all modern browsers (excluding safari, but with apple's entry
into the alliance for open media, this is expected to change)
and many hardware devices such as gaming consoles and phones.
The second and third generation of vpX/aom family codecs
(vp9 and av1) provide better video quality at smaller file sizes
than h.264. However, av1's encoders have yet to reach the
same maturity as x264 or x265,
so we cannot yet draw a comparison between av1 and its competitor h.265
on detail retention.

libvpx is a mature, feature-complete, free, open-source encoder
for the vp8/vp9 video formats, providing maximum
compression for vp9.

SVT-VP9 is a non-feature-complete, speed-optimized, free,
open-source encoder for the vp9 video format, which provides
high-speed, highly parallel encoding.

libaom is a feature-complete, free, open-source encoder
for the av1 format and future formats in the family,
such as the av2 format (currently in early development).
On average, libaom provides 20% better compression than h.265
but can sometimes struggle with a loss of detail
due to lack of maturity.

SVT-AV1 is a non-feature-complete, speed-optimized, free,
open-source encoder for the av1 video format, which provides
high-speed, highly parallel encoding.
By aiming to provide lower levels of compression at a higher speed
it has reached a higher level of maturity quicker than libaom.


## Prerequisites

To get started, you'll need a few things:

- A video to encode—for the examples,
  use a yuv file generated by VapourSynth,
  which you should be able to do
  if you've been following the previous sections of this guide.
- Either libvpx and libaom or SVT-VP9 and SVT-AV1

Here's how we get a copy of these encoders:


### Windows

libaom (git) may be built from source by downloading the sources using git-bash as such:
```
git clone https://aomedia.googlesource.com/aom
mkdir build
cd build
cmake ../aom
make
```

SVT-AV1/SVT-VP9 can be found on Github

### Linux/macOS

Generally, libaom/libvpx or svt-av1/svt-vp9 will be available
through your distribution's package manager.
Here are a few examples:

#### libaom
- **Ubuntu/Debian**: `sudo apt install aom-tools`
- **Arch Linux**: `sudo pacman -S aom`
- **macOS**: `brew install aom`

#### SVT-AV1
- **Arch Linux**: `sudo pacman -S svt-av1`
- **Clear Linux**: `sudo swupd bundle-add SVT-AV1`

#### libvpx
- **Ubuntu/Debian**: `sudo apt install vpx-tools`
- **Arch Linux**: `sudo pacman -S libvpx`
- **macOS**: `brew install libvpx`

#### SVT-VP9
- **Arch Linux**: `sudo pacman -S svt-vp9`
- **Clear Linux**: `sudo swupd bundle-add SVT-VP9`

## Getting Started

All four encoders are highly configurable,
and the options might overwhelm you at first.
But we will provide you with some basic
parameters and guidance to start with for your
first encodes. The encoders in this family have
most of their options named similarly to each other,
so for simplicity's sake, we will use libvpx and
libaom for these examples.

### On the topic of patent and copyright.

The vpX/aom family of codecs are royalty-free,
meaning you do not have to pay to use technologies
required to design, distribute or use a vpX/aom encoder
or decoder. This differs from h.264 and h.265,
which require payment for the use of the technologies needed to
design an h.264 or h.265 encoder or decoder.

Unfortunately, because of this, vpX/aom family codecs
are best used in two-pass mode due to some technologies
required to provide a satisfactory single pass encoder
being subject to patent royalties imposed by the MPEG
alliance (the creators of h.264 and h.265). As such,
all examples in this guide are two-pass encodes.

### Example 1: General-Purpose Encoding with VP9

Open up a terminal window,
and navigate to the folder
where your VapourSynth script lives (for a 1080p or larger 16:9 video).
Let's run the following commands:

```
vspipe --y4m myvideo.vpy temp.y4m
vpxenc --codec=vp9 --passes=2 --pass=1 --fpf=passdata.bin --row-mt=1 --cpu-used=4 --deadline best --auto-alt-ref=2 --lag-in-frames=25 --arnr-strength=0 --aq-mode=2 --frame-parallel=0 --tile-columns=2 --tile-rows=1 --threads=8 --end-usage=vbr --cq-level=25 --ivf -o /dev/null temp.y4m
vpxenc --codec=vp9 --passes=2 --pass=2 --fpf=passdata.bin --row-mt=1 --cpu-used=0 --deadline best --auto-alt-ref=2 --lag-in-frames=25 --arnr-strength=0 --aq-mode=2 --frame-parallel=0 --tile-columns=2 --tile-rows=1 --threads=8 --end-usage=vbr --cq-level=25 --ivf -o myvideo.ivf temp.y4m
rm temp.y4m passdata.bin
```

Let's run through what each of these options means:


##### `vspipe --y4m myvideo.vpy temp.y4m`

This portion loads your VapourSynth script
and saves it to temp.y4m.

##### `--codec=vp9`

This tells vpxenc that we're going to
be encoding using the vp9 codec.


##### `--passes=2`

This tells vpxenc that we're doing two-pass encoding.


##### `--pass=X`

This tells vpxenc which pass we are performing.


##### `--fpf=passdata.bin`

This tells vpxenc where to store the statistics used for the second pass.


##### `--row-mt=1`

This tells vpxenc to multithread on tile rows, not just tile columns.
This is an important parameter because it greatly reduces encode time.


##### `--cpu-used=X`

This is like x264's presets.
It allows you to switch between faster encoding
or higher quality.
The full list of presets mapped to their h.264 equivalents is

0. placebo
1. veryslow
2. slower
3. slow
4. medium
5. fast
6. faster
7. veryfast
8. superfast
9. ultrafast

Unlike with h.264, 0 and 9 are reasonably useful if you want to get
maximum compression (and don't care about encoding time) or
maximum encoding speed (and don't care about getting good quality
(real-time video, for example)), respectively.

##### `--deadline best`

This instructs the encoder to be more strict about the encoding quality
and to not move on until it is 100% satisfied with that frame.


##### `--auto-alt-ref=2`

This is an option that determines how keyframes are chosen. If you
are tuning for maximum playback compatibility, choose 1; if you are
tuning for maximum quality, choose 2.


##### `--lag-in-frames=25`

This option allows keyframes to be moved forward or back by this
number of frames, allowing for better keyframe placement.
Larger numbers like 25 are better suited for anime.


##### `--arnr-strength=0`

Turn off keyframe denoising as you've already pre-processed the
video with vapoursynth.


##### `--aq-mode=2`

This option tells the encoder to rely on how complex
each frame's motion is to determine the adaptive quantization.
In other words, more motion in more directions equals more bits
devoted to that frame.


##### `--frame-parallel=0`

This option allows for parallel video decoding, reducing quality,
so we disable it.


##### `--tile-columns=2` and `--tile-rows=1`

These options determine how the encoder breaks up the frame into
chunks for encoding. The more chunks, the more threads can be used
for encoding. One important note here is that the values of this
parameter are in the form of \\(2^{x}\\), meaning that the number of
columns and rows with these values is actually 4 and 2,
respectively. Tiles cannot be smaller than 65536 pixels in area.


##### `--threads=8`

Self-explanatory. Set this to the number of threads your CPU has.


##### `--end-usage=vbr`

Set the encoder in VBR mode.


##### `--cq-level=25`

Set the average quantizer level.


##### `-o /dev/null temp.y4m` or `-o myvideo.ivf temp.y4m`

This last portion tells which files to use for the input and output.
We use `-o` to tell which filename to write the encoded file to.
In the first case, vpxenc will write a file at `/dev/null`,
the null device on POSIX systems (meaning no file will be written
for the first pass (if you're on windows, use the filename nul
instead)). In the second pass, we write an ivf to `myvideo.ivf` in the
current working directory.

The last argument we are passing to vpxenc is the input file.
As we are doing a two-pass encode, it is more efficient to save the
output of vapoursynth then run the encode on that instead of running
vapoursynth twice.


### Example 2: General-Purpose Encoding with AV1

TODO

## Recap

We covered the basics of how to encode using the vpX/aom family of
codecs, including speed presets.

Here is a summary of when to use vp9 vs. AV1 (please note these are
recommendations, you do not have to follow them):

- Is your source larger than Full HD?
	- Yes:
		- AV1
	- No:
		- Is your source Full HD Quality:
			- Yes:
				- Is time important?
					- Yes:
						- VP9
					- No:
						- AV1
			- No:
				- VP9
