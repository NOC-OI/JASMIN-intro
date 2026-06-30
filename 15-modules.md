---
title: Accessing software via Modules
teaching: 30
exercises: 15
---



::::::::::::::::::::::::::::::::::::::: objectives

- Load and use a software package.
- Explain how the shell environment changes when the module mechanism loads or unloads packages.

::::::::::::::::::::::::::::::::::::::::::::::::::

:::::::::::::::::::::::::::::::::::::::: questions

- How do we load and unload software packages?

::::::::::::::::::::::::::::::::::::::::::::::::::

On a high-performance computing system, it is seldom the case that the software
we want to use is available when we log in. It is installed, but we will need
to "load" it before it can run.

Before we start using individual software packages, however, we should
understand the reasoning behind this approach. The three biggest factors are:

- software incompatibilities
- versioning
- dependencies

Software incompatibility is a major headache for programmers. Sometimes the
presence (or absence) of a software package will break others that depend on
it. Two well known examples are Python and C compiler versions.
Python 3 famously provides a `python` command that conflicts with that provided
by Python 2. Software compiled against a newer version of the C libraries and
then run on a machine that has older C libraries installed will result in an
opaque `'GLIBCXX_3.4.20' not found` error.

Software versioning is another common issue. A team might depend on a certain
package version for their research project -- if the software version was to
change (for instance, if a package was updated), it might affect their results.
Having access to multiple software versions allows a set of researchers to
prevent software versioning issues from affecting their results.

Dependencies are where a particular software package (or even a particular
version) depends on having access to another software package (or even a
particular version of another software package). For example, the VASP
materials science software may require a particular version of the
FFTW (Fastest Fourier Transform in the West) software library available for it
to work.

## Environment Modules

Environment modules are the solution to these problems. A *module* is a
self-contained description of a software package -- it contains the
settings required to run a software package and, usually, encodes required
dependencies on other software packages.

There are a number of different environment module implementations commonly
used on HPC systems: the two most common are *TCL modules* and *Lmod*. Both of
these use similar syntax and the concepts are the same so learning to use one
will allow you to use whichever is installed on the system you are using. In
both implementations the `module` command is used to interact with environment
modules. An additional subcommand is usually added to the command to specify
what you want to do. For a list of subcommands you can use `module -h` or
`module help`. As for all commands, you can access the full help on the *man*
pages with `man module`.

On login you may start out with a default set of modules loaded or you may
start out with an empty environment; this depends on the setup of the system
you are using.

### Listing Available Modules

To see available software modules, use `module avail`:


```bash
[yourUsername@sci-vm-02 ~]$ module avail | less
```

