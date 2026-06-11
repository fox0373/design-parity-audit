# Design Parity Audit Skill

Codex skill for auditing a development UI against Figma designs at feature-module level.

## Install

After this repository is uploaded to GitHub, install the skill with:

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py --repo OWNER/REPO --path design-parity-audit
```

Or ask Codex:

```text
安装 https://github.com/OWNER/REPO/tree/main/design-parity-audit
```

Restart Codex after installation.

## Usage

```text
用 design-parity-audit 做这个模块的设计走查：
开发地址：<development URL>
Figma：<Figma design URL>
模块范围：<module name>
```

The report compares Figma and development screenshots, marks issues on the development screenshot, supports page tabs, zoom, drag panning, mouse-wheel zoom, and includes concrete fix recommendations.
