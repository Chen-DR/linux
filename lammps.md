<h1 align ="center" ><font color="green">Lammps</font></h1>

## <font color = "orange">lammps编译</font>

==HPCX编译==

```shell
cd src
make yes-all #先默认安装所有的包
make no-lib #去掉lib下面所有的外部包
make mpi -j 16 #16进程编译软件
```

 ==Intel编译==

```shell
cd src
cd MAKE/OPTIONS
vi Makefile.intel_cpu_intelmpi 
```

`修改它`：

- OPTFLAGS =  -xHost改为OPTFLAGS =      -march=core-avx2
- FFT_INC =   -DFFT_MKL -DFFT_SINGLE -I/public/software/mathlib/fftw/3.3.8/double/intel/include 头文件
- FFT_PATH =  -L/public/software/mathlib/fftw/3.3.8/double/intel/lib 库地址
- FFT_LIB =   -lfftw3 -lfftw3_omp 主要是用库名

```shell
cd src
make yes-all
make no-lib
make intel_cpu_intelmpi -j 16 #16进程编译软件
```

 ==Cmake编译==

编译路径: `/work/home/mabing/software/lammps-7Aug19-intelmpi-cmake`

安装路径: `安装路径/work/home/mabing/software/lammps-7Aug19-intelmpi-cmake/lammps-7Aug19/install/bin/lmp`

```shell
${解压并创建编译和安装文件夹}
mkdir lammps_cmake
cd lammps-7Aug2019.tar.gz lammps_cmake
cd lammps_cmake
tar zxf lammps-7Aug2019.tar.gz
cd lammps-7Aug19/
mkdir build
mkdir install
${清除原本环境并更换为新环境}
module purge
module load compiler/intel/2017.5.239 mpi/intelmpi/2017.4.239 mathlib/fftw/3.3.8-intel-2017-double compiler/cmake/3.23.3
${进入编译目录并用cmake进行配置}
cd build
cmake ../cmake/ \
  -DCMAKE_CXX_COMPILER=mpiicpc \
  -DCMAKE_C_COMPILER=mpiicc \
  -DCMAKE_Fortran_COMPILER=mpiifort \
  -DCMAKE_INSTALL_PREFIX=/work/home/chenbingjie/software/lammps_cmake/lammps-7Aug19/install \
  -DCMAKE_CXX_FLAGS=-qopenmp \
  -DCMAKE_C_FLAGS=-qopenmp \
  -DCMAKE_Fortran_FLAGS=-qopenmp \
  -DBUILD_MPI=yes \
  -DBUILD_OMP=yes \
  -DFFT=FFTW3 \
  -DFFT_FFTW_THREADS=on \
  -DFFTW3_INCLUDE_DIR=/public/software/mathlib/fftw/3.3.8/double/intel/include \
  -DFFTW3_LIBRARY=/public/software/mathlib/fftw/3.3.8/double/intel/lib/libfftw3.so \
  -DFFTW3_OMP_LIBRARY=/public/software/mathlib/fftw/3.3.8/double/intel/lib/libfftw3_omp.so \
  -C ../cmake/presets/most.cmake \
  -C ../cmake/presets/nolib.cmake
${进行编译及安装 }
make -j 16
make install
```

---

==执行脚本==

```shell
#!/bin/bash
#资源调度
#SBATCH -J lammps-1
#SBATCH -p xahcnormal
#SBATCH -N 1
#SBATCH -n 32

#增加module环境
module purge 
module load compiler/devtoolset/7.3.1 mpi/hpcx/gcc-7.3.1

#将可执行程序添加为环境变量
export PATH=/work/home/chenbingjie/software/lammps-7Aug19/src:$PATH

#启动语法
srun --mpi=pmix_v3 lmp_mpi -i in.silica
#mpirun -np 32 lmp_mpi
#mpirun -np 1：实际1进程
#-pk gpu 1：实际调1卡
#mpirun -np 8
#-pk gpu 8
```

---

## <font color = "orange">lammps-dcu</font>

==执行脚本==

- `4卡脚本`

```shell
#!/bin/bash
#SBATCH -J lammps-DCU
#SBATCH -p xahdnormal
#SBATCH -N 1
#####SBATCH -n 32
#SBATCH --ntasks-per-node=32
#SBATCH --gres=dcu:4  #每个节点申请的卡数
#SBATCH --exclusive

module purge
module load compiler/devtoolset/7.3.1
module load mpi/hpcx/2.4.1-gcc-7.3.1
module load compiler/dtk/22.04.2
module load lammps/23Jun2022-DCU2-hpcx-gcc-7.3.1

mpirun -np 4 lmp_mpi -sf gpu -pk gpu 4 tpa 8 -i in.silica   
#-np表示nprocess，-sf：suffix、-pk：package gpu（这个gpu包表示
#mpirun -np 1：实际1进程
#-pk gpu 1：实际调1卡
#mpirun -np 8
#-pk gpu 8
```



- `16卡脚本`

```shell
#!/bin/bash
#SBATCH -J lammps-DCU
#SBATCH -p xahdnormal
#SBATCH -N 4
##SBATCH -n 128  #16~128范围内都可以
#SBATCH --ntasks-per-node=32
#SBATCH --gres=dcu:4  #每个节点申请的卡数
#SBATCH --exclusive

module purge
module load compiler/devtoolset/7.3.1
module load mpi/hpcx/2.4.1-gcc-7.3.1
module load compiler/dtk/22.04.2
module load lammps/23Jun2022-DCU2-hpcx-gcc-7.3.1

mpirun -np 16 lmp_mpi -sf gpu -pk gpu 16 tpa 8 -i in.silica
```

---

## <font color = "orange">lammps+voronoi</font>

==自动安装==

```shell
module purge
module load compiler/devtoolset/7.3.1  mpi/hpcx/gcc-7.3.1

tar xvf ../
cd lammps
cd src
make yes-all
make no-lib
make lib-voronoi args="-b -v voro++0.4.6"   
##这一执行之后，进入lammps/lib/voronoi看看里面已经完成了voronoi库的自动编译

make yes-VORONOI   
##这一步执行后Makefile.package文件也已经默认把voronoi库默认添加进去，这个文件决定lammps要装哪些外部包
make mpi -j 16
```

==手动安装==

```shell
module purge
module load compiler/devtoolset/7.3.1  mpi/hpcx/gcc-7.3.1

tar xvf ../
cd lammps
cd src
make yes-all
make no-lib
cd /lib/voronoi
wget https://math.lbl.gov/voro++/download/dir/voro++-0.4.6.tar.gz
tar xvf voro++-0.4.6.tar.gz
cd voro++-0.4.6
make
cd ../
ln -s voro++-0.4.6/src includelink
ln -s voro++-0.4.6/src liblink
cd ../../src
make yes-VORONOI
make mpi -j 16
```



