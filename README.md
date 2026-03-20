<p align="center">
  <img src="./light.svg" alt="skills.video logo" width="320" />
</p>

# video-skills

A collection of skills for the `skills.video` Open Platform API.
This repository is built specifically to provide reusable `skills.video` skills.

[中文说明](./README.zh-CN.md)

Currently included:

- `ai-video-skills`
- `ai-image-skills`

## Install

```bash
npx skills add git@github.com:chuyun/video-skills.git
```

## First-time API key setup

Before first API call, configure `SKILLS_VIDEO_API_KEY`:

1. Open `[https://skills.video/dashboard/developer](https://skills.video/dashboard/developer)` and sign in.
2. Click `Create API Key`.
3. Export the key:

```bash
export SKILLS_VIDEO_API_KEY="<YOUR_API_KEY>"
```

Persist for zsh:

```bash
echo 'export SKILLS_VIDEO_API_KEY="<YOUR_API_KEY>"' >> ~/.zshrc
source ~/.zshrc
```

## Usage

### Handle insufficient credits during calls

If an API call fails and you suspect low credits, You can check current balance with:

```bash
curl -X GET "https://open.skills.video/api/v1/credits" \
  -H "Authorization: Bearer $SKILLS_VIDEO_API_KEY"
```

### Invoke skills in Codex

- `Use $ai-video-skills to ...`
- `Use $ai-image-skills to ...`
