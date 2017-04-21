# sunrise-for-tipsy

[Sunrise](https://bitbucket.org/lutorm/sunrise/overview) is a popular 
"Monte-Carlo Radiation Transfer code for calculating absorption and 
scattering of light in astrophysical situations". 
Unfortunately, it is notoriously finnicky to get installed, as it relies 
on specific point release versions of nearly a dozen different libraries. 
[Ben Keller](https://github.com/bwkeller) introduces a way to build 
Sunrise on [Docker](https://docs.docker.com/) in his blog. However, his 
instruction require root permission which does not work for people who 
is working on the server. 
By referring [Sunrise's wiki](https://bitbucket.org/lutorm/sunrise/wiki/Compiling) 
and [Ben Keller's blog](http://www.physics.mcmaster.ca/~kellerbw/posts/sunrise-on-docker.html),
this instruction will build Sunrise as a normal user at your home directory 
on server.

## Prerequisites

### ICC compiler

Unfortunately, sunrise will not compile with any free compilers. You will 
need to get the Intel C++ compil on your server. 
If the ICC compiler does not work on your server, you can get it from intel
and you may need to pay for it though and you have to consume your precious
time on installing it. If you do not want to spend time on this step, 
please contact the administrator to solve this problem.

### Set up Mercurial

Mercurial is a free, distribute source control management tool like git. In the
following instructions, you need it to clone projects rather than git so that
Mercurial is highly recommended to install.

You can find full descriptions for Mercurial setting up on Mercurial's 
[webpage](https://www.mercurial-scm.org/wiki/Download). 
Since Mercurial is written in Python, it is much easier to get Mercurial working
on your server by using pip or conda as a result, just execute 
```
pip install Mercurial
```
or 
```
conda install -c conda-forge mercurial
```
This shoudl work on all platform of Linux, even on OSX and Windows. If you meet
the permission problem by using pip, you can add the option `--user` to have a
try.

### Getting the libraries

Now we need to begin by fetching all of the varied specific versions of libraries
that sunrise relies on.

#### Boost
Download Boost from SourceForge, note that Sunrise requires the version 1.48.0
or later, and version 1.48.0 is confirmed to work, later version may cause problems.
You can fetch version 1.48.0 from 
[boost archive](http://www.boost.org/users/history/version_1_48_0.html).
Cd to the directory where you put the distribution, then patch this ancient 
version of boost so it compiles with a modern compiler:
```
wget --no-check-certificate https://svn.boost.org/trac/boost/raw-attachment/ticket/6165/libstdcpp3.hpp.patch
patch -p0 < libstdcpp3.hpp.patch
for file in `grep -r -l TIME_UTC *`; do sed -i -e 's/TIME_UTC/TIME_UTC_/' $file; done
```
Build it:
```
sed -i -e 's/using intel-linux/using intel-linux : : : <compileflags>-shared-intel <linkflags>-shared-intel/' project-config.jam
echo "using mpi : /opt/intel/impi/5.0.3.048/intel64/bin/mpicc ;" > user-config.jam
./bootstrap.sh --without-libraries=python --with-toolset=intel-linux --prefix=$HOME --libdir=$HOME/lib/boost_1_48_0 --includedir=$HOME/include/boost_1_48_0
./b2 -j<n> --user-config=user-config.jam install
```
This will copy the Boost headers to `$HOME/include/boost_1_48_0/boost` and will build 
the libraries and put them in `$HOME/lib/boost_1_48_0`, you can change them to any 
directories you want to put the headers and libraries at the options of `./bootstrap.sh`.
Also, `<n>` should be some reasonable number depending on how many cores your machine has, 
because building Boost in parallel greatly decreases the build time.

If you use the above commands, Boost will install its include files in a directory
`$PREFIX/include/boost-<version>/boost`. For the files to be found without specifically
adding that directory to the include directives, you can do
```
ln -s $PREFIX/include/boost-<version>/boost $PREFIX/include/boost.

```
alternatively, if you do want to make a symbolic link, you can do
```
export LD_LIBRARY_PATH=$PREFIX/include/boost-<version>:$LD_LIBRARY_PATH
```

#### Blitz

First clone the Mercurial [repository](http://blitz.hg.sourceforge.net/hgweb/blitz/blitz)
and checkout the ab84372f3dce branch which is confirmed to work:
```
hg clone http://blitz.hg.sourceforge.net/hgweb/blitz/blitz
cd blitz/
hg checkout ab84372f3dce
```
Build it:
```
autoreconf -fiv
export CXX=icpc
export CC=icc
export CXXFLAGS='-O2 -g -pthread -shared-intel'
export CFLAGS='-O2 -pthread -shared-intel'
export LDFLAGS='-pthread -shared-intel'
 ./configure --prefix=$HOME --enable-threadsafe --disable-cxx-flags-preset --enable-serialization 
make lib
make install
```
If you get errors related to not finding the boost libraries try using 
`--with-boost-libdir=$HOME/lib/boost_1_48_0/` (or wherever you installed boost).

#### cfitsio

CFITSIO is a library of C and Fortran subroutines for reading and writing data 
files in FITS (Flexible Image Transport System) data format.
You can find the latest version at 
[NASA software archive](https://heasarc.gsfc.nasa.gov/fitsio/fitsio.html) and
fetch it:
```
wget http://heasarc.gsfc.nasa.gov/FTP/software/fitsio/c/cfitsio_latest.tar.gz
```
Untar the file and do:
```
export CFLAGS='-pthread -O3'
export CC=icc
./configure --prefix=$HOME
make
make install
```

#### CCfits

CCfits is an object oriented interface to the cfitsio library. It is designed 
to make the capabilities of cfitsio available to programmers working in C++. 
It is written in ANSI C++ and implemented using the C++ Standard Library with 
namespaces, exception handling, and member template functions.
The download link is on 
[NASA sofrware archive](https://heasarc.gsfc.nasa.gov/fitsio/CCfits/)
and fetch it:
```
wget https://heasarc.gsfc.nasa.gov/fitsio/CCfits/CCfits-2.5.tar.gz
```
Extract sources to a directory named CCfits (otherwise there will be trouble..)
```
export CXXFLAGS='-pthread -O3'
./configure --prefix=$HOME --with-cfitsio=$HOME
make
make install
```

#### libPJutil

First get libPJutil from bitbucket by doing
```
hg clone https://bitbucket.org/lutorm/libpjutil
```
It is usually fine to use the most recent version of this, as it doesn't change 
much. Then go into this directory and do:
```
autoreconf -fiv
export CPPFLAGS=-I$HOME/include (or wherever you need it to point to find blitz; also use "-pthread" if you use it for blitz)
export CXXFLAGS="-O2 -pthread"
export LDFLAGS=-L$HOME/lib (or wherever)
./configure --prefix=$HOME  (if boost is in $HOME/include and $HOME/lib)
make
make install
```
This library now contains an embedded copy of the GNU "units" program for units 
conversion. For this to work, the path to the file "units.dat" needs to be 
defined in the environment variable UNITSFILE. On standard linux systems, this 
file is in /usr/share/, so in that case you would add
```
export UNITSFILE=/usr/share/units.dat
```
to your bash_profile. If you don't have GNU units installed, you need to download 
it and put this file somewhere. NB. As of v2.0, this file is no longer included, 
so you must download v1.88.

## Building Sunrise

Get sources from the [repository](https://bitbucket.org/lutorm/sunrise) with git:
```
git clone https://bitbucket.org/lutorm/sunrise.git
```
In general you should check out the current version, which is mentioned on the 
front page, not the trunk. The trunk may be unstable at any given time.
Then build it:
```
autoreconf -fiv
export CPPFLAGS="-I$HOME/include -I$HOME/include/libPJutil" (yes, you need to separately specify the libPJutil directory, sorry)
export CFLAGS="-O2 -pthread"
export CXXFLAGS="-ggdb  -O3  -pthread"
export LDFLAGS="-pthread -L$HOME/lib"
export CXX=icpc
./configure --prefix=$HOME --with-boost=$HOME --with-boost-thread=mt --with-boost-program-options=mt
```
Finally, it's time to compile:
```
make
make install
```
If you find the executables **sfrhist**, **mcrx** and **broadband** in the 
subdirectory `src`, congralutions, you have done to make Sunrise work
on your server to produce mock observational images from Tipsy format
simulation data.

## Making mock observations

To make a mock observation images from the snapshots with tipsy units (e.g.
Gasoline/ChaNGa), You should install **pynbody** and compile **smooth** and 
**std2ascii** on your server.

### pynbody

Pynbody is a light-weight, portable, format-transparent analysis framework 
for N-body and hydrodynamic astrophysical simulations supporting 
PKDGRAV/Gasoline, Gadget, N-Chilada, and RAMSES AMR outputs.
You can find its installation guide and tutorials on its 
[website](http://pynbody.github.io/pynbody/tutorials/tutorials.html).

### smooth

You can find [**smooth**](https://github.com/N-BodyShop/smooth) on 
N-bodyShop's GitHub page, and fetch it by doing:
```
git clone https://github.com/N-BodyShop/smooth
```
to build:
```
make
```
The **smooth** executable should exist in the directory.

### std2ascii

**std2ascii** is one executable in 
[tipsy_tools](https://github.com/N-BodyShop/tipsy_tools)
on N-bodyShop's GitHub page, and fetch it by doing:
```
git clone https://github.com/N-BodyShop/tipsy_tools
```
Before build the whole set, remember to add **std2ascii** to `TOOLS` variable
```make 
TOOLS = ascii2bin bin2ascii totipnat totipstd snapshot std2ascii
```
and add a line for **std2ascii**
```make
td2ascii: std2ascii.o
        $(CC) $(CFLAGS) -o std2ascii std2ascii.o -lm $(LIBS)
```
as **std2ascii** is not the default executable produced by *tipsy_tools*.
Then build it:
```
make
```
The **std2ascii** associated with other five executables are in the directory.

### Scripts producing mock observations

It is the time to fetch scripts in this repository which created by 
[greg stinson](https://github.com/stinsong4100) who hacked away at converting
old idl scripts to python to produce your mock
observations. I recommend you clone this repository to your local directory
and make symbolic links for all scripts in the directory you want to make an
image of and move the executables **sfthist**, **mcrx**, **broadband**,
**smooth** and **std2ascii** to an identical directory, then change the path
to the variable `SR` in *sunrise.sh*.

If you have everything well installed, make sunrise images is as easy as these
one line:
```
nohup ./sunrise.sh &
```
or if you want to align the coordinates with the spin of stars, just doing
```
nohup ./sunrise.sh -s &
```
The images will be called *edge.png* and *face.png*. Enjog!
![Image of galaxy](./edge.png)
![Image of galaxy](./face.png)


## Troubleshooting

If the installation fails with the configuration with the message 
```
configure: error: Could not find a version of the boost_system library!
```
you can also use `--with-boost-libdir=$Home/lib/boost_1_48_0 (or whatever)` 
to help the compiler find the libraries.

