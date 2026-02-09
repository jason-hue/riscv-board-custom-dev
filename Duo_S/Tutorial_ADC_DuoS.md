# [DuoS-RuyiSDK] 5. ADC 读取：采集模拟信号

本教程介绍如何使用 RuyiSDK 编译 ADC 读取程序，并在 Milk-V DuoS 上采集电位器或传感器的模拟电压值。

## 1. 硬件连接

DuoS 的 SG2000 芯片集成了 SARADC（逐次逼近型模数转换器）。

### DuoS 引脚图参考
![DuoS Pinout](https://milkv.io/docs/duo/duos/duos-pinout-v1.1.webp)

### 实验接线 (以 ADC1 为例)
- **电位器一端**: 接 3.3V
- **电位器另一端**: 接 GND
- **电位器中间引脚 (信号)**: 接 DuoS J3 PIN 26 (SARADC_VIN1)

---

## 2. 源码逻辑说明

ADC 的操作在 DuoS 上主要是通过内核驱动暴露的 sysfs 接口完成的。
代码路径：`duo-examples/adc/adcRead.c`

程序会自动检测并尝试加载 `cv181x_saradc` 驱动模块，然后读取 `/sys/class/cvi-saradc/cvi-saradc0/device/cv_saradc` 文件。

---

## 3. 使用 RuyiSDK 编译

```bash
# 1. 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 2. 编译
cd duo-examples/adc
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-

# 注意：该 Makefile 检查 CFLAGS/LDFLAGS 是否为空，需显式传入
make clean
make CFLAGS="-I$(pwd)/../include/system -O3" LDFLAGS="-L$(pwd)/../libs/system/musl_riscv64"
```

---

## 4. 部署与运行

### 4.1 传输并运行
```bash
# 在开发机执行
scp adcRead root@192.168.42.1:/root/

# 在 DuoS 终端执行
chmod +x adcRead
./adcRead
```

### 4.2 选择通道
程序运行后会提示选择通道。对于接在 PIN 26 上的信号，通常对应通道 **1**。

示例输出：
```text
Define the ADC channel: 
 1: ADC1 (PIN26)  <-- 在 DuoS 上对应 VIN1
 2: ADC2 (PIN27)
 ...
1
ADC1 value is 512
ADC1 value is 1023
```

> **提示**：如果提示 "Error at opening ADC!"，请确保系统已加载 `cv181x_saradc` 模块（可手动执行 `lsmod` 查看）。
