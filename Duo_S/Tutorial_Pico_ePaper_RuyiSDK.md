# 使用 RuyiSDK 构建 Pico-ePaper-2.13 驱动程序（以 Milk-V Duo S 为例）

本文介绍如何利用 RuyiSDK 为 Milk-V Duo S 开发板快速部署交叉编译环境，并构建 Pico-ePaper-2.13 电子纸显示屏的驱动程序。

## 1. 准备工作

### 硬件环境
*   **开发板**：Milk-V Duo S (512M, SG2000)
*   **显示屏**：Pico-ePaper-2.13 (电子纸显示屏模块)
*   **其他**：microSD 卡、杜邦线

![设备型号](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/device-model.png)
![系统信息](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/device-cpuinfo.png)

### 操作系统安装与启动验证
确保您的开发板已准备好系统。

#### 下载 Duo S 的镜像
```bash
wget https://github.com/scpcom/sophgo-sg200x-debian/releases/download/v1.6.35/duos-e_sd.img.lz4
lz4 -dk duos-e_sd.img.lz4
```

#### 刷写镜像
用 dd 刷写镜像到 sd 卡：
```shell
sudo dd if=duos-e_sd.img of=/dev/sdX bs=1M status=progress
```

#### 登录系统
使用串口连接。默认用户名：`root`，默认密码：`milkv`。

![串口连接](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/uart.png)
![启动验证](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/startup.png)

## 2. 硬件接线

**注意**：连接杜邦线时不要参考显示屏的丝印，它的标注很容易让人连接错，请参考引脚图连接。

[![Pico-ePaper1-2.13](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/Pico-ePaper1-2.13.webp)](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-ePaper1-2.13.webp)

| 连接名称 | GND  | VCC        | DC    | CS    | RST   | BUSY  | CLK   | DIN   |
| -------- | ---- | ---------- | ----- | ----- | ----- | ----- | ----- | ----- |
| 引脚     | GND  | VCC (3.3V) | PIN50 | PIN11 | PIN13 | PIN46 | PIN23 | PIN19 |

[![2.13英寸 LCD Pico 扩展板引脚排列介绍](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-ePaper-2.13-details-inter.jpg)](https://raw.githubusercontent.com/jason-hue/plct/main/Pico-ePaper-2.13-details-inter.jpg)

## 3. 部署 RuyiSDK 环境

### 部署编译环境
使用 ruyi 安装工具链及示例代码包：

```bash
ruyi update
ruyi install gnu-milkv-milkv-duo-musl-bin
ruyi install milkv-duo-examples

# 创建虚拟开发环境
ruyi venv milkv-duo ./venv -t gnu-milkv-milkv-duo-musl-bin
```

## 4. 编译应用与验证

### 获取源码
```bash
# 克隆源码
git clone https://github.com/zwyzwm/Pico-ePaper-2.13.git
cd Pico-ePaper-2.13/Pico-ePaper-2.13/c
```

### 编译构建
激活虚拟环境并配置环境变量进行编译：

```bash
# 激活虚拟环境并配置环境变量
source ../../../venv/bin/ruyi-activate
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export SYSROOT=$(pwd)/../../../venv/sysroot

# 使用 ruyi 软件包提供的 wiringX 库进行编译
export CFLAGS="-mcpu=c906fdv -march=rv64imafdcv0p7xthead -mcmodel=medany -mabi=lp64d -I$(pwd)/../../../include/system"
export LDFLAGS="-L$(pwd)/../../../libs/system/musl_riscv64"

# 编译
make clean
make
```

![编译成功](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20260203170536657.png)

### 验证结果
检查生成的二进制文件：

```bash
file paper
```

![验证结果截图](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20260203170606187.png)

![运行验证结果](https://raw.githubusercontent.com/jason-hue/riscv-board-custom-dev/main/Duo_S/images/image-20250511160733556.png)
