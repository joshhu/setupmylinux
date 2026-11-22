# setupmylinux

新裝好的 Ubuntu 主機要怎麼設定成我習慣的開發環境：清單、設定檔，以及實際跑過的指令。目前以 **Ubuntu 26.04 LTS 桌面版**為準。

## 內容

| 檔案 | 用途 |
| --- | --- |
| `CLAUDE.md` | 完整設定流程（系統、zsh、語系、固定 IP、軟體清單、RDP、Chrome…），也是給 Claude Code 照著做的指示 |
| `.zshrc` | 我的 zsh 設定（oh-my-zsh、plugins、uv / nvm / brew 的 PATH） |
| `myclean.zsh-theme` | 自訂的 oh-my-zsh theme |
| `x11vnc.service` | 24.04 以前用的 x11vnc systemd unit；26.04 改用內建 RDP，不再需要 |

## 會設定哪些東西

- **系統**：apt 更新、sudo 免密碼、`zh_TW.UTF-8` 語系、關閉休眠、黑色桌面背景、固定 IP（netplan + NetworkManager）
- **Shell**：zsh + oh-my-zsh + zsh-completions / zsh-syntax-highlighting / zsh-autosuggestions + autojump
- **開發工具**：uv、nvm（Node LTS）、Homebrew、gh、Docker、ffmpeg / ffprobe
- **CLI 工具**：htop、btop、ncdu、duf、ripgrep、fd、bat、eza、jq、yq、httpie、wget、mkcert、nmap、tailscale、tmux、fzf、direnv、lazygit、tldr、glow、rsync
- **其他**：Google Chrome、GNOME Remote Desktop（RDP，Remote Login 模式）

## 使用方式

### 用 Claude Code（我平常的做法）

在這個資料夾開 Claude Code，告訴它新主機的 IP、帳號密碼、主機名稱和要固定的 IP，例如：

```
我新裝的 linux 是 192.168.1.124，主機名稱 JoshDev，幫我用 setupmylinux 設定好，固定 IP 改成 192.168.1.178
```

它會照 `CLAUDE.md` 透過 ssh 一步步設定，並在最後驗證每個工具都裝好。

### 手動

照 `CLAUDE.md` 的段落順序在新主機上執行指令即可。設定檔的放法：

```sh
cp .zshrc ~/.zshrc
mkdir -p ~/.oh-my-zsh/custom/themes
cp myclean.zsh-theme ~/.oh-my-zsh/custom/themes/
```

固定 IP 那段會讓 ssh 斷線，建議放到最後做。

## 裝完之後要自己處理的

- 重新登入一次（或重開機），zsh、語系、docker group 才會生效
- `sudo tailscale up` 登入 Tailscale
- `gh auth login` 登入 GitHub

## 測試

這個 repo 只有文件和設定檔，沒有程式碼可以跑測試。驗證方式是在設定完的主機上用 zsh 登入 shell 逐一執行各工具的 `--version`，確認都找得到。