```output
------------------------------------------------------------------------------------------- /apps/jasmin/modulefiles --------------------------------------------------------------------------------------------
   ants/2.1.0                                 contrib/arsf/arsf_dem_scripts/0.1.8        contrib/panoply/4.8.1        jaspy/3.11/v20240302              netcdf/intel2024.2.0/c++/4.3.1
   ceda_mip_tools/v1.5                        contrib/arsf/arsf_dem_scripts/0.2.0 (D)    esmvaltool/2.8               jaspy/3.11/v20240508              netcdf/intel2024.2.0/fortran/4.6.1
   ceda_mip_tools/v1.6                        contrib/arsf/arsf_tools/20160908           esmvaltool/2.9               jaspy/3.11/v20240815              oneapi/compilers/24.2.0
   ceda_mip_tools/v1.7                        contrib/arsf/laspy/1.3.0                   esmvaltool/2.10              jaspy/3.12/v20250704       (D)    oneapi/mpi/24.2.0
   ceda_mip_tools/v1.8                 (D)    contrib/arsf/lastools/20150925             esmvaltool/2.11LD            jasr/4.3/v20240320                snap/8.0
   contrib/arsf/apl/3.5.03                    contrib/arsf/lastools/20160730      (D)    esmvaltool/2.13              jasr/4.3/v20240815                snappy/8.0/jaspy-3.7-r20210320
   contrib/arsf/apl/3.5.06                    contrib/arsf/points2grid/1.3.1             esmvaltool/2.14       (D)    jasr/4.4/v20250704                wflogger/0.1
   contrib/arsf/apl/3.5.10             (D)    contrib/beat/v6.8                          idl/8.4                      jasr/4.4/v20250902         (D)    xconv2/0.2
   contrib/arsf/arsf_dem_scripts/0.1.2        contrib/beat/v6.9.1_patch                  idl/8.9                      jasr/4.4/v20251122
   contrib/arsf/arsf_dem_scripts/0.1.3        contrib/beat/v6.9.1                 (D)    idl/9.1               (D)    midl/20140411
   contrib/arsf/arsf_dem_scripts/0.1.5        contrib/ceda/1.0                           jasmin-sci                   netcdf/intel2024.2.0/4.9.2

  Where:
   D:  Default Module

Module defaults are chosen based on Find First Rules due to Name/Version/Version modules found in the module tree.
See https://lmod.readthedocs.io/en/latest/060_locating.html for details.

If the avail list is too long consider trying:

"module --default avail" or "ml -d av" to just list the default modules.
"module overview" or "ml ov" to display the number of modules for each name.

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

Note that piping the output through `less` allows us to search within the output using the <kbd>/</kbd> key.

### Listing Currently Loaded Modules

You can use the `module list` command to see which modules you currently have
loaded in your environment. If you have no modules loaded, you will see a
message telling you so.

```bash
[yourUsername@sci-vm-02 ~]$ module list
```

```output
No modules loaded
```

## Loading and Unloading Software

To load a software module, use `module load`.

In this example we will use "Python 3". Initially, it is not loaded.
We can test this by using the `which` command. `which` searches for
executables using directories listed in `$PATH`, similar to how Bash
locates commands.

```bash
[yourUsername@sci-vm-02 ~]$ which python3
```


If the `python3` command is available, `which` shows the path to the
executable:

```output
/usr/bin/python3
```

The shell finds executables by searching through the directories listed in
the `$PATH` environment variable.

If we accidentally make a typo for example:

```bash
[yourUsername@sci-vm-02 ~]$ which pyython3
```

we instead see something like:

```output
/usr/bin/which: no pyython3 in (/jdoe/.local/bin:/jdoe/bin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin)
```

This wall of text is actually a list of directories separated by the
`:` character. The output tells us that the shell searched the following
directories for `pyython3`, but could not find it:

```output
/jdoe/.local/bin
/jdoe/bin
/usr/local/bin
/usr/bin
/usr/local/sbin
/usr/sbin
```

The Python installation located in `/usr/bin` is the system-provided version.
On HPC systems, we often need a different Python build that is compiled with
specific compiler toolchains, libraries, or scientific software stacks.
Environment Modules allow us to dynamically switch to these alternative
software environments.

We can load a different Python environment using `module load`:


```bash
[yourUsername@sci-vm-02 ~]$ module load jaspy
[yourUsername@sci-vm-02 ~]$ which python3
```

```output
/apps/jasmin/jaspy/miniforge_envs/jaspy3.12/mf3-25.3.0-3/envs/jaspy3.12-mf3-25.3.0-3-v20250704/bin/python3
```

So, what just happened?

To understand the output, first we need to understand the nature of the `$PATH`
environment variable. `$PATH` is a special environment variable that controls
where a shell looks for executables. Specifically, `$PATH` is a list of
directories (separated by `:`) that the shell searches through for a command
before reporting that the command could not be found. As with all environment
variables, we can print it out using `echo`.

```bash
[yourUsername@sci-vm-02 ~]$ echo $PATH
```


```output
/apps/jasmin/jaspy/miniforge_envs/jaspy3.12/mf3-25.3.0-3/envs/jaspy3.12-mf3-25.3.0-3-v20250704/bin:/apps/jasmin/jaspy/miniforge_envs/jaspy3.12/mf3-25.3.0-3/condabin:/apps/jasmin/jaspy/miniforge_envs/jaspy3.11/mf3-23.11.0-0/condabin:/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/home/users/youruser/bin
```

You'll notice a similarity to the output of the `which` command. In this case,
there is one important difference: an additional directory appears at the
beginning. When we ran the `module load` command, it added a directory to the
front of our `$PATH` -- or "prepended to PATH". Because this directory appears
before `/usr/bin` in `$PATH`, the shell now finds the module-provided `python3`
executable before the system version. Let's examine what's located there:


```bash
[yourUsername@sci-vm-02 ~]$ ls /apps/jasmin/jaspy/miniforge_envs/jaspy3.12/mf3-25.3.0-3/bin/
```

```output
2to3       bsdunzip  bzip2recover  cph          gencnval       infocmp       keyctl       lz4               nghttp      pydoc3             reset             toe       wish8.6                         zstdless
2to3-3.12  bunzip2   bzless        c_rehash     gendict        infotocap     kinit        lz4c              nghttpd     pydoc3.12          sclient           tput      x86_64-conda_cos6-linux-gnu-ld  zstdmt
activate   bzcat     bzmore        curl-config  genrb          installcheck  klist        lz4cat            nghttpx     python             sim_client        tqdm      x86_64-conda-linux-gnu-ld
adig       bzcmp     captoinfo     deactivate   gss-client     jsondiff      kpasswd      makeconv          normalizer  python3            sqlite3_analyzer  tset      xml2-config
ahost      bzdiff    clear         derb         icu-config     jsonpatch     krb5-config  mamba             openssl     python3.1          tabs              unlz4     xmlcatalog
archspec   bzegrep   compile_et    distro       icuexportdata  jsonpointer   ksu          mamba-package     pip         python3.12         tclsh             unzstd    xmllint
bsdcat     bzfgrep   conda         dumpsolv     icuinfo        k5srvutil     kswitch      mergesolv         pip3        python3.12-config  tclsh8.6          uuclient  zstd
bsdcpio    bzgrep    conda2solv    genbrk       idle3          kadmin        ktutil       ncurses6-config   pkgdata     python3-config     testsolv          wheel     zstdcat
bsdtar     bzip2     conda-env     gencfu       idle3.12       kdestroy      kvno         ncursesw6-config  pydoc       repo2solv          tic               wish      zstdgrep
```

Note that the exact output may vary from cluster to cluster.

Taking this to its conclusion, `module load` adds software locations to your
`$PATH`. It effectively "loads" software into the current shell environment.
A special note on this: depending on the `module` system configuration at
your site, `module load` may also automatically load additional software
dependencies required by the application.


To demonstrate, let's use `module list`. `module list` shows all loaded
software modules.

```bash
[yourUsername@sci-vm-02 ~]$ module list
```

```output
Currently Loaded Modules:
  1) jaspy/3.12/v20250704
