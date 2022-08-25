---
title: "HDR Images in the Browser Using JPEG XL in Q3 2022"
date: 2022-08-24T10:30:00-04:00
---

***NOTE**: this is about displaying greater-than-sRGB still images on computer displays, not the bracketing postprocessing technique that has been popular over the last decade or so*
![an orange cat staring at the camera](cat.jxl)
*An HDR JPEG XL image of my cat, Colby Jack. Requires JPEG XL enabled in your browser, will not display otherwise. Will look bad on SDR displays.*

## tl;dr

1. acquire a greater-than-sRGB source image, usually a camera raw file
2. generate an OpenEXR image using Affinity Photo
3. convert that OpenEXR image into a PQ-mapped PNG file
4. convert that PNG file to a JPEG XL file
5. enable JPEG-XL in Chrome
6. profit!

## Requirements

- A source image with dynamic range greater than SDR
- A computer with an HDR display. I used a 2021 MacBook Pro 16"
- Familiarity with the command line, operating systems, and compiling open source software

## Steps

### Capture

For an HDR photo to look good, you need an HDR source. The easiest way to do this is to capture a RAW photo. Basically any recent camera will do this, including smartphones. There are various proprietary manufacturer-specific formats for this (CR2, ARW, NEF, etc) but there's also the interoperable DNG format, which is what most (all?) smartphones will capture. Acquire a good photo in one of these formats.

### Artistic Expression

Next we need to turn that undeveloped RAW file into an image file. We also need to make sure that through the entire development process, we do not lose information as transform the bayer-patterned sensor data into a beutiful image. In order to do that, we need two things: a 32-bit floating point imaging pipeline, and a screen that can display it so we can visually confirm that the image looks as we want it to. By using Affinity Photo on a 2021 MacBook Pro with XDR display, we can have both. There are other options for hardware and software, but this is what I'll be using for the article (because that's what I have).

1. Set your MacBook to "HDR Video (P3-ST 2084)" in System Preferences in order to properly hit the 1000-nit peak brightness target. ![Apple screenshot of display preferences](https://support.apple.com/library/content/dam/edam/applecare/images/en_US/macos/monterey/macos-monterey-12-3-system-prefs-displays-multiple-presets-menu.png)

2. Enable HDR/EDR preview by default in Affinity photo with [the 32-bit Preview Panel](https://affinity.help/photo/en-US.lproj/pages/Panels/32bitPanel.html).

3. Follow this tutorial to develop your RAW photo into an HDR OpenEXR file: {{< youtube y86rUhitQGE >}}

4. Export your photo from Affinity as an OpenEXR 32-bit float file.

### Encoding

First and foremost: huge thanks to [Sami Boukortt](https://www.reddit.com/user/spider-mario/) for writing the tool we're about to use here! I could not have done this without you!

1. Now we're going to compile and use the in-development JPEG XL tools. I'm not sure if these build on macOS, I didn't try it. What I did do was open install Ubuntu on my Windows 11 computer. You can follow [these instructions](https://ubuntu.com/tutorials/install-ubuntu-on-wsl2-on-windows-11-with-gui-support).

2. Once you have an Ubuntu terminal, you can follow the compilation steps outlined in [the readme file for JPEG XL](https://gitlab.com/wg1/jpeg-xl/-/blob/main/README.md).
**Make sure you are using JPEG XL v0.8 or greater!** There's one important change though: **you need to also build the tools directory as well**. When it comes to the `cmake` configuration, use this command:

    ```bash
    cmake -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=OFF -DJPEGXL_ENABLE_DEVTOOLS=ON ..
    ```

    Make sure the config includes `exr_to_pq`.

    ```bash
    Building tools: cjxl;djxl;jxlinfo;cjpeg_hdr;fuzzer_corpus;butteraugli_main;decode_and_encode;display_to_hlg;exr_to_pq;pq_to_hlg;render_hlg;tone_map;texture_to_cube;generate_lut_template;ssimulacra_main;xyb_range;jxl_from_tree;benchmark_xl
    ```

3. You should now have binaries for `exr_to_pq` and `cjxl` in the `tools/` directory. We will execute them in sequence.

    ```bash
    $ ./exr_to_pq --luminance='max=1000' input.exr intermediate.png
    The resulting image should be compressed with --intensity_target=1372.76.
    ```

    Take note of the intensity target here. Mine was 1372.76 for the cat image.

    ```bash
     ./cjxl --intensity_target=1372.76 intermediate.png output.jxl
    ```

    Congratulations! You now have an HDR mapped JPEG XL file!

### Viewing

As of writing in late August 2022, JPEG XL is not enabled by default in Chrome. You can always see an up-to-date status report [here](https://caniuse.com/jpegxl), which also includes step by step instructions on how to enable the developer flag.

Once enabled, you should be able to drag-n-drop the JXL file from Finder/Explorer into your Chrome window and bask in the glory of your new photo! Congratulations!

### Addendum

- Once again, huge thanks to the JPEG XL developers. [Check out their answers to my questions for troubleshooting tips and more context](https://www.reddit.com/r/jpegxl/comments/wnf0ou/).

- All of this song and dance may be unnecessary in the coming weeks, because I'm thrilled that [Adobe, Intel, and VESA all weighed in on Chrome's bug tracker](https://bugs.chromium.org/p/chromium/issues/detail?id=1178058#c61) voicing support and imminent release (!) of JPEG XL tools.
