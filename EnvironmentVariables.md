# Introduction #

For test purposes, environment variables can be set to control libyuv behavior.  These should only be used for testing, to narrow down bugs or to test performance.

# CPU #

By default the cpu is detected and the most advanced form of SIMD is used.  But you can disable instruction sets selectively, or completely, falling back on C code.  Set the variable to 1 to disable the specified instruction set.

<pre>
LIBYUV_DISABLE_ASM<br>
LIBYUV_DISABLE_X86<br>
LIBYUV_DISABLE_SSE2<br>
LIBYUV_DISABLE_SSSE3<br>
LIBYUV_DISABLE_SSE41<br>
LIBYUV_DISABLE_SSE42<br>
LIBYUV_DISABLE_AVX<br>
LIBYUV_DISABLE_AVX2<br>
LIBYUV_DISABLE_ERMS<br>
LIBYUV_DISABLE_FMA3<br>
LIBYUV_DISABLE_MIPS<br>
LIBYUV_DISABLE_MIPS_DSP<br>
LIBYUV_DISABLE_MIPS_DSPR2<br>
LIBYUV_DISABLE_NEON<br>
</pre>

# Test Width/Height/Repeat #

The unittests default to a small image (32x18) to run fast.  This can be set by environment variable to test a specific resolutions.
you can also repeat the test a specified number of iterations, allowing benchmarking and profiling.

<pre>
set LIBYUV_WIDTH=1280<br>
set LIBYUV_HEIGHT=720<br>
set LIBYUV_REPEAT=999<br>
set LIBYUV_FLAGS=-1<br>
</pre>