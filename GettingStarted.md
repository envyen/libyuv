# Getting Started #

How to get and build the libyuv code.

## Pre-requisite ##

You'll need to have depot tools installed:

https://sites.google.com/a/chromium.org/dev/developers/how-tos/depottools

Refer to chromium instructions for each platform for other prerequesites:<br>


<h2>Getting the Code</h2>

Create a working directory, enter it, and run:<br>
<br>
<b>
<pre>
gclient config http://libyuv.googlecode.com/svn/trunk<br>
gclient sync<br>
</pre>
</b>

<pre>
solutions = [<br>
{ "name"        : "trunk",<br>
"url"         : "https://libyuv.googlecode.com/svn/trunk",<br>
"deps_file"   : "DEPS",<br>
"managed"     : True,<br>
"custom_deps" : {<br>
},<br>
"safesync_url": "",<br>
},<br>
];<br>
</pre>

For iOS add ;target_os=['ios']; to your OSX .gclient and run gclient sync --nohooks twice.<br>
For Android add ;target_os=['android']; to your Linux .gclient and run gclient sync --nohooks<br>
For Windows the gclient sync must be done from an Administrator command prompt.<br>

The sync will generate native build files for your environment using gyp (Windows: Visual Studio, OSX: XCode, Linux: make). This generation can also be forced manually: gclient runhooks<br>
<br>
To get just the source (not buildable):<br>
<pre>
svn checkout https://libyuv.googlecode.com/svn/trunk/ libyuv<br>
</pre>

<h3>Git</h3>
A git mirror is available here<br>
<a href='http://git.chromium.org/gitweb/?p=external/libyuv.git;a=summary'>http://git.chromium.org/gitweb/?p=external/libyuv.git;a=summary</a>

<h2>Building the Library and Unittests</h2>

Note: To build just the library and not the tests, change libyuv_test.gyp to libyuv.gyp<br>
<br>
<h3>Windows</h3>

<pre>
set GYP_DEFINES=target_arch=ia32<br>
call python gyp_libyuv -fninja -G msvs_version=2013 --depth=. libyuv_test.gyp<br>
ninja -j7 -C out\Release<br>
ninja -j7 -C out\Debug<br>
<br>
set GYP_DEFINES=target_arch=x64<br>
call python gyp_libyuv -fninja -G msvs_version=2013 --depth=. libyuv_test.gyp<br>
ninja -C out\Debug_x64<br>
ninja -C out\Release_x64<br>
</pre>
<h4>Building with clangcl</h4>
<pre>
set GYP_DEFINES=clang=1 target_arch=ia32 libyuv_enable_svn=1<br>
set LLVM_REPO_URL=svn://svn.chromium.org/llvm-project<br>
call python tools\clang\scripts\update.py<br>
call python gyp_libyuv -fninja libyuv_test.gyp<br>
ninja -C out\Debug<br>
ninja -C out\Release<br>
</pre>
<h4>Building with GN</h4>

<pre>
call gn gen out/Release "--args=is_debug=false target_cpu=\"x86\""<br>
call gn gen out/Debug "--args=is_debug=true target_cpu=\"x86\""<br>
ninja -C out/Release<br>
ninja -C out/Debug<br>
<br>
</pre>

<h3>OSX</h3>

# Clang 64 bit shown.  Remove clang=1 for gcc and change x64 to ia32 for 32 bit <br>
<pre>
GYP_DEFINES="clang=1 target_arch=x64" ./gyp_libyuv -fninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug<br>
ninja -j7 -C out/Release<br>
<br>
GYP_DEFINES="clang=1 target_arch=ia32" ./gyp_libyuv -fninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug<br>
ninja -j7 -C out/Release<br>
</pre>

<h3>Linux</h3>

<pre>
tools/clang/scripts/update.sh<br>
GYP_DEFINES="target_arch=x64" ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug<br>
ninja -j7 -C out/Release<br>
<br>
GYP_DEFINES="target_arch=ia32" ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug<br>
ninja -j7 -C out/Release<br>
</pre>

<h4>CentOS</h4>

On CentOS 32 bit the following work around allows a sync:<br>
<pre>
export GYP_DEFINES="host_arch=ia32"<br>
gclient sync<br>
</pre>

