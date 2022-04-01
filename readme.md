## 工具链
### 下载 riscv-gnu-toolchain

* 参考[https://github.com/riscv-collab/riscv-gnu-toolchain](https://github.com/riscv-collab/riscv-gnu-toolchain)
* `riscv-gnu-toolchain`使用tag:`2022.03.25`或者`2022.03.09`的版本
```bash
cd ~
git clone https://github.com/riscv/riscv-gnu-toolchain
cd ~/riscv-gnu-toolchain
git checkout 2022.03.25 #或者选择下面版本
# git checkout 2022.03.09
```

* qemu太大且toolchain的编译并不需要，clone后排除qemu这个子仓库
```bash
cd ~/riscv-gnu-toolchain
git rm qemu
# git reset --hard HEAD #如果你想取消rm qemu
```

* clone 的主仓库并不包含子仓库的内容，所以需要继续更新子仓库。glibc 和 newlibc 托管于 sourceware.org，有时候网络可能比较差，可以考虑把 `.gitmodules` 中的 `git://`替换为 `https://`
```bash
cd ~/riscv-gnu-toolchain
git submodule init
git submodule update --progress
```

* 或者是如下命令（该命令效果等同于上面后两个命令）：
```bash
cd ~/riscv-gnu-toolchain
git submodule update --init --recursive --progress
```

* 目前发现子模块的`riscv-binutils`2.38版本存在问题，会导致后续编译器报错找不到一些伪指令，建议使用2.37版本或者2.36.1版本，且更新`riscv-dejagnu`子模块
```bash
cd ~/riscv-gnu-toolchain/riscv-dejagnu/
git checkout
cd ~/riscv-gnu-toolchain/riscv-binutils/
git checkout riscv-binutils-2.37 #或者选择下面版本
# git checkout riscv-binutils-2.36.1
```

* 构建工具链需要几个标准包。在 Ubuntu 上，执行以下命令：
```bash
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```

### 编译 riscv-gnu-toolchain

* 选择安装位置是/opt/riscv，编译nelib版本，with-arch参数rv32imc，with-abi参数ilp32
* rv32imc(int multiple compress)-ilp32(32-bit soft-float)
```bash
cd ~/riscv-gnu-toolchain/
./configure --prefix=/opt/riscv  --with-arch=rv32imc --with-abi=ilp32
sudo make -j $(nproc)
```

* 添加到环境，如果需要自动生效，建议添加到~/.bashrc
```bash
echo $PATH
# export PATH=/opt/riscv/bin:$PATH #临时变量，重启后失效
echo -e "export PATH=/opt/riscv/bin:\$PATH" >> ~/.bashrc  #添加到~/.bashrc，不要重复添加
source ~/.bashrc
echo $PATH
```


* 如果你安装位置不是`/opt/riscv`，就需要软连接，如果使用前面`--prefix=/opt/riscv`配置就不需要
```bash
# sudo ln -s /your-installation /opt/riscv #如果你安装在了其他位置，请软连接到/opt/riscv
```
* 最后检查环境
```bash
riscv32-unknown-elf-gcc -v
```

### 下载&编译 verilator

* 参考[https://verilator.org/guide/latest/install.html](https://verilator.org/guide/latest/install.html)
```bash
cd ~
# Prerequisites:
sudo apt-get install git perl python3 make autoconf g++ flex bison ccache
sudo apt-get install libgoogle-perftools-dev numactl perl-doc
sudo apt-get install libfl2  # Ubuntu only (ignore if gives error)
sudo apt-get install libfl-dev  # Ubuntu only (ignore if gives error)
sudo apt-get install zlibc zlib1g zlib1g-dev  # Ubuntu only (ignore if gives error)

git clone https://github.com/verilator/verilator   # Only first time

# Every time you need to build:
# unsetenv VERILATOR_ROOT  # For csh; ignore error if on bash
unset VERILATOR_ROOT  # For bash
cd ~/verilator
git pull         # Make sure git repository is up-to-date
#git tag          # See what versions exist
#git checkout master      # Use development branch (e.g. recent bug fixes)
#git checkout stable      # Use most recent stable release
#git checkout v{version}  # Switch to specified release version

autoconf         # Create ./configure script
./configure      # Configure and create Makefile
sudo make -j `nproc`  # Build Verilator itself (if error, try just 'make')
sudo make install
```

* 检查 verilator 环境
```bash
verilator -V
```


### 下载 core-v-verif 
* 参考[https://github.com/openhwgroup/core-v-verif](https://github.com/openhwgroup/core-v-verif)
```bash
cd ~
git clone https://github.com/openhwgroup/core-v-verif.git
```

* 暂时切换到某一老版本，新版本暂未适配
```bash
cd ~/core-v-verif
git checkout a739efc45be10aaddb103b81f443ba93606c0569 # 2022.1.4，1.5开始修改Common.mk
# git checkout cd2fe0fa6c3f5e6d0b240af65272aaeb9f6e79b6 # 2021.12.8版本，命令不同
```

* 运行hello-world检查环境
```bash
cd ~/core-v-verif/cv32e40p/sim/core/ # only once
make clean # only once
make custom CUSTOM_PROG=coremark # only once
```


## 实现
### 验证功能

* 获取源码设计
```bash
cd ~
git clone https://github.com/LongStudy/riscv-net.git # only once
cd ~/riscv-net # every time
git pull # every time
```

* copy rtl设计
```bash
mv ~/core-v-verif/core-v-cores/cv32e40p/rtl/ ~/core-v-verif/core-v-cores/cv32e40p/rtl_bk/ # only once
cp -r ~/riscv-net/rtl/ ~/core-v-verif/core-v-cores/cv32e40p/ # every time
```

* copy net设计
```bash
cp -r ~/riscv-net/lenet/ ~/core-v-verif/cv32e40p/tests/programs/custom/lenet # every time
```

* 汇编
```bash
cd ~/core-v-verif/cv32e40p/tests/programs/custom/lenet # every time
riscv32-unknown-elf-gcc -S lenet.c # every time
mv lenet.s lenet.S # every time
rm lenet.c # every time
```

* 验证
```bash
cd ~/core-v-verif/cv32e40p/sim/core/ # every time
make clean # every time
make custom CUSTOM_PROG=lenet # every time
```


