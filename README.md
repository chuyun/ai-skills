<p align="center">
  <img src="./light.svg" alt="skills.video logo" width="320" />
</p>

# video-skills

A collection of skills for the `skills.video` Open Platform API.
This repository is built specifically to provide reusable `skills.video` skills.

[中文说明](./README.zh-CN.md)

Currently included:

- `skills-video-create-video`
- `skills-video-create-image`

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

### Check API key

```bash
python /Users/jun/Documents/Development/blendduck/video-skills/skills-video-create-video/scripts/ensure_api_key.py
python /Users/jun/Documents/Development/blendduck/video-skills/skills-video-create-image/scripts/ensure_api_key.py
```

If key is missing, the script prints dashboard URL and setup steps.

### Handle insufficient credits during calls

If an API call fails and you suspect low credits, classify the runtime error:

```bash
python /Users/jun/Documents/Development/blendduck/video-skills/skills-video-create-video/scripts/handle_runtime_error.py \
  --status 402 \
  --body '{"message":"Insufficient credits"}'
```

Then guide user to recharge at `https://skills.video/dashboard` (Billing/Credits).
You can check current balance with:

```bash
curl -X GET "https://open.skills.video/api/v1/credits" \
  -H "Authorization: Bearer $SKILLS_VIDEO_API_KEY"
```

### Invoke skills in Codex

- `Use $skills-video-create-video to ...`
- `Use $skills-video-create-image to ...`

## Validate scripts

```bash
python3 -m py_compile /Users/jun/Documents/Development/blendduck/video-skills/skills-video-create-video/scripts/*.py
python3 -m py_compile /Users/jun/Documents/Development/blendduck/video-skills/skills-video-create-image/scripts/*.py
```
