<img align="right" src="http://www.hdwallpaperstock.eu/wallpapers/funny_pug_dog-2880x1800.jpg" width="320" alt="@todo, disclaimer here: unknown author, pending license http://i.ytimg.com/vi/pzPxhaYQQK8/hqdefault.jpg" />
### PUG
####

- pug, png with a twist: lossy image format.
- pug, `PUG = color.JPG + alpha.PNG + footer`
- pug file format, tools and related source code is all public domain.

#### Features
- PUG images are smaller than PNG, and perceptually similar to JPG.
- PUG manipulation does not require additional image libraries.
- Regular image viewers should display PUG files out of the box.

#### Cons
- PUG is a lossy file format. Reconstructed images from PUG files will not match original images.
- Regular image viewers will ignore PUG alpha channel until fully supported.

#### Pug = lossy color + lossless alpha layers

| Original image    | Lossy color | Lossless alpha | Pug image |
| :-------------: |:-------------:| :-----:| :-----: |
| ![image](https://raw.github.com/r-lyeh/pug/master/images/panda.png) | ![image](https://raw.github.com/r-lyeh/pug/master/images/panda.pug.color.jpg) | ![image](https://raw.github.com/r-lyeh/pug/master/images/panda.pug.alpha.png) | ![image](https://raw.github.com/r-lyeh/pug/master/images/panda.pug.rebuilt.png) |
| PNG Image, 56 KiB | JPG layer, Q=50% | PNG layer, 8-bit | PUG Image, 12 KiB |
| ![image](https://raw.github.com/r-lyeh/pug/master/images/dices.png) | ![image](https://raw.github.com/r-lyeh/pug/master/images/dices.pug.color.jpg) | ![image](https://raw.github.com/r-lyeh/pug/master/images/dices.pug.alpha.png) | ![image](https://raw.github.com/r-lyeh/pug/master/images/dices.pug.rebuilt.png) |
| PNG Image, 213 KiB | JPG layer, Q=50% | PNG layer, 8-bit | PUG Image, 74 KiB |
| ![image](https://raw.github.com/r-lyeh/pug/master/images/babytux.png) | ![image](https://raw.github.com/r-lyeh/pug/master/images/babytux.pug.color.jpg) | ![image](https://raw.github.com/r-lyeh/pug/master/images/babytux.pug.alpha.png) | ![image](https://raw.github.com/r-lyeh/pug/master/images/babytux.pug.rebuilt.png) |
| PNG Image, 40 KiB | JPG layer, Q=50% | PNG layer, 8-bit | PUG Image, 22 KiB |

#### PUG file format specification
- RGBA32 input image is split into RGB24.jpg and A8.png images, then glued together with a footer.

```c++
RGBA32.pug = [RGB24.jpg] + [optional padding] + [A8.png] + [optional padding] + [footer]

- [?? bytes]  lossy color chunk (JPG stream)
- [?? bytes]  lossy color padding (optional)
- [?? bytes]  lossless alpha chunk (PNG stream)
- [?? bytes]  lossless alpha padding (optional)
- [ 4 bytes]  footer chunk - sizeof( color + color_padding ) (little endian)
- [ 4 bytes]  footer chunk - sizeof( alpha + alpha_padding ) (little endian)
- [ 4 bytes]  footer chunk - header and version "pug1" (char *)
```

#### Theory, split pixel algebra (png->pug)
Assuming pixel algebra is present and pixel is a vec4f, split pixel operations follow:

```c++
// split rgba into rgb+a
alpha.at(w,h) = rgba.at(w,h) * pixel(0,0,0,255);
  rgb.at(w,h) = rgba.at(w,h) * pixel(255,255,255,0);
```

#### Theory, joint pixel algebra (pug->png)
Assuming pixel algebra is present and pixel is a vec4f, joint pixel operations follow:

```c++
// joint rgb+a into rgba
rgba.at(w,h) = rgb.at(w,h) * pixel(255,255,255,0) + alpha.at(w,h) * pixel(0,0,0,255);
```

#### Minimal provided tools
- [pugify.exe](https://raw.github.com/r-lyeh/pug/master/tools/pugify.exe), which does PNG <=> PUG conversions ([source code](tools/pugify.cc))
- [pugview.exe](https://raw.github.com/r-lyeh/pug/master/tools/pugview.exe), which previews image files (including pug files) ([source code](tools/pugview.cc))

#### Notes
- Pugify uses tiny image encoders that may lead to sub-optimal JPG/PNG file sizes. Therefore, Pugify may generate sub-optimal (but fully working) PUG files. If you plan to use Pugify in production scenarios consider integrating better image encoders before merging RGB24 and A8 files both together.
- [panda.png](images/panda.png) taken from http://tinypng.com website.
- [dices.png](images/dices.png) taken from POV-Ray source code (CCASA30U license), [source](http://upload.wikimedia.org/wikipedia/commons/4/47/PNG_transparency_demonstration_1.png)
- [babytux.png](images/babytux.png) by Fizyplankton (public domain), [source](http://www.minecraftwiki.net/images/8/85/Fizyplankton.png)
- Uncredited pug image logo (source? help!)

#### Appendix: on compiling provided tools
```
git submodule init
git submodule update

cl pugview.cc -I deps deps\spot\spot*.c* /Ox /Oy /MT /DNDEBUG
cl pugify.cc  -I deps deps\spot\spot*.c* /Ox /Oy /MT /DNDEBUG

pugify.exe deps\spot\images\panda.png panda.pug 50
pugview.exe panda.pug

pugify.exe panda.pug panda.png
pugview.exe panda.png
```
