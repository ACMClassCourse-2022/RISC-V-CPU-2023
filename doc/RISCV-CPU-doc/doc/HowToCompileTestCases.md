# 如何编译测试点
## Step1. 安装riscv-gnu-toolchain
> 为保证你实际使用的测试点与助教用于测试的测试点一致，你应该按照此教程配置正确版本的riscv-gnu-toolchain

在当前的仓库中， `riscv-gnu-toolchain` 已经以 submodule 的形式引入，你需要首先将其下载至本地。

在仓库根目录执行以下指令：（仓库较大，可能需要花费较长的时间下载）
``` bash
git submodule update --init --recursive
```

接下来， 你需要编译并安装 `riscv-gnu-toolchain`，具体方法可见[此README](https://github.com/riscv-collab/riscv-gnu-toolchain/blob/b86b2b37d0acc607156ff56ff17ee105a9b48897/README.md)，这里我们只做基本的介绍

在上一步之后，执行以下指令

``` bash
cd riscv-gnu-toolchain
./configure --prefix=/opt/riscv --with-arch=rv32i --with-abi=ilp32
sudo make
```

其中，第二条指令中的 `/opt/riscv` 为 `riscv-gnu-toolchain` 的安装路径，可自行修改，但同时需要修改仓库中 `riscv/Makefile` 中的 `riscv_toolchain` 和 `riscv/script/build_test.sh` 中的 `prefix` 与其一致。

安装完成后，可执行

``` bash
/opt/riscv/bin/riscv32-unknown-elf-gcc --version
```
确认安装成功

## Step2. 编译测试点
> 这里推荐使用编写好的 Makefile 进行编译

在 `riscv` 目录下执行
``` bash
make build_sim_test name=000_array_test1
```

其中 `000_array_test1` 可以替换为你需要编译的测试点名称。
编译的结果为 `riscv/testspace` 中的 `test.data` `test.bin` 文件，另外 `test.dump` 文件展示了具体的汇编指令，可以在调试时作为参考。

## Tips
1. 在simulation阶段，编译测试点前需要确认 `riscv/sys/io.h` 中 `#define SIM` 生效，否则会导致某些测试点无法正确执行。
2. To be continue