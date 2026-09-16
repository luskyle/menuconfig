menuconfig++
============

**简体中文** | [English](README.en.md)

[![CI](https://github.com/luskyle/menuconfig-plusplus/actions/workflows/ci.yml/badge.svg)](https://github.com/luskyle/menuconfig-plusplus/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/luskyle/menuconfig-plusplus?label=release)](https://github.com/luskyle/menuconfig-plusplus/releases/latest)
![Platform](https://img.shields.io/badge/platform-linux--x86__64-informational)

从 linux kernel 源码中的 mconf 工具优化改造而来。用于图形化生成项目宏配置。

在线主页: [https://luskyle.github.io/menuconfig-plusplus/](https://luskyle.github.io/menuconfig-plusplus/)

![主菜单](image/README/1723619417014.png)

## 特性

* 内核同款 TUI: 方向键导航、`<Y>`/`<N>`/`<M>` 开关高亮项、`?` 或 `< Help >` 查看帮助、`/` 搜索
* 中文界面: 链接宽字符版 `ncursesw`, 标题、菜单、帮助窗口按显示列宽居中与截断, 中文项不错位也不被砍成半个字
* 完整 kconfig 语义: `bool` / `tristate` / `int` / `hex` / `string`、`choice`、`depends on`、`select`、多层子菜单与 `source` 文件包含
* 生成 `sdkconfig`, 可直接被 CMake 读取为编译宏
* `conf` 提供非交互模式 (`--olddefconfig` / `--defconfig` / `--allyesconfig` 等), 可放入脚本与 CI

## 依赖

* C 编译器 (gcc / clang)
* ncurses 开发库 (需要其中的**宽字符版 `ncursesw`**, 窄字符库会把中文按字节转义成乱码)

```bash
# Debian / Ubuntu
sudo apt install -y libncurses-dev
```

## 编译安装

推荐直接用 cmake, 不需要 root, 也不会污染源码目录:

```bash
cmake -S . -B build -G Ninja
cmake --build build
```

产物为 `build/mconf` (图形界面) 和 `build/conf` (命令行).

构建类型由两个开关控制:

| 参数                          | 效果                                      |
| ----------------------------- | ----------------------------------------- |
| 不传 (默认)                   | `DEBUG=ON`, 生成带调试信息的 Debug 构建 |
| `-DDEBUG=OFF`               | Release 构建 (`-O3 -DNDEBUG`)           |
| `-DCMAKE_BUILD_TYPE=<type>` | 显式指定构建类型, 以此为准                |

也可以沿用仓库里的脚本 (在源码目录内构建):

```bash
sh build.sh     # clean + cmake + ninja
sh install.sh   # 构建后把 mconf 复制到 /bin, 需要写 /bin 的权限
```

## 下载

不想自己编译的话, 从 [Releases](https://github.com/luskyle/menuconfig-plusplus/releases/latest) 下载
`menuconfig-<版本>-linux-x86_64.tar.gz`, 解包后即为 `mconf` 与 `conf` 两个可执行文件.

## 测试

`mconf test/rootconf`

将弹出如下 CLI 界面

![1723705777769](image/README/1723705777769.png)

无需图形界面时, 可以直接用 `conf` 做非交互回归:

```bash
conf --alldefconfig test/rootconf   # 全部符号取默认值, 写出 sdkconfig
conf --allyesconfig test/rootconf   # 全部选项取 y
```

`conf` 的完整模式列表见 `conf --help`:

| 选项                                   | 说明                                            |
| -------------------------------------- | ----------------------------------------------- |
| `--listnewconfig`                    | 列出新增选项                                    |
| `--oldaskconfig`                     | 以行式问答方式新建配置                          |
| `--oldconfig`                        | 以已有`.config` 为基线更新配置                |
| `--silentoldconfig <file>`           | 同`--oldconfig` 但静默, 并把头写入 `<file>` |
| `--olddefconfig` (`--oldnoconfig`) | 静默更新, 新符号取默认值                        |
| `--defconfig <file>`                 | 用`<file>` 中定义的默认值生成新配置           |
| `--savedefconfig <file>`             | 把当前配置的最小集保存到`<file>`              |
| `--allnoconfig`                      | 全部选项取`n`                                 |
| `--allyesconfig`                     | 全部选项取`y`                                 |
| `--allmodconfig`                     | 全部选项取`m`                                 |
| `--alldefconfig`                     | 全部符号取默认值                                |
| `--randconfig`                       | 随机应答所有选项                                |

## 读取宏配置

以下是写好的读取宏配置 cmake 配置

```cmake
# 读取文件内容
file(STRINGS sdkconfig lines NEWLINE_CONSUME)
# message(kconfig=${lines})


message("==========================读取配置文件==========================")

string(REGEX MATCHALL "(CONFIG[a-zA-Z_0-9]+)=([a-zA-Z0-9\"]+)" result ${lines})
message(${result})

message("==========================读取完毕,解析...==========================")

set(MATCHED_LINES "")
set(FLAGS "")
# 迭代处理每一行
foreach(line ${result})
    list(APPEND MATCHED_LINES ${line})
    # add_compile_definitions(${line})
    # string(REGEX MATCH "([a-zA-Z_0-9\"]+)" p ${line})
    string(REPLACE "\n" ";" pv ${line})
    string(REPLACE "=" ";" pv ${line})
    list(GET pv 0 p)
    list(GET pv 1 v)

    string(COMPARE EQUAL ${v} y equal)
    if(${equal})
        message(NOTICE ${p} \t\t ==> \t [选项是 y] \t ==> -D${p})
        add_compile_definitions(${p})
        set(${p} 1)
    else()
        message(NOTICE ${p} \t\t ==> \t [选项是 ${v}] \t ==> -D${line})
        add_compile_definitions(${line})
        set(${p} ${v})
    endif()
endforeach()

message("==========================配置文件解析完毕==========================")
```

## CI / CD

* `.github/workflows/ci.yml`: push 到 `main` 或提 PR 时, 以 Debug / Release / `DEBUG=OFF` 三组矩阵构建, 并断言生效的构建类型, 再用 `conf --alldefconfig` 跑非交互回归
* `.github/workflows/release.yml`: 推送 `v*` tag 时构建 Release, 打包二进制与 sha256 校验文件, 创建 GitHub Release
* `.github/workflows/pages.yml`: push 到 `main` 时把 `docs/` 发布到 GitHub Pages

## TODO

* [x] 配置文件中文展示乱码 —— 已解决 (v0.1.0): 改用宽字符版 `ncursesw`, 并按显示列宽渲染/居中/截断
* [ ] 输入框 (字符串参数、`/` 搜索、`Save as` 文件名) 支持键入与编辑中文

## 许可

* 本仓库: [LICENSE](LICENSE)
* 上游 stand-alone mconf: [LICENCE.txt](LICENCE.txt) (GPL-2.0+, Copyright © 2014 Andreas Löscher)
