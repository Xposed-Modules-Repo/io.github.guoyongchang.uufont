# UU Terminal NerdFont

把**网易 UU 远程**（`com.netease.uuremote`）内置终端里的 Nerd Font 图标从"方块"修复为正常图标。

- 形式：**LSPosed 模块**（APK，**字体自包含** —— 无需修改系统字体、无需 Magisk 字体模块）
- 适用：UU 远程 Android 版（4.42 实测；其他 4.x 理论上通用）
- 前提：任意 LSPosed 系框架（LSPosed / Vector / KernelSU + Zygisk Next 等），Android 9+（API 26+）
- 源码 / 反馈：<https://github.com/guoyongchang/uu-fontfix>

## 现象

UU 远程 → 设备 → **终端** → 进入会话后，所有 Nerd Font 图标（tmux 状态栏自定义按钮、oh-my-zsh / starship / p10k 主题图标等）显示为**彩色小方块** □。

根因：UU 终端用 `Typeface.Builder` / `Typeface.CustomFallbackBuilder` 加载**随包自带的字体** `res/ak/a.ttf`（Courier New Bold 派生，**不含 Nerd 字形**），既不跟随系统字体变化，也不走 `createFromAsset` —— 所以换系统字体、换输入法均无效。

## 安装

1. 在本仓库安装 APK（或从 Release 下载）
2. LSPosed / Vector 管理器 → 模块 → **启用「UU Terminal NerdFont」**
3. **作用域勾选**：`UU远程`（`com.netease.uuremote`）
4. **重启设备**（至少重启 UU 远程进程）

## 原理

模块挂钩 UU 进程内的字体加载路径，把返回值替换为**随模块分发的 JetBrainsMono Nerd Font Mono**（首次使用时复制到目标进程缓存再加载）：

| 挂钩点 | 说明 |
|---|---|
| `Typeface.Builder.build()` | 实测主路径（日志计数 780+ 次）|
| `Typeface$CustomFallbackBuilder.build()` | AndroidX TypefaceCompat 的 API 29+ 路径 |
| `Typeface.create(String, int)` | 家族名含 `courier` 时替换 |
| `Typeface.createFromAsset` / `createFromFile` / `Resources.getFont` | 兜底 |

加载优先级：模块内置字体 → `/system/fonts/DroidSansMono.ttf`（若系统已被 Nerd 字体模块覆盖）→ `monospace` 兜底。

## 验证

打开 UU 远程 → 设备 → 终端 → 任意会话，Nerd 图标应正常显示。

日志（管理器「日志」页，或 `/data/adb/lspd/log/modules_*.log`）：

```
[UUFontFix] 注入 com.netease.uuremote
[UUFontFix] Nerd 字体就绪(bundled): true
[UUFontFix] 命中 Builder.build()
```

## 停用 / 卸载

管理器里关闭模块 + 重启；或直接卸载 APK。卸载后终端恢复自带字体（图标重新变方块），无残留。

## 许可

- 模块代码：MIT（见源码仓库）
- 字体：JetBrains Mono（SIL OFL 1.1）+ Nerd Fonts 补丁（MIT），随 APK 内附许可证
- 第三方社区模块，与网易无关；「UU远程」相关权利归其各自权利人所有。
