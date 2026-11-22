# 下面是我新安裝的 Linux，是 Ubuntu 26.04 LTS（桌面版），我要安裝下列東西

## 系統
主機名稱：安裝時就已經設好，**不要改**。每台主機有自己的名字，由我當次指定（例如 JoshDev），不沿用別台的名字。
ssh key / `~/.ssh/config` 的 Host 名稱：用我當次指定的名字；`~/.ssh/config` 已經有該主機的條目就照它的 IdentityFile。
新機器重灌後 host key 會變，先清掉 known_hosts 的舊紀錄：`ssh-keygen -R <IP>`

sudo apt update
sudo apt upgrade

sudo apt install git curl wget zsh autojump build-essential -y
sudo chsh -s $(which zsh) joshhu
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)" "" --unattended

git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-completions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

cp .zshrc ~/.zshrc
cp myclean.zsh-theme ~/.oh-my-zsh/custom/themes/myclean.zsh-theme

## 設定語系
sudo apt install language-pack-zh-hant -y
sudo localectl set-locale LANG=zh_TW.UTF-8

# 我的 oh-my-zsh 的 theme 在 @myclean.zsh-theme
# 我的 .zshrc 就在 @.zshrc 中
## visudo sudo 免密碼
echo "joshhu ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/joshhu && sudo chmod 440 /etc/sudoers.d/joshhu

## 固定 IP
網路由 NetworkManager 管理。改 `/etc/netplan/00-installer-config.yaml`（先備份），
`dhcp4: false`、`addresses`、`routes`（default via 192.168.1.1）、`nameservers`（192.168.1.1, 1.1.1.1），
`renderer: NetworkManager`，然後 `sudo netplan generate && sudo netplan apply`。
改 IP 會斷線，放在最後做，apply 用 nohup 在背景跑。

## 安裝所有重要軟體
1 uv — `curl -LsSf https://astral.sh/uv/install.sh | sh`
2 nvm — 官方 install.sh，裝完 `nvm install --lts && nvm alias default 'lts/*'`
3 brew — `NONINTERACTIVE=1` 官方 install.sh
4 gh — 用 GitHub 官方 apt repo（apt 內建版本太舊）
5 docker — 用 Docker 官方 apt repo（docker-ce + compose plugin），`sudo usermod -aG docker joshhu`
6 ffmpeg — apt
7 ffprobe — ffmpeg 內附

系統與效能監控

htop / btop — 互動式行程監控（btop 更美觀）
ncdu — 磁碟空間分析，找出大檔案
duf — 現代化 df，磁碟使用量一目了然
檔案與文字處理

ripgrep (rg) — 超快速遞迴搜尋，取代 grep
fd — 現代化 find，語法更直覺（apt 套件 fd-find，指令是 fdfind，做捷徑 `ln -s /usr/bin/fdfind ~/.local/bin/fd`）
bat — 有語法高亮的 cat（指令是 batcat，做捷徑 `ln -s /usr/bin/batcat ~/.local/bin/bat`）
eza — 現代化 ls，支援圖示與 git 狀態
jq — JSON 解析與處理，API 開發必備
yq — YAML/JSON/TOML 處理，類似 jq（用 `brew install yq`；apt 的 yq 是另一個 Python 版）
網路與 API

curl / httpie — HTTP 請求工具（httpie 語法更友善）
wget — 檔案下載
mkcert — 本地 HTTPS 憑證生成
nmap — 網路掃描診斷
tailscale — 用 Tailscale 官方 apt repo，裝完由我自己 `sudo tailscale up` 登入
開發工具

tmux — Terminal 多視窗管理，遠端工作必備
fzf — 模糊搜尋，配合其他工具威力大
direnv — 依目錄自動載入 .env
lazygit — Terminal 圖形化 git 操作
其他實用工具

tldr — 簡化版 man page，快速查指令範例（apt 套件 tealdeer）
glow — Terminal 內渲染 Markdown
rsync — 高效遠端/本地檔案同步

以上除了特別註明的，26.04 都直接用 apt 裝：
sudo apt install htop btop ncdu duf ripgrep fd-find bat eza jq httpie mkcert nmap tmux fzf direnv lazygit tealdeer glow rsync ffmpeg -y

## 遠端桌面：RDP（26.04 不換 lightdm、不裝 x11vnc）
26.04 的 GNOME 只剩 Wayland，x11vnc 不適用；改用內建 gnome-remote-desktop 的 Remote Login（system 模式），
在登入畫面就能 RDP 進來，開機不用先本機登入。帳號 joshhu，密碼同登入密碼。

D=/var/lib/gnome-remote-desktop/.local/share/gnome-remote-desktop
sudo -u gnome-remote-desktop mkdir -p $D
sudo -u gnome-remote-desktop openssl req -new -newkey rsa:4096 -days 3650 -nodes -x509 -subj "/CN=<主機名稱>" -out $D/tls.crt -keyout $D/tls.key
sudo grdctl --system rdp set-tls-key $D/tls.key
sudo grdctl --system rdp set-tls-cert $D/tls.crt
sudo grdctl --system rdp set-credentials joshhu <密碼>
sudo grdctl --system rdp enable
sudo systemctl enable --now gnome-remote-desktop.service

注意：
- 「Init TPM credentials failed」可忽略（沒有 TPM，改用 keyfile 存帳密）。
- 同一帳號同時只能有一個桌面 session。本機已登入時 RDP 會跳「remote session is running」，
  選 force stop 後要等 5–10 秒讓舊 session 關完再重連，否則會一片黑只有滑鼠。
  卡住時 `sudo systemctl restart gnome-remote-desktop` 再重連。
- 舊版（24.04 以前）才用 lightdm + x11vnc + x11vnc.service。

## 將背景改成黑色
gsettings set org.gnome.desktop.background picture-options 'none'
gsettings set org.gnome.desktop.background primary-color '#000000'
（從 ssh 設定要加 `DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1000/bus`）

## 關閉系統休眠
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target

## 還要安裝 google chrome
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install ./google-chrome-stable_current_amd64.deb

## 裝完
重新登入一次（或重開機），zsh、語系、docker group 才會生效。
