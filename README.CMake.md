# BUILDING AND INSTALLING

We use CMake to build the project.

Modern CMake (3.14) instructions:

    cmake -S . -B build
    cmake --build build
    cmake --build build -t test

This will make a build directory (`-B`) if it does not exist, with the source 
directory defined as `-S`.  CMake will configure and generate makefiles by 
default, as well as set all options to their default settings and cache them 
into a file called `CMakeCache.txt`, which will sit in the build directory. 
You can call the build directory anything you want; by convention it should 
have the word build in it to be ignored by most package’s `.gitignore` files.

You can then invoke your build system (line 2). Regardless of whether you used 
`make` (the default), `ninja`, or even an IDE-based system, you can build with
a uniform command. You can add `-j 2` to build on two cores, or `-v` to 
verbosely show commands used to build.

Finally, you can even run your tests from here, by passing the `test` target 
to the underlying build system. `-t` (`--target` before CMake 3.15) lets you 
select a target. 

Install library:

    cmake --build build -t install



The classic, battle hardened method should be shown for completeness:

    mkdir build
    cd build
    cmake ..
    make
    make test

To install:

    cd build
    make install

## PICKING A COMPLILER

Selecting a compiler must be done on the first run in an empty directory. 
It's not CMake syntax per se, but you might not be familiar with it. 
To pick Clang:

    CC=clang CXX=clang++ cmake -S . -B build

That sets the environment variables in bash for CC and CXX, and CMake will
respect those variables. This sets it just for that one line, but that's the 
only time you'll need those; afterwards CMake continues to use the paths it
deduces from those values.


## PICKING A GENERATOR

You can build with a variety of tools; make is usually the default. To see 
all the tools CMake knows about on your system, run

    cmake --help

And you can pick a tool with -G"My Tool" (quotes only needed if spaces are 
in the tool name). You should pick a tool on your first CMake call in a 
directory, just like the compiler. Feel free to have several build directories,
like `build` and `build-xcode`. You can set the environment variable 
`CMAKE_GENERATOR` to control the default generator (CMake 3.15+). Note that 
makefiles will only run in parallel if you explicitly pass a number of threads,
such as `make -j2`, while Ninja will automatically run in parallel. You can 
directly pass a parallelization option such as `-j 2` to the `cmake --build .` 
command in recent versions of CMake as well.


## OPTIONS

CMake has support for cached options. A Variable in CMake can be marked as 
"cached", which means it will be written to the cache (a file called 
`CMakeCache.txt` in the build directory) when it is encountered. You can preset
(or change) the value of a cached option on the command line with `-D`. 
When CMake looks for a cached variable, it will use the existing value and will
not overwrite it.

## STANDARD OPTIONS

These are common CMake options to most packages:

- `CMAKE_BUILD_TYPE`: Pick from `Release`, `RelWithDebInfo`, `Debug`, or 
    sometimes more.
- `CMAKE_INSTALL_PREFIX`: The location to install to. System install on UNIX 
    would often be `/usr/local` (the default), user directories are often
    `~/.local`, or you can pick a folder.

- `BUILD_SHARED_LIBS`: You can set this ON or OFF to control the default for 
    shared libraries (the author can pick one vs. the other explicitly instead
    of using the default, though)
- `BUILD_TESTING`: This is a common name for enabling tests, not all packages 
    use it, though, sometimes with good reason.

Intructions are based on [Modern CMake intro/running](https://cliutils.gitlab.io/modern-cmake/chapters/intro/running.html)


## LAME-SPECIFIC OTPIONS

TODO: document the options for LAME library