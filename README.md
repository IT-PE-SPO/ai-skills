# dotfiles

Claude Code skills and configuration.

## 설치

```bash
git clone git@github.com:cnbsoft-com/dotfiles.git ~/dotfiles
mkdir -p ~/.claude
ln -s ~/dotfiles//skills/* ~/.claude/skills/
```

## Skills

| 스킬 | 설명 |
|------|------|
| `/uml` | 유스케이스·클래스·시퀀스 다이어그램 생성 및 HTML 보고서 |

## 사전 조건

```bash
# Mermaid CLI (mmdc) 필요
npm install -g @mermaid-js/mermaid-cli
```
