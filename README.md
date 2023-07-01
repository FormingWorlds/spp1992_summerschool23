# Instructions for Friday session at SPP 1992 Summer School 2023

Website to Summer School: https://summerschool2023.spp1992-exoplanetdiversity.de

In the hands-on session on Friday (2023-08-04) we will make use of the SPIDER code (https://github.com/djbower/spider). Below you find a shortened subset of SPIDER installation instructions. Please follow the installation and test routines to make sure you have a running SPIDER version on your Mac or Linux machine for the workshop (reach step 5 of *Test your installation* and compare with the produced image). The setup is written primarily for MacOS (ARM: M1/M2 Macs, or Intel processors). If you run into problems, please first check out *Troubleshooting* at the very bottom. If that does not help, please write to tim.lichtenberg@rug.nl and describe your issue.

## Code Installation

First, check out *Software dependencies* below to make sure you have a working C and Python 3 installation. Next we install PETSc which provides the solver library and then we install SPIDER.  Finally, we can (optionally) install a [test harness](https://github.com/sciath/sciath).

1. Test you have a valid C compiler installed by running the following command in a terminal window (install a C compiler if this command fails):

    ```
    echo '#include<stdio.h>' > t.c && echo 'int main(){printf("It seems to work!\n");}' >> t.c && gcc t.c && ./a.out && rm -f t.c a.out
    ```

2. If you are already a user of PETSc, comment out any existing references to `PETSC_DIR` and `PETSC_ARCH` in your`.profile`, `.bash_profile`,`.bashrc`, etc. and clear these variables from your current terminal session.


3. Download PETSc.  Use this specific version as given below if interested in reproducing previous results, though any version of PETSc greater than or equal to 3.17 is expected to work just as well.

    ```
    cd /somewhere/to/install
    git clone https://gitlab.com/petsc/petsc -b main petsc-double
    cd petsc-double
    git checkout 63b725033a15f75ded7183cf5f88ec748e60783b
    ```
4. Configure PETSc:

    ```
    ./configure --with-debugging=0 --with-fc=0 --with-cxx=0 --with-cc=gcc --download-sundials2 --download-mpich --COPTFLAGS="-g -O3" --CXXOPTFLAGS="-g -O3"
    ```

    Note: if you use an ARM CPU (on newer Apple laptops, such as M1 or M2), you may receive an error at this step if you have installed Python via anaconda. Please check out the Software dependencies section below for handling this.

5. Follow the terminal instructions to complete the installation of PETSc.  You can copy-paste the commands returned to you in the terminal window.  Make note of `PETSC_DIR` and `PETSC_ARCH`, which are also reported in the terminal window.

6. In your environment, set `PETSC_DIR` and `PETSC_ARCH` to the PETSc installation.  `PETSC_ARCH` will look something like arch-xxx-yyy (e.g. arch-darwin-c-opt).  This completes the setup of PETSc:

    ```
    export PETSC_DIR=/somewhere/to/install/petsc-double
    export PETSC_ARCH=arch-xxx-yyy
    ```

7. Now download SPIDER:

    ```
    cd /somewhere/to/install
    git clone https://github.com/djbower/spider.git
    cd spider
    ```

8. Make SPIDER:

    ```
    make clean
    make -j
    ```

    SPIDER is now installed and you can in principle skip to *Test your installation* below.  However, you are advised to install the test harness as follows:

9. [Optional] Get SciATH (Scientific Application Test Harness), which is a Python module:

    ```
    cd /somewhere/to/install
    git clone https://github.com/sciath/sciath -b dev
    ```

10. [Requires SciATH] Add the resulting module to your Python path (for example):

    ```
    export PYTHONPATH=$PYTHONPATH:$PWD/sciath
    ```

11. [Requires SciATH] Now return to the root SPIDER directory and test basic functionality:

    ```
    make test
    ```

12. [Requires SciATH] You can also run all available tests by navigating to the `tests/` directory and running:

    ```
    python -m sciath tests.yml
    ```

You should now be ready to use the code.  Proceed to *Running a Model* to learn how to run a basic model and use SPIDER options files.

## Test your installation

1. [Optional] Add the installation directory of SPIDER to your `$PATH` so you can call the `spider` binary without requiring the absolute path.
2. You can run `spider` without an argument and standard parameters will be used to run a basic interior model with output stored in ```output/```:

    ```
    spider
    ```

3. However, in general you will add an argument to use the parameters specified in an options file.  For example, from the root SPIDER directory:

    ```
    spider -options_file tests/opts/blackbody50.opts
    ```

4. There are example options files in ```tests/opts/```.  Also see ```parameters.c``` for more parameter options.

5. A python script ```py/plot_spider_lite.py``` can be used to generate a basic figure of the interior profiles.  Run ```py/plot_spider_lite.py -h``` to see the optional arguments.  Running the script on the output data of ```blackbody50.opts``` will generate the following plot:

![blackbody50-interior](https://github.com/FormingWorlds/spp1992_summerschool23/assets/8751016/1408834a-826d-485d-b138-187f54df887e)


## Software dependencies

Installation of SPIDER and its dependencies requires:

1. A working C compiler
2. Build dependencies
3. Build SPIDER

#### C compiler

Any C compiler can be used if you want to build SPIDER for double precision calculations.  Unfortunately on a Mac, "`gcc`" is a wrapper for the default Apple compiler ("`clang`") which is not actually a GCC compiler package and hence does not support quadruple precision math.  Particularly for Mac OSX you may choose to install GCC to support both double and quadruple precision calculations using a single compiler.

A basic test to ensure you have a working compiler is (note you can swap out "`gcc`" for another compiler binary name):

  ```
  echo '#include<stdio.h>' > t.c && echo 'int main(){printf("It seems to work!\n");}' >> t.c && gcc t.c && ./a.out && rm -f t.c a.out
  ```

For Mac OSX, the easiest way to install GCC is from <http://hpc.sourceforge.net>

If you use MacPorts, Homebrew (https://brew.sh/), or apt, these can also be used to install GCC.  *Unfortunately, the GCC compiler from Macports seems to be frequently broken*.  Nevertheless, for example, using MacPorts:

  ```
  sudo port install gcc8
  ```

This will install a set of GCC binaries, typically in "`/opt/local/bin`".  The C compiler (for GCC8) will be called "`gcc-mp-8"` where the mp is obviously clarifying that it was installed by MacPorts.
Now on a Mac, you will usually access the default Apple compiler ("`clang`") using "`gcc`", but you can easily access the MacPorts GCC you just installed by using "`gcc-mp-8`" instead.  *So when you are installing the software in the next sections, just use "`gcc-mp-8`" instead of "`gcc`" when you are asked to specify the C compiler.*

You can check the version of your compiler (for example):

  ```
  gcc --version
  gcc-mp-8 --version
  ```

If you see a message about "Apple LLVM" then your compiler will not support quadruple precision.  You can also test with this command (replace `gcc` by `gcc-mp-8` to test the MacPorts compiler):

  ```
  echo '#include<stdio.h>' > t.c && echo '#include<quadmath.h>' >> t.c && echo 'int main(){printf("It seems to work!\n");}' >> t.c && gcc t.c && ./a.out && rm -f t.c a.out
  ```
#### Python environment

You need Python 3.10+. In order to test your Python version, type:
  ```
  python --version
  ```

If do not have Python installed, there are two options. We recommend Option A, because Option B may break on newer Apple Silicon (ARM) machines.
 
 * Option A (*recommended*): using the `brew` package manager
    
    * The following steps assume ``brew`` (if not, follow: https://brew.sh/) is installed on your system.
    
    * Delete all traces of a potential Anaconda package manager installation from your system. 
        
        * To do this, follow the steps at https://docs.anaconda.com/free/anaconda/install/uninstall/
        
        * Delete all Anaconda-related entries from your ``~/.bash_profile`` (Intel) or ``~/.zshrc`` (ARM)
    
    * Install Python via ``brew``:
      
         
          brew install python
          
        
        * Update to the latest stable version:
          ```
          brew upgrade python
          ```
        
        * Refresh your shell:
            * ARM:
              ```
              source ~/.zsrhrc
              ```
            
            * Intel:
              ```
              source ~/.bash_profile
              ```
        
        * Install all other necessary packages: 
          ```
          pip3 install matplotlib pandas netcdf4 matplotlib numpy pandas scipy sympy natsort
          ```
        
        * Make the new Python version the system default (check what `brew` tells you during/after the `brew install python` step), by adding the following to your:
            
            * ``~/.zsrhrc`` (ARM):
              ```
              export PATH="/opt/homebrew/opt/python/libexec/bin:$PATH"
              ```
            
            * ``~/.bash_profile`` (Intel): 
              ```
              export PATH="/usr/local/opt/python/libexec/bin:$PATH"
              ```
 
 * Option B: Using the ``anaconda`` package manager (be careful, this potentially breaks the PETSc installation on ARM)
    
    * Install ``conda``:
        * Download the appropriate Miniconda installer from https://docs.conda.io/en/latest/miniconda.html#id36
        * Create a conda environment for PROTEUS:
          ```
          conda create -n proteus python=3.10.9   
          conda activate proteus
          conda install netcdf4 matplotlib numpy pandas scipy sympy natsort
          conda install -c conda-forge f90nml
          ```
    
    * Refresh your shell:
            
      * ARM:
      
        ```
        source ~/.zsrhrc
        ```
    
      * Intel:
      
        ```
        source ~/.bash_profile
        ```

## Troubleshooting

We have compiled a list of potential problems during the installation here: https://proteus-code.readthedocs.io/en/latest/installation.html#troubleshooting.
Please check out this list and compare with your error if you run into a dependency or installation problem. If that does not solve your problem please write to tim.lichtenberg@rug.nl.
