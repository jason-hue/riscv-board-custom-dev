# [DuoS-RuyiSDK] 4. SPI 通信：读取 MAX6675 温度

本教程介绍如何使用 RuyiSDK 编译 SPI 程序，并在 Milk-V DuoS 上通过 SPI 接口读取 MAX6675 模块的温度数据。

## 1. 硬件连接

MAX6675 是一款 K 型热电偶量温模块，使用 SPI 协议。

### DuoS 引脚图参考
![DuoS Pinout](https://milkv.io/docs/duo/duos/duos-pinout-v1.1.webp)

### 接线建议 (使用 SPI2)
根据 DuoS 的 J3 排针定义：
- **VCC**: 接 3.3V
- **GND**: 接 GND
- **SCK**: 接 DuoS J3 PIN 24 (B16) -> 需复用为 SPI2_SCK
- **SO (MISO)**: 接 DuoS J3 PIN 23 (B15) -> 需复用为 SPI2_MISO
- **CS**: 接 DuoS J3 PIN 21 (B14) -> 需复用为 GPIO 或 SPI2_CS

---

## 2. 适配 DuoS 源码

进入 `duo-examples/spi/max6675_spi` 目录。我们需要确保 `max6675_spi.c` 中的平台标识为 `"milkv_duos"`。

关键修改点：
```c
if(wiringXSetup("milkv_duos", NULL) == -1) {
    wiringXGC();
    return -1;
}
```

---

## 3. 使用 RuyiSDK 编译

```bash
# 1. 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 2. 编译
cd duo-examples/spi/max6675_spi
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export CFLAGS="${CFLAGS} -I$(pwd)/../../../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../../../libs/system/musl_riscv64"

make clean && make
```

---

## 4. 部署与配置

### 4.1 开启 SPI 引脚复用
在 DuoS 终端执行：

```bash
# 将对应引脚切换为 SPI2 功能
duo-pinmux -a SPI2_SCK
duo-pinmux -a SPI2_MISO
duo-pinmux -a SPI2_CS
```

### 4.2 传输并运行
```bash
# 在开发机执行
scp max6675_spi root@192.168.42.1:/root/

# 在 DuoS 终端执行
chmod +x max6675_spi
./max6675_spi
```

运行后，终端将每隔一秒打印当前采集到的温度（单位：℃）。
