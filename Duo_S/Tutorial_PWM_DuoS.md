# [DuoS-RuyiSDK] 3. PWM 控制：呼吸灯效果

本教程介绍如何使用 RuyiSDK 编译 PWM 程序，通过 Milk-V DuoS 的 PWM 引脚控制 LED 的亮度，实现呼吸灯效果。

## 1. 硬件连接

DuoS 有多个引脚支持 PWM 功能。

### DuoS 引脚图参考
![DuoS Pinout](https://milkv.io/docs/duo/duos/duos-pinout-v1.1.webp)

### 实验接线
- **LED 正极**: 接 DuoS J3 PIN 13 (B12，对应 PWM4)
- **LED 负极**: 接串联电阻（220Ω）后接 GND (J3 PIN 14)

---

## 2. 适配 DuoS 源码

进入 `duo-examples/pwm` 目录。我们需要确保 `pwm.c` 中的平台标识为 `"milkv_duos"`。

关键修改点：
```c
if(wiringXSetup("milkv_duos", NULL) == -1) {
    wiringXGC();
    return 1;
}
```

---

## 3. 使用 RuyiSDK 编译

```bash
# 1. 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 2. 编译
cd duo-examples/pwm
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"

make clean && make
```

---

## 4. 部署与运行

### 4.1 开启 PWM 引脚复用
在使用 PWM 之前，必须通过 `duo-pinmux` 开启引脚的功能。

```bash
# 在 DuoS 终端执行，将 B12 切换为 PWM4
duo-pinmux -a PWM4
```

### 4.2 传输并运行
```bash
# 在开发机执行
scp pwm root@192.168.42.1:/root/

# 在 DuoS 终端执行
chmod +x pwm
./pwm
```

程序运行后，按提示输入 `[引脚号]:[占空比]`。
例如输入：`13:500` (假设物理引脚 13 映射为 wiringX 编号 13，具体取决于驱动实现)。

> **提示**：wiringX 的 PWM 映射通常遵循物理引脚编号。如果输入 13 没反应，请参考 `duo-pinmux -l` 确认 PWM 状态。
