# Helix DEB Builder

自动构建 [Helix Editor](https://github.com/helix-editor/helix) Linux `.deb` 包的 GitHub Actions 工作流。

## 产物

每次构建产出两个架构的 `.deb` 包：

| 架构 | 包名示例 |
|------|---------|
| x86_64 (amd64) | `helix_25.7.1+dev.20240724.a1b2c3d4_amd64.deb` |
| ARM64 (aarch64) | `helix_25.7.1+dev.20240724.a1b2c3d4_arm64.deb` |

### 命名规则

```
helix_<版本>+dev.<构建日期>.<Git commit 短哈希>_<架构>.deb
```

- **版本**：Helix 上游的正式版本号（如 `25.7.1`）
- **构建日期**：`YYYYMMDD` 格式
- **commit 短哈希**：8 位 Git 哈希，对应上游 master 分支最新提交
- **+dev**：表示这是开发版构建，在 Debian 版本比较中高于正式版

## 触发方式

### 1. 定时触发（默认）

每月 1 号 UTC 06:00 自动拉取 Helix master 分支最新代码并构建。

### 2. 手动触发

随时到 GitHub 仓库页面 → **Actions** → **Build Helix DEB** → **Run workflow**，可选择任意 tag、分支或 commit 进行构建。

## 安装 deb 包

下载产物后，在 Debian/Ubuntu 系统上安装：

```bash
sudo dpkg -i helix_*.deb
```

安装后运行 `hx` 即可启动 Helix Editor。

### 包内容

| 路径 | 说明 |
|------|------|
| `/usr/bin/hx` | 启动脚本（设置 `HELIX_RUNTIME` 环境变量后调用实际二进制） |
| `/usr/lib/helix/hx` | Helix 主程序 |
| `/usr/lib/helix/runtime/` | 运行时文件（语法高亮、主题、tree-sitter grammars 等） |
| `/usr/share/doc/helix/` | 文档 |
| `/usr/share/bash-completion/completions/hx` | Bash 补全 |
| `/usr/share/fish/vendor_completions.d/hx.fish` | Fish 补全 |
| `/usr/share/zsh/vendor-completions/_hx` | Zsh 补全 |
| `/usr/share/applications/Helix.desktop` | 桌面入口 |
| `/usr/share/icons/hicolor/256x256/apps/helix.png` | 应用图标 |

## 构建流程

1. 从 `helix-editor/helix` 拉取源码
2. 安装 Rust 工具链（stable）
3. 抓取 tree-sitter grammar 源码
4. 使用 `--profile opt`（LTO + O3 + strip）编译二进制
5. 使用 `cargo-deb` 根据上游 `[package.metadata.deb]` 配置打包 `.deb`
6. 重命名产物，上传为 Actions Artifact

## 技术栈

- **GitHub Actions**：CI/CD 平台
- **Rust / Cargo**：编译 Helix
- **cargo-deb**：生成 Debian 包
- **dtolnay/rust-toolchain**：Rust 工具链管理
- **Swatinem/rust-cache**：增量编译缓存

## 与上游的关系

本仓库**不 fork** Helix 源码，也不维护 Helix 的修改。它仅包含一个 GitHub Actions 工作流配置文件，每次运行时从 `helix-editor/helix` 官方仓库拉取最新源码进行编译打包。
