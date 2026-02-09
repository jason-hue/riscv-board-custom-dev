# [DuoS-RuyiSDK] 6. 传感器：读取 DHT22 温湿度数据

本教程介绍如何使用 RuyiSDK 编译传感器读取程序，并在 Milk-V DuoS 上通过 GPIO 模拟单总线协议读取 DHT22 模块的温湿度。

## 1. 硬件连接

DHT22 是一款常用的温湿度传感器，使用单线（Single-Bus）通信。

### DuoS 引脚图参考
![DuoS Pinout](https://milkv.io/docs/duo/duos/duos-pinout-v1.1.webp)

### 接线建议
- **VCC**: 接 3.3V
- **GND**: 接 GND
- **DATA**: 接 DuoS J3 PIN 16 (A20) -> 对应 wiringX 编号 16

---

## 2. 适配 DuoS 源码

进入 `duo-examples/dht22` 目录。我们需要确保 `dht.c` 中的平台标识为 `"milkv_duos"`，且引脚定义正确。

关键修改点：
```c
static int DHTPIN = 16; // 对应物理引脚 16
if (wiringXSetup("milkv_duos", NULL) == -1) { ... }
```

---

## 3. 使用 RuyiSDK 编译

```bash
# 1. 激活环境
source ~/venv-duos-musl/bin/ruyi-activate

# 2. 编译
cd duo-examples/dht22
export TOOLCHAIN_PREFIX=riscv64-unknown-linux-musl-
export CFLAGS="${CFLAGS} -I$(pwd)/../include/system"
export LDFLAGS="${LDFLAGS} -L$(pwd)/../libs/system/musl_riscv64"

make clean && make
```

---

## 4. 部署与运行

### 4.1 传输并运行
```bash
# 在开发机执行
scp dht22 root@192.168.42.1:/root/

# 在 DuoS 终端执行
chmod +x dht22
./dht22
```

运行后，终端将输出当前的温度和湿度：
```text
Humidity = 45.50 % Temperature = 22.30 *C 
Humidity = 45.60 % Temperature = 22.30 *C 
```

> **注意**：由于 DHT22 是对时间极其敏感的单总线协议，Linux 系统的调度可能会导致读取失败（Data not good），程序会自动重试。
