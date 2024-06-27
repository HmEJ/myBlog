---
title: win上超好用的包管理工具-scoop
seo_title: win上超好用的包管理工具-scoop
date: 2024-06-26 14:28:24
categories:
    - 工具
tags:
    - scoop
    - windows包管理
---

Linux 和 macOS 各有便捷的包管理工具。在 Ubuntu 上，可以通过 apt 轻松管理软件，使用 sdkman 管理开发环境。在 macOS 上，Homebrew 是一个流行的包管理工具，它同样可以轻松安装和管理各种软件包和开发环境。本文介绍的 Scoop 是一款适用于 Windows 系统的高效包管理工具，不仅能够安装众多软件，还能管理开发环境（如 Java 或 Node.js），从而避免因手动安装软件对系统目录造成污染。简直是强迫症患者福音！🤗

## 安装

在powershell中执行以下命令：

```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
# 安装scoop
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression
```

> 该命令使设备允许运行安装和管理脚本。这是必需的，因为默认情况下，Windows 10 客户端设备会限制任何 PowerShell 脚本的执行。

scoop会被安装至 `C:\Users\<YOUR USERNAME>\scoop`

安装完成后执行:

```powershell
scoop help
```

可以查看帮助信息

## 卸载

```powershell
scoop uninstall scoop
```

## 使用

### 查找软件

```powershell
scoop search <app>
```

例如： `scoop search jdk`

你的控制台会输出：

```powershell
Name                      Version                   Source Binaries
----                      -------                   ------ --------
corretto-jdk              21.0.3.9.1                java
corretto-lts-jdk          17.0.11.9.1               java
corretto11-jdk            11.0.23.9.1               java
corretto15-jdk            15.0.2.7.1                java
```

scoop默认预装了 main 的bucket。 这里我们可以看到source是java， 因此如果想要安装jdk，我们需要添加java到bucket中。

### 安装软件

```powershell
scoop install <app>
```

scoop只能安装本地bucket中存在的软件。接上，如果想要安装jdk， 我们需要先将java源添加到bucket中。

```powershell
scoop bucket add java
```

之后执行 `scoop install corretto-jdk` 即可安装。

scoop目前已知的bucket有：

- [main](https://github.com/ScoopInstaller/Main) - **Default bucket** which contains popular non-GUI apps.
- [extras](https://github.com/ScoopInstaller/Extras) - Apps that do not fit the main bucket's [criteria](https://github.com/ScoopInstaller/Scoop/wiki/Criteria-for-including-apps-in-the-main-bucket).
- [games](https://github.com/Calinou/scoop-games) - Open-source and freeware video games and game-related tools.
- [nerd-fonts](https://github.com/matthewjberger/scoop-nerd-fonts) - Nerd Fonts.
- [nirsoft](https://github.com/ScoopInstaller/Nirsoft) - A collection of over 250+ apps from [Nirsoft](https://nirsoft.net/).
- [sysinternals](https://github.com/niheaven/scoop-sysinternals) - The Sysinternals suite from [Microsoft](https://learn.microsoft.com/sysinternals/).
- [java](https://github.com/ScoopInstaller/Java) - A collection of Java development kits (JDKs) and Java runtime engines (JREs), Java's virtual machine debugging tools and Java based runtime engines.
- [nonportable](https://github.com/ScoopInstaller/Nonportable) - Non-portable apps (may trigger UAC prompts).
- [php](https://github.com/ScoopInstaller/PHP) - Installers for most versions of PHP.
- [versions](https://github.com/ScoopInstaller/Versions) - Alternative versions of apps found in other buckets.

### 卸载软件

```powershell
scoop uninstall <app>
```

### 切换软件版本

假如你同时安装了多个环境，比如同时安装jdk11和jdk17，可以通过reset命令来切换版本。

```powershell
# 切换到已安装的openjdk11
scoop reset openjdk11
```

---

本文介绍了scoop的安装，卸载，以及一些常用的指令。本章有提及到不全面的地方，请参考[scoop官网](https://scoop.sh/)或者[scoop官方wiki](https://github.com/ScoopInstaller/Scoop/wiki)。

