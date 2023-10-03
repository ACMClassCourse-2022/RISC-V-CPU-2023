# Toy CPU 2022

基于 RV32I 指令集。

## 实现框架

- Memory Controller
- Instruction Cache
- Instruction Fetcher
  - With Branch Predictor
- Issue Module
- Reservation Station
- Reorder Buffer
- Load and Store Buffer
- Register File

> To Do

## 运行效率

目前版本问题：`testsleep.c` 睡的时长偏小，且 `sleep(2000)` 时实际睡眠 clk 不足 1.9 s。

上一版本问题：睡眠正常但运行时间过慢（`heart.c` 1968 s）。经严格控制变量后发现问题不在 Branch Predictor 而在于以下 Warning：

```
WARNING: [Synth 8-4767] Trying to implement RAM 'valid_reg' in registers. Block RAM or DRAM implementation is not possible; see log for reasons.
Reason is one or more of the following :
	1: Invalid write to RAM. 
	2: Unable to determine number of words or word size in RAM. 
	3: No valid read/write found for RAM. 
RAM "valid_reg" dissolved into registers
```

产生该 Warning 是因为某种不规范的初始化方式。

------

上板频率 100 MHz，WNS = -0.721 ns。

| 测试点名称     | 运行时间（s) |
| -------------- | ------------ |
| array_test1    | 0.018041     |
| array_test2    | 0.007438     |
| basicopt1      | 0.018451     |
| bulgarian      | 1.731970     |
| expr           | 0.009880     |
| gcd            | 0.006399     |
| hanoi          | 3.545234     |
| heart          | 586.205736   |
| looper         | 2.008588     |
| lvalue2        | 0.014634     |
| magic          | 0.033122     |
| manyarguments  | 0.016577     |
| multiarray     | 0.017369     |
| pi             | 1.608805     |
| qsort          | 6.536273     |
| queens         | 3.126208     |
| statement_test | 0.004048     |
| superloop      | 0.014961     |
| tak            | 0.072251     |
| testsleep      | 10.008813    |
| uartboom       | 0.784658     |

## 难调的 bug

懒得整理了，见[草稿](attach/cpu_draft.pdf)。

## WSL2 上板指引

> Microsoft 官方文档：https://learn.microsoft.com/zh-cn/windows/wsl/connect-usb

### 预备

根据官方文档先完成 [USBIPD-WIN](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#install-the-usbipd-win-project) 与 [USBIP tools and hardware database in Linux](https://learn.microsoft.com/en-us/windows/wsl/connect-usb#install-the-usbipd-win-project) 的安装。

### 将 `.bit` 文件写入板中

在使用数据线连接好 FPGA 板后根据助教的文档操作即可。

### 连接 FPGA 板至 WSL2

注：接下来需要注意命令的运行环境（以**粗体**注明）。

1. 以管理员模式打开 **PowerShell**，输入以下命令列出所有连接到 Windows 的 USB 设备。

    ```powershell
    usbipd wsl list
    ```

    ![img1](attach/fig1.png)

2. 找到 FPGA 板（设备名称应该是 `USB Serial Converter A, USB Serial Converter B`）的 bus ID，然后执行

    ```powershell
    usbipd wsl attach --busid <busid>
    ```

    ![img2](attach/fig2.png)

3. 在 **WSL** 中，可以使用以下命令确认是否连接成功。

    ```bash
    lsusb
    ```

    ![img3](attach/fig3.png)

### 修改 `run_test_fpga.sh` 的端口

与 WSL1 不同，应为 `/dev/ttyUSB1`（或者可能为其它数字，可以通过 attach 前后 `ls /dev/` 大致辨别）。

### 访问权限

通常情况下 attach 至 WSL2 的设备是不允许非 root 用户访问的。有几种方案：

1. 配置 udev 规则。~~我不会（）~~
2. 使用你喜欢的方式直接修改设备文件权限，例如

    ```bash
    sudo chmod 777 /dev/ttyUSB1
    ```
    
    不过在该种方法下，每次重新连接设备时都要执行一遍。

之后理论上便可成功执行 `./run_test_fpga.sh` 来测试你的硬件代码。

### 断开连接（或需要 reprogram 时）

当你需要对 FPGA 板重新编程时，需将板重新连回 Windows 中。在 **PowerShell** 下执行

```powershell
usbipd wsl detach --busid <busid>
```

以断开该设备与 WSL 的连接，然后你可以使用 Vivado 对其重新编程。当你准备好再次测试时，重复上述连接步骤即可。
