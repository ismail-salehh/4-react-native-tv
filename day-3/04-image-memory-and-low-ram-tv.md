# Image memory on low-RAM TVs

Many TVs have 1–2 GB of RAM, shared with the video decoder and the OS. Images are usually the largest part of your app's memory.

What counts is the **decoded** size, not the file size. A decoded image takes about **width × height × 4 bytes**, whatever the JPEG weighed on the network:

| Image | Decoded |
| --- | --- |
| Card 300 × 450 | ~0.5 MB |
| Card asset served at 1000 × 1500 | ~6 MB |
| Hero 1920 × 1080 | ~8 MB |
| Hero served at 3840 × 2160 (4K) | ~33 MB |

A home screen of 10 rails × 20 cards served at 1000 × 1500 is around **1.2 GB** of decoded images, enough to get the app killed. The same cards served at 300 × 450 are about 100 MB.

Right-size the images:

- **Ask the server for the size you render.** Use the image CDN's width/height parameters, at the rendered size × pixel ratio (see [Layout and styling](../docs-2/01-layout-and-styling.md#design-at-19201080-and-scale)). A 4K TV that renders at 1080p does not need 4K posters.
- **Let the image library downsample** when you can't control the source: on Android, `resizeMethod="resize"` on `Image`, or the equivalent option in `expo-image` / FastImage.
- **Limit what is mounted**: virtualised rails (above) unmount off-screen cards and free their images. Don't preload every rail's artwork.
- **Release big images** on screens kept mounted in the background (see [Keep mounted vs unmount](../docs-2/03-app-structure.md#keep-mounted-vs-unmount)).
- **Set a memory cache limit** in your image library, if it has one, smaller than its phone default.
- **On web**, an `<img>` is decoded at its intrinsic size, whatever its CSS size, so a 4K image in a 300 px box still costs 33 MB. Serve the right size.

Measure memory on the device: Android Studio's memory profiler for Android TV, Xcode Instruments for Apple TV, and the remote inspector's Memory tab for LG and Samsung.