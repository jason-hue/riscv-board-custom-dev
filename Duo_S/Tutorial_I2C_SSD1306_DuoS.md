# [DuoS-RuyiSDK] 2. I2C 通信：驱动 SSD1306 OLED 屏幕

本教程介绍如何使用 RuyiSDK 编译 I2C 程序，并在 Milk-V DuoS 上驱动经典的 SSD1306 OLED 屏幕（128x32 或 128x64）。

## 1. 硬件连接

SSD1306 模块通常有 4 个引脚：VCC, GND, SCL, SDA。

### DuoS 引脚图参考
![DuoS Pinout](https://milkv.io/docs/duo/duos/duos-pinout-v1.1.webp)

### 接线建议 (使用 I2C1)
根据 DuoS 的 J3 排针定义：
- **VCC**: 接 DuoS J3 PIN 1 或 17 (3.3V)
- **GND**: 接 DuoS J3 PIN 6 或 14 (GND)
- **SCL**: 接 DuoS J3 PIN 12 (B19) -> 需复用为 I2C1_SCL
- **SDA**: 接 DuoS J3 PIN 11 (B11) -> 需复用为 I2C1_SDA

> **⚠️ 注意**：DuoS 的 GPIO 电平为 3.3V，请勿接 5V 电源，否则可能损坏芯片。

---

## 2. 适配 DuoS 源码

进入 `duo-examples/i2c/ssd1306_i2c` 目录。我们需要确保 `ssd1306_i2c.c` 中的平台标识为 `"milkv_duos"`。

关键修改点：
```c
if(wiringXSetup("milkv_duos", NULL) == -1) {
    wiringXGC();
    return -1;
}
```

---

## 3. 使用 RuyiSDK 编译

使用 **Musl** 环境（包含预置的 `libwiringx.so`）：

```bash
# 1. 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 2. 编译
cd duo-examples/i2c/ssd1306_i2c
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export CFLAGS="${CFLAGS} -I$(pwd)/../../../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../../../libs/system/musl_riscv64"

make clean && make
```

---

## 4. 部署与配置

### 4.1 开启 I2C 引脚复用
在 DuoS 的终端执行以下命令，将引脚切换到 I2C 模式（如果尚未切换）：

```bash
# 将 B19, B11 设置为 I2C1 功能 (具体编号需参考引脚复用文档)
# 示例命令 (实际编号请根据 duo-pinmux -l 查询):
duo-pinmux -a I2C1_SCL
duo-pinmux -a I2C1_SDA
```

### 4.2 传输并运行
```bash
# 在开发机执行
scp ssd1306_i2c root@192.168.42.1:/root/

# 在 DuoS 终端执行
chmod +x ssd1306_i2c
./ssd1306_i2c
```

运行后，OLED 屏幕上应显示 "Hello MilkV Duo!" 字样。
