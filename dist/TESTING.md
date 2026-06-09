# clinfo 测试使用说明

## 概述

本目录包含为 Android 和 Linux ARM 平台预编译的 clinfo 二进制文件。clinfo 是一个用于显示系统 OpenCL 平台和设备信息的命令行工具。

## 文件列表

| 文件名 | 平台 | 架构 | 说明 |
|--------|------|------|------|
| clinfo-linux-arm32 | Linux | ARM32 (armhf) | 适用于 32 位 ARM Linux 系统 |
| clinfo-linux-arm64 | Linux | ARM64 (aarch64) | 适用于 64 位 ARM Linux 系统 |
| clinfo-android-arm32 | Android | ARM32 (armeabi-v7a) | 适用于 32 位 Android 设备 |
| clinfo-android-arm64 | Android | ARM64 (arm64-v8a) | 适用于 64 位 Android 设备 |

## 依赖库说明

### Android 平台

Android 设备上的 OpenCL 实现通常由 GPU 驱动提供，依赖库文件位于以下位置：

- **32 位设备**: `/vendor/lib/egl/libGLES_mali.so`
- **64 位设备**: `/vendor/lib64/egl/libGLES_mali.so`

某些设备可能使用其他路径，如：
- `/system/vendor/lib/egl/libGLES_mali.so`
- `/system/vendor/lib64/egl/libGLES_mali.so`

### Linux 平台

Linux ARM 设备上的 OpenCL 实现通常由 Mali 驱动提供，依赖库文件可能位于以下位置：

- `/lib/libmali.so`
- `/lib64/libmali.so`
- `/lib/libMali.so`
- `/lib64/libMali.so`
- `/usr/lib/libmali.so`
- `/usr/lib64/libmali.so`
- `/usr/lib/libMali.so`
- `/usr/lib64/libMali.so`

## 使用方法

### Linux 平台使用方法

#### 方法一：直接运行（推荐）

如果系统已正确配置 OpenCL ICD，可以直接运行：

```bash
# 添加执行权限
chmod +x clinfo-linux-arm32  # 或 clinfo-linux-arm64

# 运行
./clinfo-linux-arm32         # 或 ./clinfo-linux-arm64
```

#### 方法二：设置 LD_LIBRARY_PATH

如果运行时找不到 OpenCL 库，需要设置 `LD_LIBRARY_PATH`：

```bash
# 32 位系统
export LD_LIBRARY_PATH=/lib:/usr/lib:$LD_LIBRARY_PATH
./clinfo-linux-arm32

# 64 位系统
export LD_LIBRARY_PATH=/lib64:/usr/lib64:$LD_LIBRARY_PATH
./clinfo-linux-arm64
```

#### 方法三：使用 Mali 库

如果使用 Mali GPU 驱动，可能需要指定库路径：

```bash
# 32 位
export LD_LIBRARY_PATH=/lib:/usr/lib:$LD_LIBRARY_PATH
./clinfo-linux-arm32

# 64 位
export LD_LIBRARY_PATH=/lib64:/usr/lib64:$LD_LIBRARY_PATH
./clinfo-linux-arm64
```

### Android 平台使用方法

#### 方法一：通过 ADB 运行

```bash
# 推送二进制文件到设备
adb push clinfo-android-arm64 /data/local/tmp/clinfo

# 添加执行权限
adb shell chmod +x /data/local/tmp/clinfo

# 运行（需要设置 LD_LIBRARY_PATH）
adb shell "LD_LIBRARY_PATH=/vendor/lib64:/vendor/lib64/egl:$LD_LIBRARY_PATH /data/local/tmp/clinfo"
```

#### 方法二：在设备上直接运行

如果已在 Android 设备上（如通过 Termux）：

```bash
# 复制到可执行目录
cp clinfo-android-arm64 /data/local/tmp/clinfo
chmod +x /data/local/tmp/clinfo

# 设置库路径并运行
export LD_LIBRARY_PATH=/vendor/lib64:/vendor/lib64/egl:$LD_LIBRARY_PATH
/data/local/tmp/clinfo
```

#### 方法三：32 位 Android 设备

```bash
# 推送 32 位二进制文件
adb push clinfo-android-arm32 /data/local/tmp/clinfo

# 添加执行权限
adb shell chmod +x /data/local/tmp/clinfo

# 运行
adb shell "LD_LIBRARY_PATH=/vendor/lib:/vendor/lib/egl:$LD_LIBRARY_PATH /data/local/tmp/clinfo"
```

## 常用命令选项

```bash
# 显示设备摘要列表
./clinfo -l

# 显示所有可用信息
./clinfo -a

# 尝试显示所有属性（即使平台不支持）
./clinfo --all

# 显示帮助信息
./clinfo -h
```

## 验证 OpenCL 环境

运行 clinfo 可以帮助验证 OpenCL 环境是否正确配置：

1. **成功输出**: 显示平台和设备信息，说明 OpenCL 环境正常
2. **无输出或错误**: 可能是 OpenCL 库未找到或未正确安装
3. **Segmentation fault**: 可能是 OpenCL 驱动存在问题

## 故障排除

### 问题 1: 找不到 libOpenCL.so

**解决方案**: 设置 `LD_LIBRARY_PATH` 指向包含 OpenCL 库的目录。

```bash
export LD_LIBRARY_PATH=/path/to/opencl/lib:$LD_LIBRARY_PATH
```

### 问题 2: Permission denied

**解决方案**: 确保文件有执行权限。

```bash
chmod +x clinfo-*
```

### 问题 3: No platforms found

**解决方案**:
- 检查 OpenCL ICD 配置文件是否存在 (`/etc/OpenCL/vendors/*.icd`)
- 确认 GPU 驱动已正确安装
- 验证设备支持 OpenCL

### 问题 4: Android 上无法执行

**解决方案**:
- 确保使用正确的架构版本（ARM32 或 ARM64）
- 检查 SELinux 设置
- 使用 `adb shell` 进入设备后手动运行

## 输出示例

成功运行后，clinfo 会输出类似以下的信息：

```
Number of platforms                               1
  Platform Name                                   ARM Platform
  Platform Vendor                                 ARM
  Platform Version                                OpenCL 2.0
  Platform Profile                                FULL_PROFILE
  Platform Extensions                             cl_khr_icd cl_khr_global_int32_base_atomics...

  Platform Name                                   ARM Platform
Number of devices                                 1
  Device Name                                     Mali-G52
  Device Vendor                                   ARM
  Device Version                                  OpenCL 2.0
  Driver Version                                  2.0
  Device Type                                     GPU
...
```

## 相关链接

- [clinfo GitHub 仓库](https://github.com/Oblomov/clinfo)
- [Khronos OpenCL 官网](https://www.khronos.org/opencl/)
- [ARM Mali GPU OpenCL](https://developer.arm.com/tools-and-software/graphics-and-gaming/mali-gpu)