<h3>iOS</h3>
<a href='http://www.chromium.org/developers/how-tos/build-instructions-ios'>http://www.chromium.org/developers/how-tos/build-instructions-ios</a><br>
Add to .gclient last line: target_os=['ios'];<br>
<br>
armv7<br>
<pre>
GYP_DEFINES="OS=ios target_arch=armv7" GYP_CROSSCOMPILE=1 GYP_GENERATOR_FLAGS="output_dir=out_ios" ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out_ios/Debug-iphoneos libyuv_unittest<br>
ninja -j7 -C out_ios/Release-iphoneos libyuv_unittest<br>
</pre>

arm64<br>
<pre>
GYP_DEFINES="OS=ios target_arch=armv7 target_subarch=64" GYP_CROSSCOMPILE=1 GYP_GENERATOR_FLAGS="output_dir=out_ios" ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out_ios/Debug-iphoneos libyuv_unittest<br>
ninja -j7 -C out_ios/Release-iphoneos libyuv_unittest<br>
</pre>

<h3>Android</h3>
<a href='https://code.google.com/p/chromium/wiki/AndroidBuildInstructions'>https://code.google.com/p/chromium/wiki/AndroidBuildInstructions</a><br>
Add to .gclient last line: target_os=['android'];<br>
<br>
armv7<br>
<pre>
GYP_DEFINES="OS=android" GYP_CROSSCOMPILE=1 ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug libyuv_unittest<br>
ninja -j7 -C out/Release libyuv_unittest<br>
</pre>

arm64<br>
<pre>
GYP_DEFINES="OS=android target_arch=arm64" GYP_CROSSCOMPILE=1 ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug libyuv_unittest<br>
ninja -j7 -C out/Release libyuv_unittest<br>
</pre>

ia32<br>
<pre>
GYP_DEFINES="OS=android target_arch=ia32" GYP_CROSSCOMPILE=1 ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug libyuv_unittest<br>
ninja -j7 -C out/Release libyuv_unittest<br>
</pre>

mipsel<br>
<pre>
GYP_DEFINES="OS=android target_arch=mipsel" GYP_CROSSCOMPILE=1 ./gyp_libyuv -f ninja --depth=. libyuv_test.gyp<br>
ninja -j7 -C out/Debug libyuv_unittest<br>
ninja -j7 -C out/Release libyuv_unittest<br>
</pre>

arm64 disassembly:<br>
<pre>
./third_party/android_tools/ndk//toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-objdump -d out/Release/obj/source/libyuv_neon.row_neon64.o<br>
</pre>

Old method:<br>
<pre>
source build/android/envsetup.sh<br>
gclient runhooks<br>
ninja -j7 -C out/Debug<br>
ninja -j7 -C out/Release<br>
</pre>

<h3>Windows Shared Library</h3>

Modify libyuv.gyp from 'static_library' to 'shared_library', and add 'LIBYUV_BUILDING_SHARED_LIBRARY' to 'defines'. <br>
gclient runhooks <br>
After this command follow the building the library instructions above.<br>

If you get a compile error for atlthunk.lib on Windows, read <a href='http://www.chromium.org/developers/how-tos/build-instructions-windows'>http://www.chromium.org/developers/how-tos/build-instructions-windows</a><br>

<h3>Build targets</h3>

<pre>
ninja -C out/Debug libyuv<br>
ninja -C out/Debug libyuv_unittest<br>
ninja -C out/Debug compare<br>
ninja -C out/Debug convert<br>
ninja -C out/Debug psnr<br>
ninja -C out/Debug cpuid<br>
</pre>

<h2>Building the Library with make</h2>

<h3>Linux</h3>

<pre>
make -j7 V=1 -f linux.mk<br>
make -j7 V=1 -f linux.mk clean<br>
make -j7 V=1 -f linux.mk CXX=clang++<br>
</pre>

<h2>Building the Library with cmake</h2>

Install cmake:<br>
<a href='http://www.cmake.org/'>http://www.cmake.org/</a>
<pre>
Default debug build:<br>
mkdir out<br>
cd out<br>
cmake ..<br>
cmake --build .<br>
<br>
Release build/install<br>
mkdir out<br>
cd out<br>
cmake -DCMAKE_INSTALL_PREFIX="/usr/lib" -DCMAKE_BUILD_TYPE="Release" ..<br>
cmake --build . --config Release<br>
sudo cmake --build . --target install --config Release<br>
<br>
</pre>

