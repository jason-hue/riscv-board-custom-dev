# 使用 RuyiSDK 编译 Milk-V DuoS 示例程序教程

本教程介绍如何脱离官方的 `envsetup.sh` 脚本，使用 **RuyiSDK** 提供的工具链来编译 `duo-examples`。这种方法更加灵活，且可以使用更新版本的 GCC 工具链（如 GCC 15）。

## 1. 准备工作

### 安装 Ruyi 包管理器
如果尚未安装 Ruyi，请执行以下命令：
```bash
wget https://mirror.iscas.ac.cn/ruyisdk/ruyi/tags/0.41.0/ruyi-0.41.0.amd64 -O /usr/local/bin/ruyi
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

### 激活环境并编译

根据您的目标系统选择对应的激活和编译方案：

#### 方案 A：针对 Debian (Glibc)
```bash
# 激活环境
source ~/venv-duos-plct/bin/ruyi-activate

# 进入示例目录
cd duo-examples/hello-world

# 设置交叉编译前缀
export TOOLCHAIN_PREFIX=riscv64-plct-linux-gnu-

# 补充项目路径（ruyi 已自动设置基础 CFLAGS）
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"

# 编译
make
```

#### 方案 B：针对 Buildroot (Musl)
```bash
# 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 进入示例目录
cd duo-examples/hello-world

# 设置交叉编译前缀
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-

# 补充项目路径
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"

# 编译
make
```

### 退出环境
```bash
ruyi-deactivate
```


---

## 4. 编译示例程序

以 `hello-world` 为例：

```bash
cd hello-world
make clean
make
```

编译完成后，会在当前目录下生成 `helloworld` 可执行文件。

---

## 5. 部署与登录开发板 (两种方法)

根据您使用的系统版本（Debian 或 Buildroot），登录和传输文件的方法有所不同。

### 方法 A：通过串口 (Serial) 登录 —— Debian 系统推荐
部分 Debian 镜像默认不开启 USB-NCM 网络，因此初始登录和操作通常通过串口进行。

1.  **硬件连接**：
    *   使用 USB 转 UART 调试器。
    *   连接 DuoS 串口：开发板 GND -> 调试器 GND, 开发板 TX -> 调试器 RX, 开发板 RX -> 调试器 TX。
2.  **启动终端连接**：
    *   在 Linux 主机上，可以使用 `minicom`、`tio` 或 `screen`。
    *   命令示例：`minicom -D /dev/ttyACM0 -c on` (设备路径根据实际情况可能是 `/dev/ttyUSB0`)。
3.  **登录系统**：
    *   默认用户名：`root`
    *   默认密码：`milkv`
4.  **传输文件**：
    由于没有网络，您可以将编译好的二进制文件拷贝到 microSD 卡的 FAT 分区，然后在开发板上挂载该分区：
    ```bash
    [root@milkv]~# mount /dev/mmcblk0p1 /mnt
    [root@milkv]~# cp /mnt/helloworld /root/
    ```

### 方法 B：通过网络 (SSH/USB-NCM) —— Buildroot 系统推荐
Buildroot 系统默认开启了 USB-NCM 虚拟网卡，电脑识别后可直接通过网络传输。

1.  **检查连接**：
    *   确保电脑识别到新网卡，并尝试：`ping 192.168.42.1`。
2.  **传输文件**：
    ```bash
    scp helloworld root@192.168.42.1:/root/
    ```
3.  **SSH 登录**：
    ```bash
    ssh root@192.168.42.1
    # 密码：milkv
    ```

---

## 6. 运行程序

在开发板终端中运行：

```bash
[root@milkv]~# chmod +x helloworld
[root@milkv]~# ./helloworld
Hello, World!
```
