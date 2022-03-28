
## 工具链
### 下载 riscv-gnu-toolchain

* 参考https://github.com/riscv-collab/riscv-gnu-toolchain
* clone 后排除了 qemu 这个子仓库，一来因为 qemu 完整下载太大；二来 qemu 对 toolchain 的编译本身来说其实并不需要。
```bash
cd ~
git clone https://github.com/riscv/riscv-gnu-toolchain
cd riscv-gnu-toolchain
git rm qemu
```

* clone 的主仓库并不包含子仓库的内容，所以需要继续更新子仓库。glibc 和 newlibc 托管于 sourceware.org，有时候网络可能比较差。
```bash
git submodule init
git submodule update --progress
```

* 或者是如下命令（该命令效果等同于上面两个命令）：
```bash
git submodule update --init --recursive --progress
```

* 构建工具链还需要几个标准包。在 Ubuntu 上，执行以下命令就足够了：
```bash
sudo apt-get install autoconf automake autotools-dev curl python3 libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev
```

* **注意**：如果子模块文件夹内无文件造成后续make失败，请cd到子目录然后执行以下命令：
```bash
git checkout
```
### 编译 riscv-gnu-toolchain

* 选择安装位置是/opt/riscv-gun-toolchain，编译nelib rv32imc ilp32
* rv32imc(int multiple compress)-ilp32(32-bit soft-float)
```bash
./configure --prefix=/opt/riscv-gun-toolchain  --with-arch=rv32imc --with-abi=ilp32
sudo make -j $(nproc)
```

* 添加到环境，如果需要自动生效，建议添加到~/.bashrc
```bash
echo $PATH
export PATH=/opt/riscv-gun-toolchain/bin:$PATH
```

* 更新本terminal内的PATH
```bash
source ~/.bashrc
echo $PATH
```

* 检查环境
```bash
riscv32-unknown-elf-gcc -v
```

### 下载 core-v-verif & verilator

* 参考https://github.com/openhwgroup/core-v-verif
```bash
cd ~
git clone https://github.com/openhwgroup/core-v-verif.git
```

* 暂时切换到老版本，新版本暂未适配
```bash
cd core-v-verif
git checkout cd2fe0fa6c3f5e6d0b240af65272aaeb9f6e79b6
```

* 参考https://verilator.org/guide/latest/install.html
```bash
cd ~
# Prerequisites:
#sudo apt-get install git perl python3 make autoconf g++ flex bison ccache
#sudo apt-get install libgoogle-perftools-dev numactl perl-doc
#sudo apt-get install libfl2  # Ubuntu only (ignore if gives error)
#sudo apt-get install libfl-dev  # Ubuntu only (ignore if gives error)
#sudo apt-get install zlibc zlib1g zlib1g-dev  # Ubuntu only (ignore if gives error)

git clone https://github.com/verilator/verilator   # Only first time

# Every time you need to build:
unsetenv VERILATOR_ROOT  # For csh; ignore error if on bash
unset VERILATOR_ROOT  # For bash
cd verilator
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