<h3>Windows 8 Phone</h3>

Pre-requisite:<br>
Install Visual Studio 2012 and Arm to your environment.<br>
<pre>
call "c:\Program Files (x86)\Microsoft Visual Studio 11.0\VC\bin\x86_arm\vcvarsx86_arm.bat"<br>
</pre>
or Visual Studio 2013<br>
<pre>
call "c:\Program Files (x86)\Microsoft Visual Studio 12.0\VC\bin\x86_arm\vcvarsx86_arm.bat"<br>
nmake /f winarm.mk clean<br>
nmake /f winarm.mk<br>
</pre>

<h2>Building the Library with native builds (deprecated)</h2>

<h3>Windows</h3>

<pre>
gclient runhooks<br>
devenv -build "Debug" libyuv.sln<br>
devenv -build "Release" libyuv.sln<br>
</pre>
<h3>OSX</h3>

<pre>
gclient runhooks<br>
xcodebuild -project libyuv.xcodeproj -configuration Debug<br>
xcodebuild -project libyuv.xcodeproj -configuration Release<br>
</pre>

<h3>Linux</h3>

<pre>
gclient runhooks<br>
make -j7 V=1 -r libyuv BUILDTYPE=Debug<br>
make -j7 V=1 -r libyuv BUILDTYPE=Release<br>
</pre>

<h3>iOS</h3>

<pre>
GYP_DEFINES="OS=ios target_arch=armv7 armv7=1 arm_neon=1" GYP_CROSSCOMPILE=1 ./gyp_libyuv  --depth=. libyuv_test.gyp<br>
xcodebuild -project libyuv.xcodeproj -configuration Debug -arch armv7 -sdk iphoneos7.0<br>
xcodebuild -project libyuv.xcodeproj -configuration Release -arch armv7 -sdk iphoneos7.0<br>
</pre>

<h3>Windows Shared Library</h3>

Modify libyuv.gyp from 'static_library' to 'shared_library', and add 'LIBYUV_BUILDING_SHARED_LIBRARY' to 'defines'. Then run this.<br>
gclient runhooks <br>
After this command follow the building the library instructions above.<br>

If you get a compile error for atlthunk.lib on Windows, read <a href='http://www.chromium.org/developers/how-tos/build-instructions-windows'>http://www.chromium.org/developers/how-tos/build-instructions-windows</a>

<h3>64 bit Windows</h3>

<pre>
set GYP_DEFINES=target_arch=x64<br>
gclient runhooks V=1<br>
</pre>

<h3>ARM Linux</h3>

<pre>
$ export GYP_DEFINES="target_arch=arm"<br>
$ export CROSSTOOL=<path>/arm-none-linux-gnueabi<br>
$ export CXX=$CROSSTOOL-g++<br>
$ export CC=$CROSSTOOL-gcc<br>
$ export AR=$CROSSTOOL-ar<br>
$ export AS=$CROSSTOOL-as<br>
$ export RANLIB=$CROSSTOOL-ranlib<br>
$ gclient runhooks<br>
</pre>

<h2>Running Unittests</h2>

<h3>Windows</h3>

<pre>
out\Release\libyuv_unittest.exe --gtest_catch_exceptions=0 --gtest_filter=*<br>
</pre>

<h3>OSX</h3>

<pre>
out/Release/libyuv_unittest --gtest_filter=*<br>
</pre>

<h3>Linux</h3>

<pre>
out/Release/libyuv_unittest --gtest_filter=*<br>
</pre>

Replace --gtest_filter=<code>*</code> with specific unittest to run.  May include wildcards. e.g.<br>

<pre>
out/Release/libyuv_unittest --gtest_filter=libyuvTest.I420ToARGB_Opt<br>
</pre>

<h2>CPU Emulator tools</h2>

<h3>Intel SDE (Software Development Emulator)</h3>

Pre-requisite:<br>
Install IntelSDE for Windows.<br>
<a href='http://software.intel.com/en-us/articles/intel-software-development-emulator'>http://software.intel.com/en-us/articles/intel-software-development-emulator</a>

<pre>
c:\intelsde\sde -hsw -- out\release\libyuv_unittest.exe --gtest_filter=*<br>
</pre>

