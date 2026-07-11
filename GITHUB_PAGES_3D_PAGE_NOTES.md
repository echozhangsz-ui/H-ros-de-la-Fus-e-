# GitHub Pages 3D 页面防空白说明

## 问题原因

`character_3d_list.html` 之前把所有角色卡片都放在 `type="module"` 脚本里生成。

如果 `three.module.js` 加载失败、模块脚本报错、WebGL 初始化失败，浏览器会停止执行这段模块脚本，结果 `#grid` 里没有任何卡片，页面只剩标题和背景，看起来就是空白。

## 必须遵守

1. 不要让 3D 模块脚本负责生成基础内容。
2. 页面必须先用普通 HTML 或普通 `<script>` 渲染出可见卡片。
3. Three.js 只能做增强：成功时覆盖成 WebGL 3D，失败时保留 CSS fallback。
4. `#grid` 不能依赖 `import * as THREE from "./three.module.js"` 才出现内容。
5. 发布前必须检查 GitHub Pages 公开链接，不只检查本地文件。
6. 公开链接检查时要加版本参数，例如 `?v=commitHash`，避免浏览器缓存旧文件。

## 当前页面结构

`character_3d_list.html` 现在分两层：

1. 普通脚本：
   - 定义 `window.characterData`
   - 立即生成 6 张角色卡片
   - 每张卡片都有 CSS 方块角色 fallback

2. Three.js 模块脚本：
   - 读取 `window.characterData`
   - 找到已经存在的 `.card`
   - 在卡片里的 `<canvas>` 上渲染 3D 模型
   - 成功后给 `.viewer` 加 `.webgl-ready`，隐藏 CSS fallback

这样即使 Three.js 失败，页面仍然能显示角色卡片，不会空白。

## 修改前检查清单

- `#grid` 初始加载后必须有 6 个 `.card`
- 每个 `.card` 必须有 `.fallback-model`
- 每个 `.viewer` 必须有 `<canvas>` 和 `.fallback-model`
- 不要写 `grid.innerHTML = ""` 后只依赖模块脚本重新生成
- 不要删除普通脚本里的 `window.characterData`

## 发布前验证

本地验证：

```bash
node -e "/* start local static server */"
```

浏览器验证要看：

- 桌面尺寸能看到 6 张卡
- 手机尺寸能看到 6 张卡
- DevTools Console 没有红色错误
- WebGL 加载失败时仍能看到 CSS 方块角色

GitHub Pages 验证：

```bash
curl.exe -I "https://echozhangsz-ui.github.io/H-ros-de-la-Fus-e-/character_3d_list.html?v=COMMIT"
```

然后打开：

```text
https://echozhangsz-ui.github.io/H-ros-de-la-Fus-e-/character_3d_list.html?v=COMMIT
```

如果打开后只有标题没有卡片，说明基础渲染结构被破坏，必须先恢复普通 HTML/CSS fallback，再检查 Three.js。
