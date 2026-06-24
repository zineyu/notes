# Nix Home Manager 最佳实践

核心原则：

> **Flake 负责组装，Host 描述机器，User 描述用户，Module 描述功能，原生配置文件与对应模块放在一起。**

对于 NixOS 用户，通常推荐把 Home Manager 作为 **NixOS module** 使用，使系统配置和用户配置一次构建、一次切换；非 NixOS 系统或希望用户配置独立更新时，再使用 standalone 模式。Home Manager 官方同时支持 standalone、NixOS module 和 nix-darwin module。([home-manager.dev](https://home-manager.dev/manual/25.11/?utm_source=chatgpt.com "Manual (Home Manager 25.11)"))

---

## 一、推荐目录结构

适合单用户、多机器，并且可以逐渐扩展：

```text
nix-config/
├── flake.nix
├── flake.lock
│
├── hosts/
│   ├── desktop/
│   │   ├── default.nix
│   │   └── hardware-configuration.nix
│   └── laptop/
│       ├── default.nix
│       └── hardware-configuration.nix
│
├── users/
│   └── zine/
│       ├── default.nix
│       ├── desktop.nix
│       └── laptop.nix
│
├── modules/
│   ├── os/
│   │   ├── common.nix
│   │   ├── networking.nix
│   │   ├── desktop.nix
│   │   └── services/
│   │       └── ssh.nix
│   │
│   └── home/
│       ├── common.nix
│       ├── shell/
│       │   ├── default.nix
│       │   ├── zsh.nix
│       │   └── starship.nix
│       ├── programs/
│       │   ├── git.nix
│       │   ├── cli.nix
│       │   ├── neovim/
│       │   │   ├── default.nix
│       │   │   └── config/
│       │   ├── alacritty/
│       │   │   ├── default.nix
│       │   │   └── alacritty.toml
│       │   └── waybar/
│       │       ├── default.nix
│       │       └── config/
│       └── desktop/
│           ├── default.nix
│           └── wayland.nix
│
├── secrets/
│   └── secrets.yaml
│
└── lib/
    └── default.nix
```

不要一开始就创建 `packages/`、`overlays/`、`lib/`、`profiles/` 等所有目录。只有真正出现需求时再增加。

---

# 二、每层目录的职责

## `flake.nix`：只负责组装

`flake.nix` 应该负责：

- 声明 inputs
    
- 固定 Nixpkgs 与 Home Manager 来源
    
- 声明机器和用户
    
- 向模块传递参数
    
- 组装 NixOS/Home Manager 配置
    

不要在里面放：

- Git alias
    
- Neovim 设置
    
- Shell 配置
    
- 软件包长列表
    
- 桌面环境细节
    

示例：

```nix
{
  description = "Zine's NixOS configuration";

  inputs = {
    nixpkgs.url = "github:NixOS/nixpkgs/nixos-26.05";

    home-manager = {
      url = "github:nix-community/home-manager/release-26.05";
      inputs.nixpkgs.follows = "nixpkgs";
    };
  };

  outputs =
    inputs@{
      nixpkgs,
      home-manager,
      ...
    }:
    let
      system = "x86_64-linux";
      username = "zine";
    in
    {
      nixosConfigurations.desktop = nixpkgs.lib.nixosSystem {
        inherit system;

        specialArgs = {
          inherit inputs username;
        };

        modules = [
          ./hosts/desktop

          home-manager.nixosModules.home-manager

          {
            home-manager = {
              useGlobalPkgs = true;
              useUserPackages = true;
              backupFileExtension = "hm-backup";

              extraSpecialArgs = {
                inherit inputs username;
                hostname = "desktop";
              };

              users.${username} = import ./users/${username};
            };
          }
        ];
      };
    };
}
```

关键配置：

```nix
home-manager.inputs.nixpkgs.follows = "nixpkgs";
```

这样 Home Manager 和 NixOS 共用同一个 Nixpkgs，避免维护两套包版本。

稳定版配置中，Home Manager release 分支应与对应 NixOS 分支匹配。官方手册提供了 Flake 和 NixOS module 的标准配置方式。([home-manager.dev](https://home-manager.dev/manual/25.11/?utm_source=chatgpt.com "Manual (Home Manager 25.11)"))

---

# 三、Host 只存放机器差异

`hosts/desktop/default.nix`：

```nix
{
  pkgs,
  username,
  ...
}:

{
  imports = [
    ./hardware-configuration.nix

    ../../modules/nixos/common.nix
    ../../modules/nixos/networking.nix
    ../../modules/nixos/desktop.nix
  ];

  networking.hostName = "desktop";

  users.users.${username} = {
    isNormalUser = true;

    extraGroups = [
      "networkmanager"
      "wheel"
    ];
  };

  system.stateVersion = "26.05";
}
```

适合放在 Host 层的内容：

- `networking.hostName`
    
- `hardware-configuration.nix`
    
- 文件系统和磁盘
    
- 显卡驱动
    
- CPU、蓝牙、声卡配置
    
- 该机器独有的系统服务
    
- 机器启用的模块集合
    

不适合放在 Host 层：

- Git 用户配置
    
- Neovim 配置
    
- Shell alias
    
- 用户主题
    
- 普通用户软件
    

---

# 四、User 层只负责组合用户模块

`users/zine/default.nix`：

```nix
{
  hostname,
  ...
}:

{
  imports = [
    ../../modules/home/common.nix

    ../../modules/home/shell
    ../../modules/home/programs/cli.nix
    ../../modules/home/programs/git.nix
    ../../modules/home/programs/neovim

    ./${hostname}.nix
  ];

  home = {
    username = "zine";
    homeDirectory = "/home/zine";
    stateVersion = "26.05";
  };

  programs.home-manager.enable = true;
}
```

机器对应的用户差异放在：

```text
users/zine/desktop.nix
users/zine/laptop.nix
```

例如：

```nix
# users/zine/desktop.nix
{ ... }:

{
  imports = [
    ../../modules/home/desktop
  ];

  home.sessionVariables = {
    TERMINAL = "foot";
  };
}
```

如果机器数量比较多，建议不要长期依赖：

```nix
./${hostname}.nix
```

而是在 `flake.nix` 或 Host 层显式传入模块列表。显式导入更容易追踪依赖，也能避免文件不存在导致求值失败。

---

# 五、模块按功能拆分

推荐：

```text
modules/home/
├── shell/
├── programs/
├── desktop/
└── services/
```

而不是：

```text
modules/home/
├── package-list.nix
├── config1.nix
├── config2.nix
└── misc.nix
```

好的模块应该有明确职责，例如：

```text
programs/git.nix
programs/cli.nix
programs/neovim/
desktop/waybar/
shell/zsh.nix
services/syncthing.nix
```

## 一个模块应尽量自包含

例如 Neovim 模块同时管理：

- Neovim 软件包
    
- 外部依赖
    
- 配置文件
    
- 插件
    
- 环境变量
    

而不是把 Neovim 内容散落到：

```text
packages.nix
editors.nix
dotfiles.nix
aliases.nix
```

---

# 六、软件原生配置文件放在哪里

推荐采用 **模块与配置文件共置**：

```text
modules/home/programs/
├── neovim/
│   ├── default.nix
│   └── config/
│       ├── init.lua
│       └── lua/
├── alacritty/
│   ├── default.nix
│   └── alacritty.toml
├── waybar/
│   ├── default.nix
│   └── config/
│       ├── config.jsonc
│       └── style.css
└── tmux/
    ├── default.nix
    └── tmux.conf
```

好处：

- 软件模块自包含
    
- 删除软件时只删除一个目录
    
- 配置文件容易找到
    
- 不需要维护庞大的顶层 `dotfiles/`
    
- 原生语言的 LSP、格式化和语法高亮正常工作
    

---

## Neovim 推荐结构

```text
modules/home/programs/neovim/
├── default.nix
└── config/
    ├── init.lua
    └── lua/
        ├── options.lua
        ├── keymaps.lua
        ├── autocmds.lua
        └── plugins/
            ├── lsp.lua
            ├── telescope.lua
            └── treesitter.lua
```

`default.nix`：

```nix
{ pkgs, ... }:

{
  programs.neovim = {
    enable = true;
    defaultEditor = true;
    viAlias = true;
    vimAlias = true;

    extraPackages = with pkgs; [
      fd
      ripgrep

      lua-language-server
      nil
      nixfmt-rfc-style
      stylua
    ];

    plugins = with pkgs.vimPlugins; [
      nvim-lspconfig
      telescope-nvim
      nvim-treesitter
      lualine-nvim
    ];
  };

  xdg.configFile."nvim" = {
    source = ./config;
    recursive = true;
  };
}
```

最终生成：

```text
~/.config/nvim/init.lua
~/.config/nvim/lua/options.lua
```

XDG 规范将用户配置默认放在 `$HOME/.config`；Home Manager 的 `xdg.configFile` 正适合管理这类配置。([Nix Community](https://nix-community.github.io/home-manager/options.xhtml?utm_source=chatgpt.com "Appendix A. Home Manager Configuration Options"))

---

# 七、优先使用原生 Home Manager Option

推荐优先级如下。

## 1. 优先使用 `programs.*`

例如：

```nix
programs.git = {
  enable = true;

  settings = {
    user = {
      name = "zine yu";
      email = "zine.xlws@gmail.com";
    };

    init.defaultBranch = "main";
    pull.rebase = true;
    push.autoSetupRemote = true;
  };
};
```

优点：

- 类型检查
    
- 默认值处理
    
- 自动安装程序
    
- 更容易跨平台
    
- 升级时通常会提供迁移提示
    

可通过 Home Manager 官方 options 文档确认某个程序支持哪些选项。([Nix Community](https://nix-community.github.io/home-manager/options.xhtml?utm_source=chatgpt.com "Appendix A. Home Manager Configuration Options"))

## 2. 没有合适 Option 时，使用 `xdg.configFile`

```nix
xdg.configFile."myapp/config.toml".source =
  ./config.toml;
```

或者：

```nix
xdg.configFile."myapp/config.toml".text = ''
  theme = "dark"
  enable_feature = true
'';
```

## 3. 非 XDG 文件使用 `home.file`

```nix
home.file.".ideavimrc".source = ./ideavimrc;
```

## 4. 谨慎使用激活脚本

```nix
home.activation.someScript = ...
```

激活脚本属于最后手段。只要 `programs.*`、`home.file`、`xdg.configFile` 或 systemd user service 能完成，就不要写命令式脚本。

---

# 八、原生配置文件与 Nix 配置如何取舍

适合直接写成 Nix：

- 简单开关
    
- 软件包列表
    
- 环境变量
    
- 少量 alias
    
- 结构简单的配置
    
- Home Manager 已经提供完整 module 的软件
    

```nix
programs.zsh.shellAliases = {
  ll = "ls -alh";
  gs = "git status";
};
```

适合保留原生格式：

- 大型 Neovim Lua 配置
    
- Waybar CSS
    
- Hyprland 大量规则
    
- Alacritty TOML  
    -复杂 JSON/JSONC
    
- 需要独立 LSP 和格式化的配置
    

不要把几百行 Lua 塞入：

```nix
programs.neovim.extraLuaConfig = ''
  # 几百行 Lua
'';
```

更推荐：

```nix
xdg.configFile."nvim".source = ./config;
```

`extraLuaConfig` 只保留少量需要由 Nix 动态生成的内容。

---

# 九、软件包应该放在哪里

## 用户命令行软件

放 Home Manager：

```nix
home.packages = with pkgs; [
  bat
  eza
  fd
  jq
  ripgrep
];
```

## 有 Home Manager module 的软件

优先：

```nix
programs.fzf.enable = true;
programs.bat.enable = true;
programs.eza.enable = true;
```

而不是只写：

```nix
home.packages = with pkgs; [
  fzf
  bat
  eza
];
```

因为 module 通常还会配置 shell integration、生成配置文件等。

## 系统级软件和服务

放 NixOS：

```nix
services.openssh.enable = true;
networking.networkmanager.enable = true;
hardware.bluetooth.enable = true;
```

## 项目开发依赖

放项目自己的 `devShell`：

```nix
devShells.${system}.default = pkgs.mkShell {
  packages = with pkgs; [
    nodejs
    pnpm
    typescript
  ];
};
```

不要把所有语言工具链永久塞进全局 `home.packages`。

判断标准：

|内容|放置位置|
|---|---|
|用户 CLI 工具|Home Manager|
|编辑器、终端、Shell|Home Manager|
|用户级 systemd service|Home Manager|
|驱动、内核、网络|NixOS|
|系统守护进程|NixOS|
|项目构建依赖|项目 `devShell`|
|自定义软件包|`packages/`|

---

# 十、让软件包跟随对应模块

不推荐维护一个几百行的全局包列表：

```nix
home.packages = with pkgs; [
  git
  neovim
  ripgrep
  lua-language-server
  stylua
  nodejs
  ...
];
```

推荐把依赖放在使用它的模块里：

```nix
# programs/neovim/default.nix
{
  programs.neovim.extraPackages = with pkgs; [
    ripgrep
    fd
    lua-language-server
    stylua
  ];
}
```

```nix
# programs/cli.nix
{
  home.packages = with pkgs; [
    jq
    curl
    wget
  ];
}
```

这样删除 Neovim 模块时，它相关的依赖也会自然消失。

---

# 十一、插件管理的建议

以 Neovim 为例，有三种路线。

## Nix 管理插件，Lua 管理配置

推荐作为默认方案：

```nix
programs.neovim.plugins = with pkgs.vimPlugins; [
  nvim-lspconfig
  telescope-nvim
  nvim-treesitter
];
```

插件配置仍使用原生 Lua。

优点：

- 插件版本被 `flake.lock` 和 Nixpkgs 固定
    
- 不需要启动时联网
    
- 可复现性强
    
- Lua 配置仍然自然
    

## `lazy.nvim` 管理插件

适合已经有成熟 Neovim 配置，或者希望完全按照插件 README 配置。

代价是：

- 插件不完全由 Nix 管理
    
- 首次启动可能需要联网
    
- 插件锁文件需要单独维护
    

## NixVim

NixVim 使用 Nix module 表达 Neovim 配置，并仍允许插入 Lua。([Nix Community](https://nix-community.github.io/nixvim/?utm_source=chatgpt.com "Home - nixvim docs"))

适合：

- 希望 Neovim 也完全声明式
    
- 需要模块开关和跨机器覆盖
    
- 熟悉 Nix module system
    

不一定适合：

- 已有大型 Lua 配置
    
- 经常直接参考插件 Lua 文档
    
- 希望配置可在非 Nix 环境复用
    

一般个人配置中，**Nix 管插件和二进制，Lua 管编辑器行为**，平衡最好。

---

# 十二、不要过早构建自定义 Option

一开始没必要全部写成：

```nix
my.programs.neovim.enable = true;
my.shell.enable = true;
my.desktop.enable = true;
```

简单导入就足够：

```nix
imports = [
  ./programs/neovim
  ./shell
  ./desktop
];
```

只有出现下列情况时才创建自定义 option：

- 多台机器需要开关
    
- 模块需要参数
    
- 一个功能横跨多个程序
    
- 多处存在真实重复
    
- 需要通过 `lib.mkIf` 条件启用
    

示例：

```nix
{
  config,
  lib,
  pkgs,
  ...
}:

let
  cfg = config.my.development;
in
{
  options.my.development = {
    enable = lib.mkEnableOption "development environment";

    languages = lib.mkOption {
      type = lib.types.listOf lib.types.str;
      default = [ ];
    };
  };

  config = lib.mkIf cfg.enable {
    programs.git.enable = true;
    programs.direnv.enable = true;

    home.packages = with pkgs; [
      jq
      gnumake
    ];
  };
}
```

抽象应该用于减少真实重复，而不是为了让目录看起来“高级”。

---

# 十三、谨慎传递 `inputs`

常见写法：

```nix
specialArgs = {
  inherit inputs username;
};
```

和：

```nix
home-manager.extraSpecialArgs = {
  inherit inputs username hostname;
};
```

这种写法方便，但不要让所有模块都随意直接引用 `inputs`。

优先使用：

```nix
{ pkgs, lib, config, ... }:
```

只有模块确实需要某个额外 Flake input 时才传递它。更严格的写法是只传具体 input：

```nix
extraSpecialArgs = {
  inherit username hostname;
  inherit (inputs) nixvim;
};
```

这样依赖关系更清楚。

---

# 十四、`stateVersion` 不要随升级修改

```nix
system.stateVersion = "26.05";
home.stateVersion = "26.05";
```

它们表示配置采用哪一版默认行为和数据兼容规则，不是当前安装的软件版本。

升级系统时：

```bash
nix flake update
```

通常不应该顺手修改：

```nix
home.stateVersion
system.stateVersion
```

只有阅读 release notes，并确认完成对应迁移后才考虑调整。Home Manager 官方维护独立 release notes，应在升级前检查不兼容变化。([Nix Community](https://nix-community.github.io/home-manager/release-notes.xhtml?utm_source=chatgpt.com "Appendix D. Release Notes"))

---

# 十五、密钥绝不能明文进入 Nix Store

不要这样写：

```nix
home.sessionVariables.API_TOKEN = "真实密钥";
```

也不要这样：

```nix
home.file.".ssh/id_ed25519".text = ''
  私钥内容
'';
```

Flake 源文件和引用文件可能被复制到 `/nix/store`。官方文档明确警告，不应把未加密 secret 放入 Flake；Nix Store 中的相关文件可能对本机普通用户可读。([nixos.org](https://nixos.org/manual/nixos/stable/options?utm_source=chatgpt.com "Appendix A. Configuration Options"))

推荐：

- `sops-nix`
    
- `agenix`
    
- 系统凭据机制
    
- 密码管理器
    
- 运行时读取受权限保护的文件
    

仓库中只能保存：

- 加密后的 secret
    
- secret 的路径或声明
    
- 公钥
    
- 非敏感模板
    

即使使用 `git-crypt`，如果解密后的 secret 最终进入 derivation 或 Nix Store，仍然不安全。

---

# 十六、谨慎使用 `mkOutOfStoreSymlink`

普通声明：

```nix
xdg.configFile."nvim".source = ./config;
```

会把配置加入 Nix Store，然后在目标位置创建链接。这具有：

- 可复现
    
- 可回滚
    
- 与 generation 一致
    
- 内容不可直接修改
    

`mkOutOfStoreSymlink` 会链接到仓库原始路径：

```nix
xdg.configFile."nvim".source =
  config.lib.file.mkOutOfStoreSymlink
    "${config.home.homeDirectory}/nix-config/modules/home/programs/neovim/config";
```

适合：

- 正在频繁调试配置
    
- 软件必须修改自己的配置
    
- 确实需要保存后立即生效
    

缺点：

- 降低可复现性
    
- 回滚不再保证配置内容回滚
    
- 路径与机器绑定
    
- 仓库移动后链接失效
    

正式配置优先使用 Store 内文件；开发调试期间才考虑 out-of-store symlink。

---

# 十七、使用 XDG 路径

XDG 规范中：

```text
配置：~/.config
数据：~/.local/share
状态：~/.local/state
缓存：~/.cache
```

([Freedesktop.org 规范](https://specifications.freedesktop.org/basedir/?utm_source=chatgpt.com "XDG Base Directory Specification"))

Home Manager 中对应：

```nix
xdg.configFile."app/config.toml".source = ./config.toml;
xdg.dataFile."app/data.json".source = ./data.json;
```

不要把本应在 `.config` 中的文件随意放到 `$HOME` 根目录。

但软件自身不支持 XDG 时，应遵循软件要求，使用：

```nix
home.file.".some-config".source = ./some-config;
```

---

# 十八、避免使用 `with pkgs;` 的范围过大

模块内部的小列表可以：

```nix
home.packages = with pkgs; [
  jq
  fd
  ripgrep
];
```

不要在整个文件顶部写：

```nix
with pkgs;
{
  ...
}
```

较大的列表可以显式写：

```nix
home.packages = [
  pkgs.jq
  pkgs.fd
  pkgs.ripgrep
];
```

显式写法更容易识别变量来自 `pkgs`、`lib` 还是局部定义。

---

# 十九、统一格式化和检查

推荐在 Flake 中定义 formatter：

```nix
{
  outputs = { nixpkgs, ... }: {
    formatter.x86_64-linux =
      nixpkgs.legacyPackages.x86_64-linux.nixfmt-rfc-style;
  };
}
```

日常修改流程：

```bash
nix fmt
nix flake check
sudo nixos-rebuild build --flake .#desktop
sudo nixos-rebuild switch --flake .#desktop
```

如果 Home Manager standalone：

```bash
nix fmt
nix flake check
home-manager build --flake .#zine@desktop
home-manager switch --flake .#zine@desktop
```

建议先 `build`，确认没有求值或构建错误，再 `switch`。

---

# 二十、固定依赖，但不要盲目更新

`flake.lock` 应提交 Git：

```bash
git add flake.lock
git commit
```

更新全部依赖：

```bash
nix flake update
```

只更新一个 input：

```bash
nix flake update home-manager
```

更稳妥的升级流程：

```bash
nix flake update
nix flake check
sudo nixos-rebuild build --flake .#desktop
```

确认无误后再切换。

不要在每次 rebuild 前自动执行 `nix flake update`，否则每次部署的输入都可能变化，失去锁文件带来的可控性。

---

# 二十一、Git Flake 的常见陷阱

Flake 位于 Git 仓库中时，新建文件后可能出现：

```text
error: path ... does not exist
```

即使文件实际上存在，也可能因为它还没有进入 Git working tree。

执行：

```bash
git add modules/home/programs/neovim/default.nix
```

然后重新构建。

官方 Flakes 文档也指出，Git Flake 只会复制工作树中的文件，因此新文件需要先加入 Git。([wiki.nixos.org](https://wiki.nixos.org/wiki/Flakes?utm_source=chatgpt.com "Flakes - Official NixOS Wiki"))

---

# 二十二、推荐的最小可维护结构

如果目前只有一台机器和一个用户，不需要一开始就采用大型结构：

```text
nix-config/
├── flake.nix
├── flake.lock
├── hosts/
│   └── desktop/
│       ├── default.nix
│       └── hardware-configuration.nix
├── users/
│   └── zine/
│       └── default.nix
└── modules/
    ├── os/
    │   └── common.nix
    └── home/
        ├── common.nix
        ├── shell.nix
        ├── git.nix
        └── neovim/
            ├── default.nix
            └── config/
```

等配置增长后再变成：

```text
modules/home/
├── shell/
├── programs/
├── desktop/
└── services/
```

不要为了预想中的十台机器而提前设计复杂框架。

---

# 二十三、一套完整的判断规则

遇到新配置时，可以按这个顺序判断：

1. **这是系统级还是用户级？**
    
    - 系统级放 NixOS
        
    - 用户级放 Home Manager
        
2. **Home Manager 是否有原生 option？**
    
    - 有：使用 `programs.*` 或 `services.*`
        
    - 没有：继续判断
        
3. **软件是否使用 XDG？**
    
    - 是：使用 `xdg.configFile`
        
    - 否：使用 `home.file`
        
4. **配置很小还是很大？**
    
    - 很小：可以直接用 Nix 生成
        
    - 很大：保留 TOML、Lua、CSS、JSON 等原生格式
        
5. **依赖属于谁？**
    
    - 某个软件专用：放对应模块
        
    - 通用用户工具：放 `cli.nix`
        
    - 项目专用：放项目 `devShell`
        
6. **包含 secret 吗？**
    
    - 包含：不得进入普通 Nix 配置和 Store
        
7. **是否真的需要自定义 option？**
    
    - 没有复用、参数化或条件启用需求：直接 import
        

---

# 最终推荐规范

你的配置可以遵循这十条：

1. `flake.nix` 只负责输入和组装。
    
2. `hosts/` 只描述机器差异。
    
3. `users/` 只描述用户与模块组合。
    
4. `modules/nixos/` 管系统功能。
    
5. `modules/home/` 管用户环境。
    
6. 每个软件的 Nix 模块与原生配置文件放在同一目录。
    
7. 优先使用 Home Manager 原生 options。
    
8. 软件专用依赖跟随软件模块，项目依赖放 `devShell`。
    
9. `stateVersion` 不随普通升级修改。
    
10. Secret 永远不以明文进入 Flake 或 Nix Store。
    

其中最重要的目录模式是：

```text
modules/home/programs/<软件名>/
├── default.nix
└── config/
```

这套结构在保持声明式和可复现性的同时，也保留了 Neovim Lua、Waybar CSS、Alacritty TOML 等原生配置的编辑体验。