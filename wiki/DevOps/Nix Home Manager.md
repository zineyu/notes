---
type: page
topic: DevOps
created: 2026-06-24
updated: 2026-06-24
sources: ["raw/Articles/Nix Home Manager 最佳实践.md"]
tags:
  - wiki
  - DevOps
  - page
---
# Nix Home Manager

## 概述

Home Manager 适合把用户环境声明式纳入 Nix 配置。核心组织原则是：**Flake 负责组装，Host 描述机器，User 描述用户，Module 描述功能，原生配置文件与对应模块放在一起**。

## 核心内容

### 推荐集成方式

- NixOS 用户通常优先把 Home Manager 作为 **NixOS module** 使用，使系统配置与用户配置一次构建、一次切换。
- 非 NixOS 系统、或希望用户配置独立于系统配置更新时，再选择 standalone 模式。
- `home-manager.inputs.nixpkgs.follows = "nixpkgs"` 可让 Home Manager 与 NixOS 共用同一套 Nixpkgs，减少版本漂移。
- 稳定版配置中，Home Manager release 分支应与对应 NixOS 分支匹配。

### 目录职责分层

推荐从单用户、多机器可扩展结构开始：

```text
nix-config/
├── flake.nix
├── hosts/
├── users/
├── modules/
│   ├── os/
│   └── home/
├── secrets/
└── lib/
```

各层边界：

| 层级 | 职责 | 不应放入 |
|---|---|---|
| `flake.nix` | 声明 inputs、固定版本、组装 NixOS/Home Manager 配置、传递必要参数 | Git alias、Neovim 设置、Shell 配置、普通软件包长列表 |
| `hosts/` | 机器差异：hostname、硬件配置、文件系统、驱动、机器独有服务 | 用户主题、Shell alias、普通用户软件 |
| `users/` | 用户身份、home 目录、用户模块组合、少量机器对应用户差异 | 具体软件的大段配置 |
| `modules/os/` | 系统级功能和服务 | 用户级编辑器或终端配置 |
| `modules/home/` | 用户环境、CLI、编辑器、终端、用户级 service | 内核、驱动、系统守护进程 |

### 模块按功能拆分并自包含

Home Manager 模块应按功能拆分，而不是维护一个大杂烩文件或全局包列表。推荐把软件模块和它的原生配置文件放在一起：

```text
modules/home/programs/neovim/
├── default.nix
└── config/
    ├── init.lua
    └── lua/
```

这种结构的收益：

- 删除软件时可以删除一个目录，相关依赖自然消失。
- Lua、TOML、CSS、JSONC 等原生配置保留 LSP、格式化和语法高亮体验。
- Nix 负责软件包、插件和链接，原生语言负责复杂行为配置。

### 配置表达优先级

处理一个新软件配置时，优先级可以按以下顺序判断：

1. 是否是系统级？系统级放 NixOS，用户级放 Home Manager。
2. Home Manager 是否有原生 option？有则优先使用 `programs.*` 或 `services.*`。
3. 没有合适 option 时，XDG 配置用 `xdg.configFile`，非 XDG 文件用 `home.file`。
4. 小配置可以直接用 Nix 生成，大型 Neovim Lua、Waybar CSS、Alacritty TOML 等保留原生格式。
5. 软件专用依赖跟随软件模块，通用 CLI 放用户工具模块，项目依赖放项目 `devShell`。
6. 包含 secret 时，不得以明文进入 Flake、普通 Nix 配置或 Nix Store。
7. 只有出现多机器开关、参数化、跨程序功能、真实重复或条件启用时，才创建自定义 option。

### 关键反模式

- 过早创建 `packages/`、`overlays/`、`profiles/` 等高级目录，而实际还没有需求。
- 把 Neovim、Shell、Git 等用户配置塞进 `flake.nix`。
- 用全局 `home.packages` 维护几百行软件清单，导致依赖归属不清。
- 把几百行 Lua/CSS/TOML 写进 Nix 字符串，牺牲原生编辑体验。
- 把 `inputs` 无差别传给所有模块；应只传模块确实需要的具体 input。
- 随系统升级顺手修改 `system.stateVersion` 或 `home.stateVersion`。
- 把 secret 明文写入 Nix 源文件、`home.sessionVariables` 或 `home.file.*.text`。
- 正式配置中过度依赖 `mkOutOfStoreSymlink`，削弱可复现性与回滚一致性。
- 在每次 rebuild 前自动 `nix flake update`，使部署输入不可控。

### 最小可维护结构

单机器单用户时不必一开始采用大型架构，可从以下结构起步：

```text
nix-config/
├── flake.nix
├── flake.lock
├── hosts/desktop/
├── users/zine/default.nix
└── modules/
    ├── os/common.nix
    └── home/
        ├── common.nix
        ├── shell.nix
        ├── git.nix
        └── neovim/
```

当配置增长后，再把 `modules/home/` 演进为 `shell/`、`programs/`、`desktop/`、`services/` 等子目录。

## 相关页面

- [[运维与 SRE]] — 同属 DevOps/基础设施实践，强调可维护性、自动化和可回滚性。
- [[Kubernetes]] — NixOS 与 Home Manager 可作为个人或服务器基础设施声明式管理的补充方向。

## 演化记录

- [2026-06-24] 从 `Nix Home Manager 最佳实践` 摄入，创建 Home Manager 配置组织与反模式总结。
