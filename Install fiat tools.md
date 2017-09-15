Environment: macOS Sierra 10.12 + `astroconda` + `homebrew`

Following the instruction on http://dls.physics.ucdavis.edu/fiat/fiat.html

1. Install miniconda
  ```
$ wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh -O miniconda.sh
$ bash miniconda.sh -b -p $HOME/miniconda
$ export PATH="$HOME/miniconda/bin:$PATH"
$ conda config --set always_yes yes --set changeps1 no
$ conda update -q conda
```

2. Install astroconda
```
$ conda config --add channels http://ssb.stsci.edu/astroconda
$ conda create -n astroconda stsci
$ source activate astroconda
```

3. Install homwbrew
```
$ /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

4. Download fiat tools from http://dls.physics.ucdavis.edu/fiat/fiatdls.tgz

5. Unpack the file
```
$ tar -zxvf fiatdls.tgz && cd fiat
```

6. Install fiat tools
```
$ make -f makefile.dls
```

It may give some warnings/errors when compiling the .c files. You may need to fix the stderr/instruction parts in some source codes.

This will install everything in the "bin" subdirectory. There will also be a fiat.h and libfiat.a for C programmers. The binary files can be copied into /usr/local/bin or any directories included in $PATH.

---
Now, the basic fiat tools are installed. This does not include fiatreview. fiatreview requires `PGPLOT`, the `cfitsio` library, and the `wcstools` library.

We already have cfitsio lib in astroconda environment. Also it is possible to use homebrew to install cfitsio: `$ brew install cfitsio`.

* To install `wcstools`: 
```
$ source activate astroconda
$ conda install wcstools
```

* To Install `PGPLOT`:
```
$ brew install pgplot
```

* Download this tarball: [fiatreview.tgz](http://dls.physics.ucdavis.edu/fiat/fiatreview.tgz). This unpacks into the `fiat` directory. 

You will probably have to fiddle with the makefile to get it it to link with all the libraries, especially `pgplot` which seems to have different locations on different machines. Also make sure `libfiat.a` is linked to.

* Modify the stderr part (~line 102) of `fiatreview.c`

* Add `#include <math.h>` and `#include <float.h>` to `cat.c`

* Modify the makefile:
```
PGPLOT = -I/usr/local/include -L/usr/local/lib -lcpgplot -lpgplot -lm
 
PGPLOTINC = -I/usr/local/include
 
FITSIO = -lcfitsio
 
CC = gcc -Wall -g
 
fiatreview: fiatreview.o fiatreview.h draw.o image.o cat.o
$(CC) -o fiatreview fiatreview.o draw.o image.o cat.o -lfiat $(FITSIO) $(PGPLOT)
#        mv fiatreview ../../bin
 
cat.o: cat.c fiatreview.h
$(CC) -c cat.c $(PGPLOTINC)
 
draw.o: draw.c fiatreview.h
$(CC) -c draw.c $(PGPLOTINC)
 
fiatreview.o: fiatreview.c fiatreview.h
$(CC) -c fiatreview.c $(PGPLOTINC)
 
image.o: image.c fiatreview.h
$(CC) -c image.c $(PGPLOTINC)
 
 
plotppm: plotppm.c
$(CC) -o plotppm plotppm.c $(PGPLOT)
 
ppmfits: ppmfits.c
$(CC) -o ppmfits ppmfits.c $(FITSIO) -lm
 
fiatreview.tgz: 
/usr/bin/tar czvf fiatreview.tgz fiatreview.c fiatreview.h cat.c draw.c image.c makefile
```

Then
```
$ make
```
=> gcc -Wall -g -c fiatreview.c -I/usr/local/include

=> gcc -Wall -g -o fiatreview fiatreview.o draw.o image.o cat.o -lfiat -lcfitsio -I/usr/local/include -L/usr/local/lib -lcpgplot -lpgplot -lm

==> fiatreview

```
$ cp fiatreview /usr/local/bin
```

**Issue**: when using `-` to zoom out or using `n` to go to the next object, the program quits with `Abort trap: 6`.