<h2>Memory tools</h2>

<h3>Running Dr Memory memcheck for Windows</h3>

Pre-requisite:<br>
Install Dr Memory for Windows and add it to your path.<br>
<a href='http://www.drmemory.org/docs/page_install_windows.html'>http://www.drmemory.org/docs/page_install_windows.html</a>

<pre>
set GYP_DEFINES=build_for_tool=drmemory target_arch=ia32<br>
call python gyp_libyuv -fninja -G msvs_version=2012 --depth=. libyuv_test.gyp<br>
ninja -C out\Debug<br>
drmemory out\Debug\libyuv_unittest.exe --gtest_catch_exceptions=0 --gtest_filter=*<br>
</pre>


<h3>Running Valgrind memcheck</h3>

Memory errors and race conditions can be found by running tests under special memory tools. <a href='http://valgrind.org/'>Valgrind</a> is an instrumentation framework for building dynamic analysis tools. Various tests and profilers are built upon it to find memory handling errors and memory leaks, for instance.<br>
<br>
<pre>
solutions = [<br>
{ "name"        : "trunk",<br>
"url"         : "http://libyuv.googlecode.com/svn/trunk",<br>
"deps_file"   : "DEPS",<br>
"managed"     : True,<br>
"custom_deps" : {<br>
"third_party/valgrind": "http://src.chromium.org/svn/trunk/deps/third_party/valgrind/binaries",<br>
},<br>
"safesync_url": "",<br>
},<br>
]<br>
<br>
GYP_DEFINES="clang=0 target_arch=x64 build_for_tool=memcheck" python gyp_libyuv -fninja --depth=. libyuv_test.gyp<br>
ninja -C out/Debug<br>
valgrind out/Debug/libyuv_unittest<br>
</pre>

For more information, see<br>
<a href='http://www.chromium.org/developers/how-tos/using-valgrind'>http://www.chromium.org/developers/how-tos/using-valgrind</a>

<h3>Running Thread Sanitizer (tsan)</h3>

<pre>
GYP_DEFINES="clang=0 target_arch=x64 build_for_tool=tsan" python gyp_libyuv -fninja --depth=. libyuv_test.gyp<br>
ninja -C out/Debug<br>
valgrind out/Debug/libyuv_unittest<br>
</pre>

For more info, see<br>
<a href='http://www.chromium.org/developers/how-tos/using-valgrind/threadsanitizer'>http://www.chromium.org/developers/how-tos/using-valgrind/threadsanitizer</a>

<h3>Running Address Sanitizer (asan)</h3>

<pre>
GYP_DEFINES="clang=0 target_arch=x64 build_for_tool=asan" python gyp_libyuv -fninja --depth=. libyuv_test.gyp<br>
ninja -C out/Debug<br>
valgrind out/Debug/libyuv_unittest<br>
</pre>

For more info, see<br>
<a href='http://dev.chromium.org/developers/testing/addresssanitizer'>http://dev.chromium.org/developers/testing/addresssanitizer</a>

<h2>Benchmarking</h2>

The unittests can be used to benchmark.<br><br>

<h3>Windows</h3>
<pre>
set LIBYUV_WIDTH=1280<br>
set LIBYUV_HEIGHT=720<br>
set LIBYUV_REPEAT=999<br>
set LIBYUV_FLAGS=-1<br>
out\Release\libyuv_unittest.exe --gtest_filter=*I420ToARGB_Opt<br>
</pre>

<h3>Linux and Mac</h3>
<pre>
LIBYUV_WIDTH=1280 LIBYUV_HEIGHT=720 LIBYUV_REPEAT=1000 out/Release/libyuv_unittest --gtest_filter=*I420ToARGB_Opt<br>
<br>
libyuvTest.I420ToARGB_Opt (547 ms)<br>
</pre>
Indicates 0.547 ms/frame for 1280 x 720.<br>
<br>
<h2>Making a change</h2>
<pre>
gclient sync<br>
make changed<br>
gcl change mycl<br>
gcl upload mycl<br>
requires auth2 once<br>
depot-tools-auth login webrtc-codereview.appspot.com<br>
gcl try mycl<br>
gcl lint mycl<br>
gcl commit mycl<br>
</pre>