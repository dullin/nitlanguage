# Nit on windows

The support on windows is more than experimental.
Fell free to report your attempts.

## Using msys2

msys2 and mingw-w64 package tools and services to compile native Windows applications using GCC and the GNU toolchain.

1. Download and install msys2 by following the instructions at https://msys2.github.io/.

2. Install the minimum required package to download and compile Nit:

    ~~~
    pacman -S make git ccache pkg-config libgc-devel libcurl-devel diffutils mingw-w64-x86_64-gc mingw-w64-x86_64-libunwind mingw-w64-x86_64-pcre mingw-w64-x86_64-libsystre mingw-w64-x86_64-dlfcn mingw-w64-x86_64-pkg-config mingw-w64-x86_64-curl
    ~~~

    We use two regex related packages, `-pcre` provides `libpcreposix.so` and `-libsystre` provides `regex.h`. It is not ideal, but it works.

3.  Then, from the msys2 (or mingw64) command line, clone the Nit repo as usual:

    ~~~
    git clone https://github.com/nitlang/nit.git
    ~~~

4. Compile the Nit tools with:

    ~~~
    GC_MARKERS=1 make
    ~~~

5. As suggested by the previous make `call`, setup your environment with:

    ~~~
    source misc/nit_env.sh install
    ~~~

6. To test the compiler at `bin/nitc.exe`, compile and execute `hello_world.exe` with:

    ~~~
    nitc examples/hello_world.nit
    ./hello_world.exe
    ~~~

7. To distribute a program, copy the required DLL files local to mingw64. For `hello_world.exe`, a call to `ldd hello_world.exe` lists the following files which must be copied:

    ~~~
    C:\msys64\mingw64\bin\libgc-1.dll
    C:\msys64\mingw64\bin\libwinpthread-1.dll
    C:\msys64\mingw64\bin\libpcreposix-0.dll
    ~~~

### Issues

*   If the compiled Nit program hangs, it may be an issue with libgc.
    A workaround is to set the env var `GC_MARKER` to 1, which prevents a deadlock between the many GC threads.

    `GC_MARKERS=1 bin/nitc examples/hello_world.nit`
    
    See: https://github.com/Alexpux/MINGW-packages/issues/1110
    
*   Some modules, including `glesv2` and `egl`, do not need the `pkgconfig` annotation on Windows.
    You may have to remove the line `pkgconfig` from these modules to compile them and their clients.

## Using Cygwin

1.  Download Cygwin from <https://cygwin.com/>
2.  Install it and select the following packages:

    ~~~
    gcc-core gcc-g++ git make ccache pkg-config libisl10 libcloog-isl libgc-devel
    ~~~

3.  Then, from the cygwin command line, clone the Nit repo as usual:

    ~~~
    git clone https://github.com/nitlang/nit.git
    ~~~

    It will complain about the "very bad name" test file. This file cannot be represented on a Windows FS. Ignore this message, all other thousands of files should have unpacked successfully.
    To clear the repository and have a nice working dir status (if you care), you should run:
   
    ~~~
    git chechout -f HEAD
    ~~~

5.  As proposed by the output of the make, setup your environment with

    ~~~
    source misc/nit_env.sh
    ~~~

5.  When you finally have a `bin/nitc.exe`, compile and execute hello_world with:

    ~~~
    nitc examples/hello_world.nit
    ./hello_world.exe
    ~~~

5.  To distribute a program outside of the cygwin environment, copy the cygwin and libgc DDLs with the executable. More DLLs may be needed for programs using native libraries, but for `hello_world.exe` and `nitc.exe` only the following libraries are needed:

    `/usr/bin/cyggc-1.dll /usr/bin/cygwin1.dll`

6.  More work is needed to actually develop Nit programs under a Windows environment. Simple programs like hello_world and Fibonacci work well outside of cygwin. However the compiler still expects UNIX-like paths and can't find nit_dir.

Additional notes:

* I didn't find libunwind for cygwin, but I didn't search much either.

### Cygwin FAQ/issues

*   The initial make does not work
   
    If you have something like:
   
    ~~~
    src/git-gen-version.sh: line 2: $'\r': command not found
    ~~~
   
    It is because you checked out the Nit repository with the config set to `core.autocrlf true`.
    Some `git` tools on Windows activate this flag by default.
    The `git` command from the Cygwin package does not that. 

*   Heap size errors

    If tools or compiled programs crash with heap size errors, you should enlarge the heap of the executable with `peflag` (that comes with Cygwin):
   
    ~~~
    peflags --cygwin-heap=4096 prog.exe
    ~~~
