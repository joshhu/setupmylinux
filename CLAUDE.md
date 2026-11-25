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
- 舊版（24.04 以前）才用 lightdm + x11vnc + x11vnc.service，見下一段。

## 遠端桌面：VNC（24.04 用這個，例如 DGX OS）
先看版本：`grep PRETTY /etc/os-release`。24.04 → VNC；26.04 → 上面的 RDP。
echo "lightdm shared/default-x-display-manager select lightdm" | sudo debconf-set-selections
sudo DEBIAN_FRONTEND=noninteractive apt install -y lightdm x11vnc
echo /usr/sbin/lightdm | sudo tee /etc/X11/default-display-manager && sudo DEBIAN_FRONTEND=noninteractive dpkg-reconfigure lightdm
sudo x11vnc -storepasswd <密碼> /home/joshhu/.vnc/passwd     # VNC 密碼同登入密碼（只取前 8 字元）；記得 chown joshhu
sudo cp x11vnc.service /etc/systemd/system/ && sudo systemctl enable x11vnc.service
重開機後生效（會關掉本機目前的 gdm 桌面）。
- repo 的 x11vnc.service 必須有 `Restart=always`：使用者登入時 X 授權會換，x11vnc 會斷一次，靠它自動重啟（JoshDGX2 實測）。
- 登入後 session 是 X11（lightdm 用 /usr/share/xsessions），x11vnc 接 `:0`。

## DGX Spark（DGX OS = Ubuntu 24.04 arm64）注意事項
- aarch64：Chrome 用 `google-chrome-stable_current_arm64.deb`；gh / tailscale 官方 repo 都有 arm64。
- Docker + NVIDIA Container Toolkit 已內建，不重裝，只要 `usermod -aG docker joshhu`。
- noble 的 apt 沒有 lazygit、glow → 跟 yq 一樣用 brew 裝。CUDA 在 /usr/local/cuda-13.0，.zshrc 路徑可直接用。
- 系統語系是中文，用 `apt-cache policy` 判斷套件要加 `LC_ALL=C`（否則是「候選：」不是 Candidate）。
- 有線網路是 NetworkManager 自己的連線（不在 netplan），固定 IP 用 nmcli：
  sudo nmcli con mod "<有線連線名稱>" ipv4.method manual ipv4.addresses <IP>/24 ipv4.gateway 192.168.1.1 ipv4.dns "192.168.1.1 1.1.1.1" && sudo nmcli con up "<有線連線名稱>"
- 有線和 WiFi 可能同時連著（WiFi metric 較高，當備援）。
- 已設好：JoshDGX2（192.168.1.93 固定 / Tailscale 100.104.113.48），金鑰 `~/.ssh/JoshDGX2`。

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

---

# macOS 主機（Apple Silicon）

不是 Linux，Linux 系統層的東西（apt、語系、netplan、RDP、休眠、Chrome、sudo 免密碼）都不做。
只做 zsh、Homebrew、Ghostty，以及上面清單裡 macOS 也有的 CLI，全部用 brew 裝。

## 連線
- `.env` 三行：IP、帳號、密碼（`.env` 已 gitignore，不可 commit）。密碼只從 `.env` 讀，用 stdin 餵給 sudo / SSH_ASKPASS，不要寫進指令。
- 主機名稱已設好不改；`~/.ssh/config` 加一筆 Host（小寫主機名稱），Mac 都用 `JoshWorkToNas` 金鑰。
- 需要 sudo 的步驟（Homebrew 安裝）暫時放 `/etc/sudoers.d/zz-temp-setup`，**裝完就刪掉**；Mac 不保留 sudo 免密碼。

## Homebrew + zsh
NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"   # 會順便裝 Command Line Tools
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> ~/.zprofile
brew install git autojump
brew install --cask ghostty
oh-my-zsh（--unattended）+ 三個 plugins + myclean theme 同 Linux
cp .zshrc.macos ~/.zshrc     # Mac 版：brew 在 oh-my-zsh 之前載入、無 CUDA、nvm 從 /opt/homebrew/opt/nvm 載入

## CLI
brew install uv nvm gh ffmpeg yt-dlp htop btop ncdu duf ripgrep fd bat eza jq yq httpie wget mkcert nmap tmux fzf direnv lazygit tealdeer glow rsync mole
mkdir -p ~/.nvm && nvm install --lts && nvm alias default 'lts/*'
（Mac 上 fd / bat 直接就是原名，不用做捷徑；mole 是 Mac 清理工具，指令 `mole` / `mo`）

## Container：用 Apple container，不裝 Docker
brew install container
container system start --enable-kernel-install
brew services start container     # 登入時自動 `container system start`（使用者層級，要登入才會跑；brew services 顯示 stopped 是正常的，它跑完就結束）
測試：`container run --rm docker.io/library/alpine:latest uname -a`

## Tailscale：用 brew 版 tailscaled（系統 LaunchDaemon），不用 GUI App
不用登入桌面、開機就啟動。GUI 版（App Store / 官網）要登入桌面才會跑，且會跟 tailscaled 衝突，兩者擇一。
brew install tailscale
sudo brew services start tailscale          # /Library/LaunchDaemons/sh.brew.tailscale.plist
sudo tailscale up --hostname=<小寫主機名稱>   # 把授權網址給我，在瀏覽器核准
管理後台對這台選「Disable key expiry」。

