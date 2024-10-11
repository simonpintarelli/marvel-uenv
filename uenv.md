---
marp: false
theme: marvel
# class: lead
paginate: true
backgroundColor: #fff
backgroundImage: url('./slides-support/common/4k-slide-bg-white.png')
size: 16:9
---

# UENV: CSCS approach to deploy user environments and scientific applications
<!-- ![bg cover](../slides-support/common/4k-slide-) -->
<!-- _paginate: skip  -->
<!-- _class: titlecover -->
<!-- _footer: "" -->

---

# Providing software via CPE
<div>
<img src="cpe.png", style="width: 50%" align="right" />
</div>
<div>
The Cray Programming Environment is complex by necessity:
- Modules provide a combinatorial set of libraries and tools that serve as many use cases as possible
- Ingration is provided by HPE: once an issue is identified HPE have to fix the issue in a future release
  - long latency between issue reporting and the fix
- Each new release requires extensive testing to check that the issues have been fixed
  - And to identify the inevatible new issues
- CSCS will no longer provide software built using CPE for users
</div>

<!-- ## GH200 SIMD -->

<!-- - 72x Arm Neoverse V2 cores -->
<!-- - 4×128-bit SIMD units per core -->

---

# A UENV is a self-contained software stack

A uenv is two components:

1. A squashfs file
   - "a read-only file system that lets you compress whole file systems or single directories, to write them to ordinary files, and then mount them using a loopback device"
   - A single file that contains everything in a working environment
2. Meta data
   - information about the uenv build (when, where, who)
   - the recipe that was used to build
   - information about the contents of the uenv
   - **environment configurations**

---


# A UENV has to available on the local file system to be used

Store in repository, which is a directory with:
- A database: **index.db**
- A hashed path for each uenv that contains
  - the squashfs image **store.squashfs**
  - meta data: **env.json**

---

# How it works

1. Unshare linux mount namespace
2. Mount the squashfs image
3. Return to unprivileged user

Implications:


---
# Material science UENVs

- CP2K
- Gromacs
- QuantumESPRESSO
- SIRIUS
- NAMD


---
# Code compilation using modules

```bash
uenv start quantumespresso/v7.3.1
uenv modules use
module load cmake \
    fftw \
    nvhpc \
    nvpl-lapack \
    nvpl-blas \
    cray-mpich \
    netlib-scalapack \
    libxc

mkdir build && cd build
FC=mpif90 CXX=mpic++ CC=mpicc cmake .. \
    -DQE_ENABLE_MPI=ON \
    -DQE_ENABLE_OPENMP=ON \
    -DQE_ENABLE_SCALAPACK:BOOL=OFF \
    -DQE_ENABLE_LIBXC=ON \
    -DQE_ENABLE_CUDA=ON \
    -DQE_ENABLE_PROFILE_NVTX=ON \
    -DQE_CLOCK_SECONDS:BOOL=OFF \
    -DQE_ENABLE_MPI_GPU_AWARE:BOOL=OFF \
    -DQE_ENABLE_OPENACC=ON
make -j20
```

---
# Code compilation using spack

```bash
uenv start quantumespresso
. $SCRATCH/spack/share/spack/setup-env.sh
```


Use an independent spack environment [https://spack.readthedocs.io/en/latest/environments.html#independent-environments](link)
```yaml
spack:
  specs:
  - quantumespresso%nvhpc +cuda build_type=Release
  develop:
    quantumespresso:
      path: /path/to/q-e/src
      spec: quantumespresso@develop
  packages:
    all:
      variants: cuda_arch=90
  view:
      default
```


---
# Useful links

- [https://confluence.cscs.ch/display/KB/UENV+user+environments](UENV user environments)
- [https://github.com/eth-cscs/alps-uenv](alps-uenv recipe repository)

