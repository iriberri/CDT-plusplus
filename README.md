CDT-plusplus
============
 [![Build Status](https://travis-ci.org/acgetchell/CDT-plusplus.png?branch=master)](https://travis-ci.org/acgetchell/CDT-plusplus) [![Build status](https://ci.appveyor.com/api/projects/status/qjvbk6u86sp6cm59?svg=true)](https://ci.appveyor.com/project/acgetchell/cdt-plusplus) [![Join the chat at https://gitter.im/acgetchell/CDT-plusplus](https://img.shields.io/badge/gitter-join%20chat%20→-brightgreen.svg)](https://gitter.im/acgetchell/CDT-plusplus?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

**Quantize spacetime on your laptop.**

For an introduction to [Causal Dynamical Triangulations](https://github.com/acgetchell/CDT-plusplus/wiki), including the foundations and recent results, please see the [wiki](https://github.com/acgetchell/CDT-plusplus/wiki).

[Causal Dynamical Triangulations][1] in C++ using the
[Computational Geometry Algorithms Library][cgal] and [Eigen][eigen]>3.1.0, compiled
with [CMake][cmake] using [Clang][clang]/[LLVM][llvm].
Arbitrary-precision numbers and functions by [MPFR][mpfr] and [GMP][gmp].
Option-types via [Boost][boost] >= 1.63.0.
Uses [Docopt][docopt] to provide a beautiful command-line interface, and [Howard Hinnant's date and timezone][date] library for accurate time (both as [subrepos][git-subrepo]).
[Gmock 1.8][gmock] may be optionally installed in order to build/run unit tests.
[Ninja][ninja] is a nice (but optional) replacement for `make`.
Intel's [TBB][tbb] provides significantly better performance if present (3x+).
Follows the [CppCore Guidelines][guidelines] as enforced by [clang-tidy][tidy], which you can run using the 
[clang-tidy.sh][8] script:

~~~
./clang-tidy.sh
~~~

(Or use your favorite linter plugin for your editor/IDE.)

The goals and targets of this project are:

- [x] Developed with [literate programming][12] generated using [Doxygen][doxygen]
- [x] [Efficient Pure Functional Programming in C++ Using Move Semantics][36]
- [x] Validation tests using [CTest][10]
- [x] Unit tests with [Gmock][gmock]
- [x] Test builds with [Travis CI][travis]
- [x] 3D Simplex
- [x] 3D Spherical triangulation
- [x] 2+1 foliation
- [x] Integrate [docopt][docopt] CLI
- [x] S3 Bulk action
- [x] 3D Ergodic moves
- [x] Multithreading via [TBB][tbb]
- [x] Metropolis algorithm
- [ ] Output via [HDF5][hdf5]
- [ ] 4D Simplex
- [ ] 4D Spherical triangulation
- [ ] 3+1 foliation
- [ ] S4 Bulk action
- [ ] 4D Ergodic moves
- [ ] Initialize two masses
- [ ] Shortest path algorithm
- [ ] Einstein tensor
- [ ] Complete test coverage
- [ ] Complete documentation
- [ ] ???
- [ ] (Non)profit

### Prerequisites ###
------

[CDT++][38] should build on any system (e.g. Linux, MacOS, Windows) with
[CMake][14], [CGAL][cgal], [Boost][boost], [MPFR][mpfr], and [Eigen][eigen] installed. 
[TBB][tbb] provides an optional (but significant) speed boost.  [Gmock][gmock] is 
optional (but recommended) for running the unit tests, and [Ninja][ninja] is an 
optional replacement for `make` which provides quick parallel builds of the 
unit tests.

On MacOS, the easiest way to do this is with [HomeBrew][homebrew]:

~~~
brew upgrade cmake
brew install ninja
brew upgrade boost
brew install eigen
brew install tbb --c++11
brew install cgal --imaging --with-eigen3 --with-lapack
~~~

On Ubuntu, you will need an updated versions of [clang][clang] or [gcc][gcc], 
[CMake][cmake], and [Boost][boost], which is scripted in [.travis.yml][39].
With Howard Hinnant's [date][date] library, you may also need to install
`libcurl-dev`, which is a virtual package with many flavors, OpenSSL being
the most tried and tested.

### Build ###
------
This project uses a separate `build/` directory, which allows you to rebuild the
project without cluttering the source code. Thus, download this source code
(clone this repo from GitHub or grab a zipped archive [here][17]) and run the
following commands in the top-level directory:

~~~
mkdir build && cd build
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
~~~

(Or run [build.sh][27] if you have [Ninja][ninja] installed.)

This should result in the main program executable, `cdt` in the `build/`
directory, along with several others.

* `cdt-gv` converts output files to GeomView format for visualization
* `cdt-opt` is a simplified version with hard-coded inputs, mainly useful for 
debugging and scripting

If you build [GMock][gmock] unit tests, the executable
`unittests` will also be present. See [Tests](#tests) for details.

For some versions of Linux, you may have to build [CGAL][cgal] from source.
Follow the instructions (or their equivalent) given in the install section
of the [.travis.yml][39] build file.

There are enough unit tests that it's worthwhile doing fast parallel builds.
[Ninja][ninja] is just the ticket. It's effectively a drop-in replacement for
`make`, and works nicely because [CMake][cmake] generates the build files.
There's quite a difference in speed. CMake also [abstracts away the build][47]
tool nicely. Again, see [build.sh][27] for an example.

Possible build troubles:
-----------------------
* While running `build.sh` under linux with `gcc` you may encounter 
`virtual memory exhausted: Cannot allocate memory`, a common solution for
which is to
[allocate swap space](http://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/).

### Usage ###
------
CDT-plusplus uses [docopt][docopt] to parse options from the help message, and so
understands long or short argument formats, provided the short argument given
is an unambiguous match to a longer one. The help message should be instructive:

~~~
./build/cdt --help
Causal Dynamical Triangulations in C++ using CGAL.

Copyright (c) 2014-2017 Adam Getchell

A program that generates d-dimensional triangulated spacetimes
with a defined causal structure and evolves them according
to the Metropolis algorithm. Specify the number of passes to control
how much evolution is desired. Each pass attempts a number of ergodic
moves equal to the number of simplices in the simulation.

Usage:./cdt (--spherical | --toroidal) -n SIMPLICES -t TIMESLICES [-d DIM] -k K --alpha ALPHA --lambda LAMBDA [-p PASSES] [-c CHECKPOINT]

Examples:
./cdt --spherical -n 64000 -t 256 --alpha 0.6 -k 1.1 --lambda 0.1 --passes 1000
./cdt --s -n64000 -t256 -a.6 -k1.1 -l.1 -p1000

Options:
  -h --help                   Show this message
  --version                   Show program version
  -n SIMPLICES                Approximate number of simplices
  -t TIMESLICES               Number of timeslices
  -d DIM                      Dimensionality [default: 3]
  -a --alpha ALPHA            Negative squared geodesic length of 1-d
                              timelike edges
  -k K                        K = 1/(8*pi*G_newton)
  -l --lambda LAMBDA          K * Cosmological constant
  -p --passes PASSES          Number of passes [default: 100]
  -c --checkpoint CHECKPOINT  Checkpoint every n passes [default: 10]
~~~

The dimensionality of the spacetime is such that each slice of spacetime is
`d-1`-dimensional, so setting `d=3` generates 2 spacelike dimensions and one
timelike dimension, with a defined global time foliation. Thus a
`d`-dimensional simplex will have some `d-1` sub-simplices that are purely
spacelike (all on the same timeslice) as well as some that are timelike
(span two timeslices). In [CDT][1] we actually care more about the timelike
links (in 2+1 spacetime) and the timelike faces (in 3+1 spacetime).

### Documentation ###
--------------
Online documentation may be found at http://www.adamgetchell.org/CDT-plusplus/

If you have [Doxygen][doxygen] installed you can generate the same information
locally by simply typing at the top level directory
([Doxygen][doxygen] will recursively search):

~~~
doxygen
~~~

This will generate the `html/` directory which will contain
documentation generated from CDT++ source files. `USE_MATHJAX` has been enabled
in [Doxyfile][41] so that the LaTeX formulae can be rendered in the html
documentation using [MathJax][mathjax]. `HAVE_DOT` is set to **YES** which allows
various graphs to be autogenerated by [Doxygen][doxygen] using [GraphViz][graphviz].
If you do not have GraphViz installed, set this option to **NO**
(along with UML_LOOK).

### Tests ###
-----------
[GMock][gmock] is optional, but strongly recommended if you want to make changes to
or understand the source code in detail. Building the [GMock][gmock] `unittests`
executable is set by the `GMOCK_TESTS` variable in [CMakeLists.txt][28], or
at the command line using:

~~~
cmake -DGMOCK_TESTS:BOOL=ON -DCMAKE_BUILD_TYPE=Debug ..
~~~

GMock is setup and installed by CMake using the [technique described here.][cmake-gtest] Consult [CMakeLists.txt][28] for details.

Unit tests using GMock are then run (in the `build/` directory) via:

~~~
./unittests
~~~

You can also run function tests (using [CTest][ctest]) via:

~~~
cmake --build . --target test
~~~

In addition to the command line output, you can see detailed results in the
`build/Testing` directory which is generated thereby.

### Static Analysis ###
-----------
The [cppcheck-build.sh][35] script runs a quick static analysis using
[cppcheck][cppcheck].

[Clang][clang] comes with [scan-build][32] which can run a much more thorough,
but slower static analysis integrated with [CMake][cmake] and [Ninja][ninja].
Simply run the [scan.sh][33] script.

Also, these tools build in **DEBUG** mode. You should probably not then run
`unittests` as you will get thousands of lines of debugging output from the
tests that create large triangulations. (You could use `--gtest_filter`
to run just the tests that you want.)

One of the [Travis-CI][travis] jobs runs [Valgrind][valgrind]; be sure to look at the 
results to ensure you're not leaking memory.

### Contributing ###

Please see CONTRIBUTING.md and our CODE_OF_CONDUCT.md.

[1]: http://arxiv.org/abs/hep-th/0105267
[cgal]: http://www.cgal.org
[cmake]: http://www.cmake.org
[clang]: http://clang.llvm.org
[llvm]: http://llvm.org
[gmock]: https://github.com/google/googletest/tree/master/googlemock
[guidelines]: http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines
[8]: https://github.com/acgetchell/CDT-plusplus/blob/master/clang-tidy.sh
[9]: https://code.google.com/p/cgal-bindings/]
[ctest]: http://cmake.org/Wiki/CMake/Testing_With_CTest
[travis]: http://about.travis-ci.org/docs/user/getting-started/
[12]: http://www.literateprogramming.com
[doxygen]: http://www.doxygen.org
[14]: http://www.cmake.org/cmake/help/install.html
[cgal]: http://www.cgal.org/Manual/latest/doc_html/installation_manual/Chapter_installation_manual.html
[homebrew]: http://brew.sh
[17]: https://github.com/acgetchell/CDT-plusplus/archive/master.zip
[ninja]: https://martine.github.io/ninja/
[docopt]: https://github.com/docopt/docopt.cpp
[mathjax]: http://www.mathjax.org
[graphviz]: http://www.graphviz.org
[22]: http://scipher.wordpress.com/2010/05/10/setting-your-pythonpath-environment-variable-linuxunixosx/
[24]: https://github.com/google/googletest/blob/master/googlemock/README.md
[eigen]: http://eigen.tuxfamily.org/index.php?title=Main_Page
[26]: http://public.kitware.com/pipermail/cmake-developers/2011-November/002490.html
[27]: https://github.com/acgetchell/CDT-plusplus/blob/master/build.sh
[28]: https://github.com/acgetchell/CDT-plusplus/blob/master/CMakeLists.txt
[mpfr]: http://www.mpfr.org
[gmp]: https://gmplib.org
[hdf5]: http://www.hdfgroup.org/HDF5/
[32]: http://clang-analyzer.llvm.org/scan-build.html
[33]: https://github.com/acgetchell/CDT-plusplus/blob/master/scan.sh
[cppcheck]: http://cppcheck.sourceforge.net
[35]: https://github.com/acgetchell/CDT-plusplus/blob/master/cppcheck-build.sh
[36]: http://blog.knatten.org/2012/11/02/efficient-pure-functional-programming-in-c-using-move-semantics/
[tbb]: https://www.threadingbuildingblocks.org
[38]: https://github.com/acgetchell/CDT-plusplus
[39]: https://github.com/acgetchell/CDT-plusplus/blob/master/.travis.yml
[41]: https://github.com/acgetchell/CDT-plusplus/blob/master/Doxyfile
[boost]: http://www.boost.org
[gcc]: https://gcc.gnu.org/
[contrib]: https://github.com/acgetchell/CDT-plusplus/blob/master/CONTRIBUTING.md
[tidy]: http://llvm.org/releases/3.6.0/tools/clang/docs/ClangFormatStyleOptions.html
[47]: https://crascit.com/2016/04/03/scripting-cmake-builds/
[valgrind]: http://valgrind.org/docs/manual/quick-start.html#quick-start.mcrun
[conduct]: https://github.com/acgetchell/CDT-plusplus/blob/master/CODE_OF_CONDUCT.md
[date]: https://howardhinnant.github.io/date/date.html

[cmake-gtest]: https://crascit.com/2015/07/25/cmake-gtest/

[git-subrepo]: https://github.com/ingydotnet/git-subrepo