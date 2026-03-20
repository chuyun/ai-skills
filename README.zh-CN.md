<p align="center">
  <img src="./light.svg" alt="skills.video logo" width="320" />
</p>

# video-skills

这是一个面向 `skills.video` Open Platform API 的 skills 集合仓库。
仓库目标是为 `skills.video` 提供可复用、可直接调用的技能能力。

[English README](./README.md)

当前包含：

- `skills-video-create-video`
- `skills-video-create-image`

## 安装

```bash
npx skills add git@github.com:chuyun/video-skills.git
```

## 首次配置 API Key

首次调用前，请先配置 `SKILLS_VIDEO_API_KEY`：

1. 打开 `[https://skills.video/dashboard/developer](https://skills.video/dashboard/developer)` 并登录。
2. 点击 `Create API Key`。
3. 在终端导出环境变量：

```bash
export SKILLS_VIDEO_API_KEY="<YOUR_API_KEY>"
```

如需持久化到 zsh：

```bash
echo 'export SKILLS_VIDEO_API_KEY="<YOUR_API_KEY>"' >> ~/.zshrc
source ~/.zshrc
```

## 使用方式

### 调用时遇到额度不足

如果 API 调用失败且怀疑是额度不足，可以用下面命令查询当前额度：

```bash
curl -X GET "https://open.skills.video/api/v1/credits" \
  -H "Authorization: Bearer $SKILLS_VIDEO_API_KEY"
```

### 在 Codex 中调用技能

- `Use $skills-video-create-video to ...`
- `Use $skills-video-create-image to ...`
