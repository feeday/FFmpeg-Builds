- https://github.com/BtbN/FFmpeg-Builds/releases/tag/latest
- https://hf-mirror.com/datasets/feeday/datxy/blob/main/win/ffmpeg-n8.1-latest-win64-gpl-8.1.zip
- https://youtube.iiilab.com/
```
import subprocess
import os
import glob

# =========================
# FFmpeg 路径
# =========================
FFMPEG_PATH = r"D:\ffmpeg-n8.1-latest-win64-gpl-8.1\bin\ffmpeg.exe"


def fast_merge(folder_path):
    # 查找目录下的 .webm 和 .weba 文件
    video_list = glob.glob(os.path.join(folder_path, "*.webm"))
    audio_list = glob.glob(os.path.join(folder_path, "*.weba"))

    if not video_list:
        print("错误：指定路径下未找到 .webm 文件。")
        return

    if not audio_list:
        print("错误：指定路径下未找到 .weba 文件。")
        return

    # 选取第一个匹配文件
    v_path = video_list[0]
    a_path = audio_list[0]

    out_path = os.path.join(folder_path, "merged_output.webm")

    command = [
        FFMPEG_PATH,
        "-y",
        "-i", v_path,
        "-i", a_path,
        "-c", "copy",
        out_path
    ]

    print("视频：", v_path)
    print("音频：", a_path)
    print("正在无损快速合并...")

    try:
        subprocess.run(command, check=True)
        print(f"\n合并完成：{out_path}")
    except subprocess.CalledProcessError as e:
        print("FFmpeg 合并失败：", e)
    except FileNotFoundError:
        print("找不到 FFmpeg，请检查：")
        print(FFMPEG_PATH)


# 修改这里
TARGET_DIR = r"C:\test"

fast_merge(TARGET_DIR)
```




# FFmpeg Static Auto-Builds

Static Windows (x86_64) and Linux (x86_64) Builds of ffmpeg master and latest release branch.

Windows builds are targetting Windows 7 and newer, provided UCRT is installed.
The minimum supported version is Windows 10 22H2, no guarantees on anything older.

Linux builds are targetting RHEL/CentOS 8 (glibc-2.28 + linux-4.18) and anything more recent.

## Auto-Builds

Builds run daily at 12:00 UTC (or GitHubs idea of that time) and are automatically released on success.

**Auto-Builds run ONLY for win(arm)64 and linux(arm)64. There are no win32/x86 auto-builds, though you can produce win32 builds yourself following the instructions below.**

### Release Retention Policy

- The last build of each month is kept for two years.
- The last 14 daily builds are kept.
- The special "latest" build floats and provides consistent URLs always pointing to the latest build.

## Package List

For a list of included dependencies check the scripts.d directory.
Every file corresponds to its respective package.

## How to make a build

### Prerequisites

* bash
* docker

### Build Image

* `./makeimage.sh target variant [addin [addin] [addin] ...]`

### Build FFmpeg

* `./build.sh target variant [addin [addin] [addin] ...]`

On success, the resulting zip file will be in the `artifacts` subdir.

### Targets, Variants and Addins

Available targets:
* `win64` (x86_64 Windows)
* `win32` (x86 Windows)
* `linux64` (x86_64 Linux, glibc>=2.28, linux>=4.18)
* `linuxarm64` (arm64 (aarch64) Linux, glibc>=2.28, linux>=4.18)

The linuxarm64 target will not build some dependencies due to lack of arm64 (aarch64) architecture support or cross-compiling restrictions.

* `davs2` and `xavs2`: aarch64 support is broken.
* `libmfx` and `libva`: Library for Intel QSV, so there is no aarch64 support.

Available variants:
* `gpl` Includes all dependencies, even those that require full GPL instead of just LGPL.
* `lgpl` Lacking libraries that are GPL-only. Most prominently libx264 and libx265.
* `nonfree` Includes fdk-aac in addition to all the dependencies of the gpl variant.
* `gpl-shared` Same as gpl, but comes with the libav* family of shared libs instead of pure static executables.
* `lgpl-shared` Same again, but with the lgpl set of dependencies.
* `nonfree-shared` Same again, but with the nonfree set of dependencies.

All of those can be optionally combined with any combination of addins:
* `4.4`/`5.0`/`5.1`/`6.0`/`6.1`/`7.0`/`7.1` to build from the respective release branch instead of master.
* `debug` to not strip debug symbols from the binaries. This increases the output size by about 250MB.
* `lto` build all dependencies and ffmpeg with -flto=auto (HIGHLY EXPERIMENTAL, broken for Windows, sometimes works for Linux)