```

```bash
[yourUsername@sci-vm-02 ~]$ module load netcdf/intel2024.2.0/4.9.2
[yourUsername@sci-vm-02 ~]$ module list
```

```output
Currently Loaded Modules:
  1) jaspy/3.12/v20250704   2) oneapi/compilers/24.2.0   3) oneapi/mpi/24.2.0   4) netcdf/intel2024.2.0/4.9.2
```

So in this case, loading the `netcdf` module also loaded `oneapi/compilers/24.2.0` and
`oneapi/mpi/24.2.0` as well. Let's try unloading the
`netcdf` package.

```bash
[yourUsername@sci-vm-02 ~]$ module unload netcdf/intel2024.2.0/4.9.2
[yourUsername@sci-vm-02 ~]$ module list
```

```output
Currently Loaded Modules:
  1) jaspy/3.12/v20250704
```

So using `module unload` "un-loads" a module, and depending on how a site is
configured it may also unload all of the dependencies (in our case it does
not). If we wanted to unload everything at once, we could run `module purge`
(unloads everything).

```bash
[yourUsername@sci-vm-02 ~]$ module purge
[yourUsername@sci-vm-02 ~]$ module list
```

```output
No modules loaded
```

Note that `module purge` is informative. It will also let us know if a default
set of "sticky" packages cannot be unloaded (and how to actually unload these
if we truly so desired).

Note that this module loading process happens primarily through the
manipulation of environment variables like `$PATH`. There is usually little
or no data transfer involved.

The module system modifies other environment variables as well, including
variables that influence where the shell and runtime linker look for software
libraries. Examples include variables such as:

```output
LD_LIBRARY_PATH
LIBRARY_PATH
CPATH
MANPATH
PKG_CONFIG_PATH
```

On some systems, modules may also configure environment variables that tell
commercial software packages where to locate license servers.
The `module` command restores these shell environment variables to their
previous state when a module is unloaded. This allows users to switch between
different software environments cleanly and reproducibly.

## Software Versioning

So far, we've learned how to load and unload software packages. This is very
useful. However, we have not yet addressed the issue of software versioning. At
some point or other, you will run into issues where only one particular version
of some software will be suitable. Perhaps a key bugfix only happened in a
certain version, or version X broke compatibility with a file format you use.
In either of these example cases, it helps to be very specific about what
software is loaded.

Let's examine the output of `module avail` more closely, using the pager since
there may be reams of output:


```bash
[yourUsername@sci-vm-02 ~]$ module avail | less
```

```output
------------------------------------------------------------------------------------------- /apps/jasmin/modulefiles --------------------------------------------------------------------------------------------
   ants/2.1.0                                 contrib/arsf/arsf_dem_scripts/0.1.8        contrib/panoply/4.8.1        jaspy/3.11/v20240302              netcdf/intel2024.2.0/c++/4.3.1
   ceda_mip_tools/v1.5                        contrib/arsf/arsf_dem_scripts/0.2.0 (D)    esmvaltool/2.8               jaspy/3.11/v20240508              netcdf/intel2024.2.0/fortran/4.6.1
   ceda_mip_tools/v1.6                        contrib/arsf/arsf_tools/20160908           esmvaltool/2.9               jaspy/3.11/v20240815              oneapi/compilers/24.2.0
   ceda_mip_tools/v1.7                        contrib/arsf/laspy/1.3.0                   esmvaltool/2.10              jaspy/3.12/v20250704       (D)    oneapi/mpi/24.2.0
   ceda_mip_tools/v1.8                 (D)    contrib/arsf/lastools/20150925             esmvaltool/2.11LD            jasr/4.3/v20240320                snap/8.0
   contrib/arsf/apl/3.5.03                    contrib/arsf/lastools/20160730      (D)    esmvaltool/2.13              jasr/4.3/v20240815                snappy/8.0/jaspy-3.7-r20210320
   contrib/arsf/apl/3.5.06                    contrib/arsf/points2grid/1.3.1             esmvaltool/2.14       (D)    jasr/4.4/v20250704                wflogger/0.1
   contrib/arsf/apl/3.5.10             (D)    contrib/beat/v6.8                          idl/8.4                      jasr/4.4/v20250902         (D)    xconv2/0.2
   contrib/arsf/arsf_dem_scripts/0.1.2        contrib/beat/v6.9.1_patch                  idl/8.9                      jasr/4.4/v20251122
   contrib/arsf/arsf_dem_scripts/0.1.3        contrib/beat/v6.9.1                 (D)    idl/9.1               (D)    midl/20140411
   contrib/arsf/arsf_dem_scripts/0.1.5        contrib/ceda/1.0                           jasmin-sci                   netcdf/intel2024.2.0/4.9.2

  Where:
   D:  Default Module

