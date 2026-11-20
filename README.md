# MRtc — MRcode 编译工具链分片仓库

本仓库为 [MRcode](https://github.com/MushBot/MRcode) 桌面版提供按需下载的编译工具链分片，
不包含任何源码，只存放 `manifest.json` 与分片 ZIP。

## 目录

```
manifest.json        分片清单（版本、sha256、大小、下载地址、落盘位置）
shards/*.zip         分片包，解压后按 manifest 的 dest 落到 ct/ 或安装根目录
```

## 客户端如何使用

MRcode 启动时按需拉取 `manifest.json`，比对本地已装分片的 sha256，只补齐缺失或变更的分片，
解压到 `<安装目录>/ct`（Python 运行时落到安装根目录），因此同一版本重复安装不会重复下载。

- 清单地址：<https://raw.githubusercontent.com/MushBot/MRtc/main/manifest.json>
- 分片地址：`https://raw.githubusercontent.com/MushBot/MRtc/main/shards/<name>.zip`

## 分片设计

分片按「变化频率」而不是「功能」切分，升级 core 或库时不需要重下 400MB 的编译器：

| 类别 | 分片 | 说明 |
| --- | --- | --- |
| 主程序 | `cli` | arduino-cli 可执行文件与配置 |
| ESP32 编译器 | `tc-gcc-riscv*` | RISC-V gcc 8.4，C3 编译必需 `tc-gcc-riscv` + `tc-gcc-riscv-mc`，其余为可选 ABI 变体 |
| ESP32 编译器 | `tc-gcc-xts3` | Xtensa gcc 8.4，S3 编译必需 |
| ESP32 工具 | `tc-esp32-tools` | esptool / mkspiffs 等 |
| ESP32 core | `core-c3` / `core-s3` / `core-common` | 核心与共用部分分离 |
| AVR | `tc-avr` | Arduino Nano C/C++，可选（已剔除 Nano 不用的 arm-none-eabi-gcc） |
| 库 | `libs` | 默认库集合 |
| Python | `py-core` / `py-scripts` | MicroPython 运行时与脚本 |
| Python 扩展 | `py-numpy*` / `py-vision` / `py-ml` / `py-media` | 可选，numpy 因 MKL 体积拆为 3 片 |

单片体积全部低于 GitHub 单文件 100MB 限制（当前最大 83.8MB）。

## 重新生成

在 MRcode 仓库内执行：

```bash
npm run pack:toolchain            # 全量打包到 toolchain-out/
node scripts/pack-toolchain.js --list          # 查看分片定义
node scripts/pack-toolchain.js --only cli,core-c3 --out out   # 只打部分分片
```

打包后把 `toolchain-out/manifest.json` 与 `toolchain-out/shards/` 覆盖到本仓库即可。
sha256 由脚本在打包时写入 manifest，未变化的分片内容一致，可跨版本复用。
