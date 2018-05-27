## AndroidSerialPort

### 说明

AndroidSerialPort 用于串口通信，由 Google 官方串口通信库迁移而来，并在此基础上做了扩展。

AndroidSerialPort 支持设置 su 路径、串口路径、波特率、校验位、数据位、停止位。
<br>

### 使用

1. 在 modul 下的 build.gradle 中添加

   ```groovy
   implementation "com.deemons.serialport:serialport:1.1.0"
   ```

2. 打开串口

   ```java
   SerialPort mSerialPort = new SerialPort("/dev/ttyS1", 9600);
   //获取串口文件的输入输出流，以便数据的收发
   InputStream inputStream = serialPort.getInputStream();
   OutputStream outputStream = serialPort.getOutputStream();
   ```

<br>

### API

#### 设置 su 路径

Android 主板在与其它硬件进行串口通信时，串口作为底层实现，Android 系统把设备作为一个文件，与其他设备进行串口通信就相当于读写此文件。

因此需要 root 权限来操作串口文件，默认的 su 文件路径在 `/system/bin/su` ，你可以重新设置 su 的文件路径，以便获取 root 权限，例如，设置路径`/system/xbin/su`

```java
//需要在打开串口前调用
SerialPort.setSuPath("/system/xbin/su");
```

<br>

#### 查看串口设备列表

Android 串口文件都在 ``/proc/tty/drivers`` 目录下，因此可以获取所有串口文件。

```java
SerialPortFinder serialPortFinder = new SerialPortFinder();
String[] allDevices = serialPortFinder.getAllDevices();
String[] allDevicesPath = serialPortFinder.getAllDevicesPath();
```

<br>

#### 打开串口

如果你需要设置更多参数，请使用以下构造函数

```java
/**
 * 打开串口
 * @param device 串口设备文件
 * @param baudRate 波特率
 * @param parity 奇偶校验，0 None（默认）； 1 Odd； 2 Even
 * @param dataBits 数据位，5 ~ 8  （默认8）
 * @param stopBit 停止位，1 或 2  （默认 1）
 * @param flags 标记 0（默认）
 */
public SerialPort(File device, int baudRate, int parity, int dataBits, int stopBit, int flags)
```

检验位一般默认是0（NONE），数据位一般默认为8，停止位默认为1。

<br>

#### 读写串口

##### 读数据

```java
//开子线程，持续接收串口数据
new Thread(() -> {
    //获取输入流
    InputStream is = mSerialPort.getInputStream();
    byte[] buffer = new byte[64];
    while (!isInterrupted) {
        if (mSerialPort == null || is == null) {
            close();
            return;
        }
        is.read(buffer);
    }
}).start();
```
不过，现在一般都用 Rxjava2 ，处理异常更方便

```java
Flowable.create((FlowableOnSubscribe<byte[]>) emitter -> {
            InputStream is = mSerialPort.getInputStream();
            byte[] buffer = new byte[1];
            while (!isInterrupted) {
                if (mSerialPort == null || is == null) {
                    close();
                    return;
                }
                is.read(buffer);
                emitter.onNext(buffer);
            }
        }, BackpressureStrategy.LATEST)
```

##### 写数据

```java
//获取输出流
OutputStream outputStream = mSerialPort.getOutputStream();
outputStream.write(ByteUtils.hexStringToBytes("CCAA0300"));
```

<br>

#### 关闭串口

```java
 mSerialPort.close();
```
