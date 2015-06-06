# Introduction #

This document discusses the current state of filtering in libyuv. An emphasis on maximum performance while avoiding memory exceptions, and minimal amount of code/complexity.  See future work at end.

# LibYuv Filter Subsampling #

There are 2 challenges with subsampling
1. centering of samples, which involves clamping on edges
2. clipping a source region

Centering depends on scale factor and filter mode.

# Down Sampling #

If scaling down, the stepping rate is always src\_width / dst\_width.

> dx = src\_width / dst\_width;

e.g. If scaling from 1280x720 to 640x360, the step thru the source will be 2.0, stepping over 2 pixels of source for each pixel of destination.

Centering, depends on filter mode.

**Point** downsampling takes the middle pixel.
> x = dx >> 1;
For odd scale factors (e.g. 3x down) this is exactly the middle.  For even scale factors, this rounds up and takes the pixel to the right of center.  e.g. scale of 4x down will take pixel 2.

**Bilinear** filter, uses the 2x2 pixels in the middle.
> x = dx / 2 - 0.5;
For odd scale factors (e.g. 3x down) this is exactly the middle, and point sampling is used.
For even scale factors, this evenly filters the middle 2x2 pixels.  e.g. 4x down will filter pixels 1,2 at 50% in both directions.

**Box** filter averages the entire box so sampling starts at 0.
> x = 0;
For a scale factor of 2x down, this is equivalent to bilinear.

# Up Sampling #

**Point** upsampling use stepping rate of src\_width / dst\_width and a starting coordinate of 0.

> x = 0;<br>
<blockquote>dx = src_width / dst_width;</blockquote>

e.g. If scaling from 640x360 to 1280x720 the step thru the source will be 0.0, stepping half a pixel of source for each pixel of destination. Each pixel is replicated by the scale factor.<br>
<br>
<b>Bilinear</b> filter stretches such that the first pixel of source maps to the first pixel of destination, and the last pixel of source maps to the last pixel of destination.<br>
<blockquote>x = 0;<br>
dx = (src_width - 1) / (dst_width - 1);</blockquote>

This method is not technically correct, and will likely change in the future.<br>
<ul><li>It is inconsistent with the bilinear down sampler.  The same method could be used for down sampling, and then it would be more reversible, but that would prevent specialized 2x down sampling.<br>
</li><li>Although centered, the image is slightly magnified.<br>
</li><li>The filtering was changed in early 2013 - previously it used:<br>
<blockquote>x = 0;<br>
dx = (src_width - 1) / (dst_width - 1);<br>
</blockquote></li></ul><blockquote>Which is the correct scale factor, but shifted the image left, and extruded the last pixel.  The reason for the change was to remove the extruding code from the low level row functions, allowing 3 functions to sshare the same row functions - ARGBScale, I420Scale, and ARGBInterpolate.  Then the one function was ported to many cpu variations: SSE2, SSSE3, AVX2, Neon and 'Any' version for any number of pixels and alignment.  The function is also specialized for 0,25,50,75%.</blockquote>

The above goes still has the potential to read the last pixel 100% and last pixel + 1 0%, which may cause a memory exception.  So the left pixel goes to a fraction less than the last pixel, but filters in the minimum amount of it, and the maximum of the last pixel.<br>
<br>
<blockquote>dx = FixedDiv((src_width << 16) - 0x00010001, (dst << 16) - 0x00010000);</blockquote>

<b>Box</b> filter for upsampling switches over to Bilinear.<br>
<br>
<h1>Scale snippet:</h1>
<pre><code>#define CENTERSTART(dx, s) (dx &lt; 0) ? -((-dx &gt;&gt; 1) + s) : ((dx &gt;&gt; 1) + s)<br>
#define FIXEDDIV1(src, dst) FixedDiv((src &lt;&lt; 16) - 0x00010001, \<br>
                                     (dst &lt;&lt; 16) - 0x00010000);<br>
<br>
// Compute slope values for stepping.<br>
void ScaleSlope(int src_width, int src_height,<br>
                int dst_width, int dst_height,<br>
                FilterMode filtering,<br>
                int* x, int* y, int* dx, int* dy) {<br>
  assert(x != NULL);<br>
  assert(y != NULL);<br>
  assert(dx != NULL);<br>
  assert(dy != NULL);<br>
  assert(src_width != 0);<br>
  assert(src_height != 0);<br>
  assert(dst_width &gt; 0);<br>
  assert(dst_height &gt; 0);<br>
  if (filtering == kFilterBox) {<br>
    // Scale step for point sampling duplicates all pixels equally.<br>
    *dx = FixedDiv(Abs(src_width), dst_width);<br>
    *dy = FixedDiv(src_height, dst_height);<br>
    *x = 0;<br>
    *y = 0;<br>
  } else if (filtering == kFilterBilinear) {<br>
    // Scale step for bilinear sampling renders last pixel once for upsample.<br>
    if (dst_width &lt;= Abs(src_width)) {<br>
      *dx = FixedDiv(Abs(src_width), dst_width);<br>
      *x = CENTERSTART(*dx, -32768);<br>
    } else if (dst_width &gt; 1) {<br>
      *dx = FIXEDDIV1(Abs(src_width), dst_width);<br>
      *x = 0;<br>
    }<br>
    if (dst_height &lt;= src_height) {<br>
      *dy = FixedDiv(src_height,  dst_height);<br>
      *y = CENTERSTART(*dy, -32768);  // 32768 = -0.5 to center bilinear.<br>
    } else if (dst_height &gt; 1) {<br>
      *dy = FIXEDDIV1(src_height, dst_height);<br>
      *y = 0;<br>
    }<br>
  } else if (filtering == kFilterLinear) {<br>
    // Scale step for bilinear sampling renders last pixel once for upsample.<br>
    if (dst_width &lt;= Abs(src_width)) {<br>
      *dx = FixedDiv(Abs(src_width), dst_width);<br>
      *x = CENTERSTART(*dx, -32768);<br>
    } else if (dst_width &gt; 1) {<br>
      *dx = FIXEDDIV1(Abs(src_width), dst_width);<br>
      *x = 0;<br>
    }<br>
    *dy = FixedDiv(src_height, dst_height);<br>
    *y = *dy &gt;&gt; 1;<br>
  } else {<br>
    // Scale step for point sampling duplicates all pixels equally.<br>
    *dx = FixedDiv(Abs(src_width), dst_width);<br>
    *dy = FixedDiv(src_height, dst_height);<br>
    *x = CENTERSTART(*dx, 0);<br>
    *y = CENTERSTART(*dy, 0);<br>
  }<br>
  // Negative src_width means horizontally mirror.<br>
  if (src_width &lt; 0) {<br>
    *x += (dst_width - 1) * *dx;<br>
    *dx = -*dx;<br>
    src_width = -src_width;<br>
  }<br>
}<br>
</code></pre>