## 無人值守遠端 Mac（關蓋、不接螢幕，只接電源 + WiFi，從台灣用 Tailscale 連）
最高原則：**Tailscale 永遠連著比什麼都重要**。所以只用 brew 版 tailscaled（唯一能在登入前跑的版本），不裝 Tailscale GUI App。
已在 JoshMacUSA（MacBook Air M1, macOS 27）驗證：關蓋、無螢幕下 SSH 與螢幕共享都能透過 Tailscale 連入。

- FileVault 必須關閉（`fdesetup status`），否則重開機卡在解鎖畫面、沒網路。
- 電源：
  sudo pmset -a disablesleep 1      # 關蓋不睡（M1 實測有效）
  sudo pmset -a sleep 0 disksleep 0 standby 0 powernap 0 hibernatemode 0 womp 1 tcpkeepalive 1 ttyskeepawake 1 lowpowermode 0 displaysleep 10
  sudo systemsetup -setrestartfreeze on
  斷電後：nvram 不要動，Apple silicon 筆電預設接上電源就開機（不要設 BootPreference）。
- 螢幕共享（不用 GUI）：
  sudo launchctl enable system/com.apple.screensharing
  sudo launchctl bootstrap system /System/Library/LaunchDaemons/com.apple.screensharing.plist   # 回「Bootstrap failed: 5」也沒關係，5900 有在聽就好
  檢查：從外面讀 5900 要回 `RFB 003.889`（非 root 的 lsof 看不到 launchd 的 socket）。
  **不開傳統 VNC 密碼**（Windows VNC Viewer 會報「Key length too large」，但我只用 Mac 連）。要關掉時：
  sudo .../ARDAgent.app/Contents/Resources/kickstart -configure -clientopts -setvnclegacy -vnclegacy no && sudo rm -f /Library/Preferences/com.apple.VNCSettings.txt
- 自動登入：`sysadminctl -autologin set` 從 SSH 會 error 22、不產生 /etc/kcpassword。
  改成 `defaults write /Library/Preferences/com.apple.loginwindow autoLoginUser joshhu` + 手動產生 /etc/kcpassword
  （密碼補 NUL 到 12 的倍數，與 key 7D 89 52 23 D2 BC DD EA A3 B9 1F 循環 XOR；root:wheel 600）。
- 自動更新：只自動裝安全性回應與系統資料檔，macOS 版本更新不自動裝：
  /Library/Preferences/com.apple.SoftwareUpdate 設 AutomaticCheckEnabled/AutomaticDownload/CriticalUpdateInstall/ConfigDataInstall = true，AutomaticallyInstallMacOSUpdates = false
- 虛擬顯示器：`brew install --cask betterdisplay`，用 CLI 建虛擬螢幕（名稱統一「虛擬 16:9」）：
  open /Applications/BetterDisplay.app        # 剛裝完 `open -a` 會找不到（LaunchServices 還沒更新），用完整路徑
  B=/opt/homebrew/bin/betterdisplaycli
  $B create -type=VirtualScreen -virtualScreenName="虛擬 16:9" -aspectWidth=16 -aspectHeight=9 -virtualScreenHiDPI=on
  $B set -name="虛擬 16:9" -connected=on && $B set -name="虛擬 16:9" -resolution=2560x1440 -hiDPI=on
  **一定要在機器前跑完 BetterDisplay 第一次的引導畫面**，否則重開機後虛擬螢幕不會自動接上，且 betterdisplaycli 會一直卡住。
  登入自動啟動用 LaunchAgent `~/Library/LaunchAgents/local.betterdisplay.launch.plist`（`/usr/bin/open -g /Applications/BetterDisplay.app`，LimitLoadToSessionType=Aqua，RunAtLoad）。
  從 Mac 用螢幕共享「高效能」模式時，會暫時改用它自己建的虛擬顯示器。
- WiFi 登入前連線：WiFi 密碼要在**系統**鑰匙圈（從選單列加入的就是）。到新地點的 WiFi 要先在現場加好，否則開機沒網路就再也連不回來。
- `~/.ssh/config` 的 HostName 用 **Tailscale IP**（區網 DHCP 會換，到美國也沒用）。
- 全新帳號沒有 `~/Library/LaunchAgents`，寫 plist 前先 `mkdir -p`。
- `~/.ssh/authorized_keys` 只能**附加**、不能覆蓋（JoshMacUSA 曾被覆蓋成只剩 joshmba 的金鑰，導致 Windows 金鑰登入失效）。
- 這台 Windows 的 Tailscale 沒開時，可拿 tailnet 上另一台 Mac 當跳板：`ssh -o IdentitiesOnly=yes -i ~/.ssh/JoshWorkToNas -J <跳板> joshhu@<Tailscale IP>`。
- 已設好的無人值守 Mac：JoshMacUSA（100.66.89.93）、JoshMacM1（100.119.159.113），皆 MacBook Air M1、macOS 27。
- 要在機器前用 GUI 做的：充電上限 80%（電池 → 充電 ⓘ）、確認螢幕共享允許 joshhu、BetterDisplay 首次引導。
- Apple container 的 apiserver 依 session 分 domain（SSH = user/、桌面 = gui/），只能服務其中一邊；目前用 `~/Library/LaunchAgents/local.container.system-start.plist`（LimitLoadToSessionType=Background）給 SSH 用。不重要，可不設。
