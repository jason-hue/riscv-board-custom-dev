# 使用 RuyiSDK 编译 Milk-V DuoS 示例程序教程

本教程介绍如何脱离官方的 `envsetup.sh` 脚本，使用 **RuyiSDK** 提供的工具链来编译 `duo-examples`。这种方法更加灵活，且可以使用更新版本的 GCC 工具链。

## 1. 准备工作

### 安装 Ruyi 包管理器
如果尚未安装 Ruyi，请执行以下命令：
```bash
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.45.0/ruyi-0.45.0.amd64 -O /usr/local/bin/ruyi
chmod +x /usr/local/bin/ruyi
```

### 获取示例源码
```bash
git clone https://github.com/milkv-duo/duo-examples.git
cd duo-examples
```

---

## 2. 安装工具链

根据你的目标系统（Debian 或 Buildroot），选择安装对应的工具链：

### 方案 A：针对 Debian 系统 (Glibc) —— 推荐使用最新的 PLCT 工具链
```bash
ruyi update
ruyi install gnu-plct
```

### 方案 B：针对官方 Buildroot 系统 (Musl) —— 使用 Milk-V 专用工具链
```bash
ruyi update
ruyi install gnu-milkv-milkv-duo-musl-bin
```

---

## 3. 使用 Ruyi 预置虚拟环境 (推荐)

Ruyi 提供了 `venv` 功能，可以自动配置编译参数（CFLAGS/LDFLAGS）和系统根目录（Sysroot）。这是使用 RuyiSDK 的**标准做法**。

### 创建虚拟环境
针对不同系统，创建对应的环境：

*   **针对 Debian (Glibc)**:
    ```bash
    ruyi venv -t gnu-plct generic ~/venv-duos-plct
    ```
*   **针对 Buildroot (Musl)**:
    ```bash
    ruyi venv -t gnu-milkv-milkv-duo-musl-bin milkv-duo ~/venv-duos-musl
    ```

### 激活环境并编译 (示例：Hello World)

根据您的目标系统选择对应的激活和编译方案：

#### 方案 A：针对 Debian (Glibc)
```bash
# 激活环境
source ~/venv-duos-plct/bin/ruyi-activate

# 编译 hello-world
cd duo-examples/hello-world
export TOOLCHAIN_PREFIX=riscv64-plct-linux-gnu-
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"
make
```

#### 方案 B：针对 Buildroot (Musl)
```bash
# 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 编译 hello-world
cd duo-examples/hello-world
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"
make
```

---

## 4. 进阶示例：控制板载 LED 闪烁 (Blink)

`blink` 示例演示了如何使用 **wiringX** 库操作 GPIO。

### 禁用系统默认闪烁脚本
DuoS 的官方固件默认上电后 LED 会自动闪烁。在运行我们编译的 `blink` 程序前，需要先将其禁用：

```bash
# 在 DuoS 终端执行
mv /mnt/system/blink.sh /mnt/system/blink.sh_backup && sync
# 重启开发板以生效
reboot
```
*如需恢复默认闪烁，将文件名改回并重启即可。*

### 适配 DuoS 硬件
由于 `duo-examples` 默认针对 Duo 编写，DuoS 的 LED 引脚和平台名称不同。编译前需修改 `blink/blink.c`：
*   **LED 引脚**：DuoS 为 `0`
*   **平台名称**：DuoS 为 `milkv_duos`

修改后的代码片段：
```c
int DUO_LED = 0;
if(wiringXSetup("milkv_duos", NULL) == -1) { ... }
```

### 编译 Blink 程序
> **⚠️ 重要兼容性提示**：`duo-examples` 提供的 `libwiringx.so` 是针对 **musl** 链接的。
> *   **方案 A (Glibc)** 编译此程序时，由于库不匹配，链接阶段会报错。
> *   **无论您的目标系统是 Debian 还是 Buildroot，建议统一使用方案 B (Musl 工具链) 编译 blink。** 编译出的二进制文件在 Debian 下通常可以直接运行。

#### 方案 B：针对 DuoS (统一推荐方案)
```bash
# 激活 Musl 环境
source ~/venv-duos-musl/bin/ruyi-activate

# 进入目录并编译
cd duo-examples/blink
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"

make clean
make
```

### 退出环境
```bash
ruyi-deactivate
```

---

## 5. 部署与登录开发板 (两种方法)

根据您使用的系统版本（Debian 或 Buildroot），登录和传输文件的方法有所不同。

### 方法 A：通过串口 (Serial) 传输 —— Debian 系统推荐
由于部分 Debian 镜像默认不开启网络，建议使用物理介质传输。

1.  **准备文件**：将编译好的程序（如 `helloworld` 或 `blink`）拷贝到 microSD 卡的 FAT 分区。
2.  **挂载并拷贝**：
    在 DuoS 串口终端执行：
    ```bash
    [root@milkv]~# mount /dev/mmcblk0p1 /mnt
    [root@milkv]~# cp /mnt/helloworld /root/  # 或 cp /mnt/blink /root/
    ```

### 方法 B：通过网络 (SSH/SCP) 传输 —— Buildroot 系统推荐
Buildroot 系统默认开启了 USB-NCM 虚拟网卡 (IP: 192.168.42.1)。

1.  **传输文件**：
    ```bash
    # 传输 hello-world
    scp hello-world/helloworld root@192.168.42.1:/root/
    # 传输 blink
    scp blink/blink root@192.168.42.1:/root/
    ```

---

## 6. 运行程序

在开发板终端中运行：

### 运行 Hello World
```bash
chmod +x helloworld
./helloworld
```

### 运行 Blink
```bash
chmod +x blink
./blink
```