<h1>Future Work</h1>

Point sampling should ideally be the same as bilinear, but pixel by pixel, round to nearest neighbor.  But as is, it is reversible and exactly matches ffmpeg at all scale factors, both up and down.  The scale factor is<br>
<blockquote>dx = src_width / dst_width;<br>
The step value is centered for down sample:<br>
x = dx / 2;<br>
Or starts at 0 for upsample.<br>
x = 0;<br>
Bilinear filtering is currently correct for down sampling, but not for upsampling.<br>
Upsampling is stretching the first and last pixel of source, to the first and last pixel of destination.<br>
dx = (src_width - 1) / (dst_width - 1);<br>
x = 0;<br>
It should be stretching such that the first pixel is centered in the middle of the scale factor, to match the pixel that would be sampled for down sampling by the same amount.  And same on last pixel.<br>
dx = src_width / dst_width;<br>
x = dx / 2 - 0.5;<br>
This would start at -0.5 and go to last pixel + 0.5, sampling 50% from last pixel + 1.<br>
Then clamping would be needed.  On GPUs there are numerous ways to clamp.<br>
1. Clamp the coordinate to the edge of the texture, duplicating the first and last pixel.<br>
2. Blend with a constant color, such as transparent black.  Typically best for fonts.<br>
3. Mirror the UV coordinate, which is similar to clamping.  Good for continuous tone images.<br>
4. Wrap the coordinate, for texture tiling.<br>
5. Allow the coordinate to index beyond the image, which may be the correct data if sampling a subimage.<br>
6. Extrapolate the edge based on the previous pixel.  pixel -0.5 is computed from slope of pixel 0 and 1.<br>
Some of these are computational, even for a GPU, which is one reason textures are sometimes limited to power of 2 sizes.<br>
We do care about the clipping case, where allowing coordinates to become negative and index pixels before the image is the correct data.  But normally for simple scaling, we want to clamp to the edge pixel.  For example, if bilinear scaling from 3x3 to 30x30, we’d essentially want 10 pixels of each of the original 3 pixels.  But we want the original pixels to land in the middle of each 10 pixels, at offsets 5, 15 and 25.  There would be filtering between 5 and 15 between the original pixels 0 and 1.  And filtering between 15 and 25 from original pixels 1 and 2.  The first 5 pixels are clamped to pixel 0 and the last 5 pixels are clamped to pixel 2.<br>
The easiest way to implement this is copy the original 3 pixels to a buffer, and duplicate the first and last pixels.  0,1,2 becomes 0, 0,1,2, 2.  Then implement a filtering without clamping.  We call this source extruding.  Its only necessary on up sampling, since down sampler will always have valid surrounding pixels.<br>
Extruding is practical when the image is already copied to a temporary buffer.   It could be done to the original image, as long as the original memory is restored, but valgrind and/or memory protection would disallow this, so it requires a memcpy to a temporary buffer, which may hurt performance.  The memcpy has a performance advantage, from a cache point of view, that can actually make this technique faster, depending on hardware characteristics.<br>
Vertical extrusion can be done with a memcpy of the first/last row, or clamping a pointer.</blockquote>


The other way to implement clamping is handle the edges with a memset.  e.g. Read first source pixel and memset the first 5 pixels.  Filter pixels 0,1,2 to 5 to 25.  Read last pixel and memset the last 5 pixels.  Blur is implemented with this method like this, which has 3 loops per row - left, middle and right.<br>
<br>
Box filter is only used for 2x down sample or more.  Its based on integer sized boxes.  Technically it should be filtered edges, but thats substantially slower (roughly 100x), and at that point you may as well do a cubic filter which is more correct.<br>
<br>
Box filter currently sums rows into a row buffer.  It does this with<br>
<br>
Mirroring will use the same slope as normal, but with a negative.<br>
The starting coordinate needs to consider the scale factor and filter.  e.g. box filter of 30x30 to 3x3 with mirroring would use -10 for step, but x = 20.  width (30) - dx.<br>
<br>
Step needs to be accurate, so it uses an integer divide.  This is as much as 5% of the profile.  An approximated divide is substantially faster, but the inaccuracy causes stepping beyond the original image boundaries.  3 general solutions:<br>
1. copy image to buffer with padding.  allows for small errors in stepping.<br>
2. hash the divide, so common values are quickly found.<br>
3. change api so caller provides the slope.