Module defaults are chosen based on Find First Rules due to Name/Version/Version modules found in the module tree.
See https://lmod.readthedocs.io/en/latest/060_locating.html for details.

If the avail list is too long consider trying:

"module --default avail" or "ml -d av" to just list the default modules.
"module overview" or "ml ov" to display the number of modules for each name.

Use "module spider" to find all possible modules and extensions.
Use "module keyword key1 key2 ..." to search for all possible modules matching any of the "keys".
```

If the software your Slurm script runs requires on a specific version
of a dependency, make sure you use the full name of the module, rather
than the _default_ loaded when you give only its name (up to the first
slash).

:::::::::::::::::::::::::::::::::::::::  challenge

## Using Software Modules in Scripts

Create a job that is able to run `python3 --version`. Remember, no software
is loaded by default! Running a job is just like logging on to the system
(you should not assume a module loaded on the login node is loaded on a
compute node).

:::::::::::::::  solution

## Solution

```bash
[yourUsername@sci-vm-02 ~]$ nano python-module.sh
[yourUsername@sci-vm-02 ~]$ cat python-module.sh
```

```output
#!/bin/bash
#SBATCH 
#SBATCH --qos
#SBATCH --time 00:00:30

module load jaspy

python3 --version
```

```bash
[yourUsername@sci-vm-02 ~]$ sbatch --account=workshop --qos=workshop --partition=debug python-module.sh
```

:::::::::::::::::::::::::

::::::::::::::::::::::::::::::::::::::::::::::::::


:::::::::::::::::::::::::::::::::::::::: keypoints

- Load software with `module load softwareName`.
- Unload software with `module unload`
- The module system handles software versioning and package conflicts for you automatically.

::::::::::::::::::::::::::::::::::::::::::::::::::
