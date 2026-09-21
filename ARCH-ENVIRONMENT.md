# My Arch Linux Environment

> Lenovo IdeaPad Slim 3 14AHP10 개발용 Arch Linux 환경 문서
>
> 목적: **환경 인수인계 + Linux 학습 + 재현 가능한 환경 구성 + 확장 가능한 운영 문서**

---

## 0. 문서 사용 규칙

이 문서는 이 노트북의 **Single Source of Truth(SSOT)** 역할을 한다.

다음 에이전트/사람은 작업을 시작하기 전에 이 문서를 먼저 읽는다.

### 작업 원칙

1. 이미 완료된 작업을 다시 실행하지 않는다.
2. 실제 시스템 상태가 필요한 경우 문서 내용만 믿지 말고 **검증 명령으로 확인**한다.
3. `완료`와 `계획`을 절대 혼동하지 않는다.
4. 새로운 설정을 추가할 때는 **왜 필요한지**를 먼저 설명한다.
5. KISS 원칙을 유지한다. 문제가 없으면 설정을 추가하지 않는다.
6. Arch의 공식 문서/ArchWiki를 우선 참고한다.
7. 시스템 전체를 자동 변경하기보다 **자동 점검 → 사용자 승인 → 변경** 흐름을 선호한다.
8. 변경 후에는 아래의 `Change Log`를 업데이트한다.

---

# 1. 사용 목적

이 시스템은 다음 목적의 **개인 개발용 Linux 노트북**이다.

- Backend
- Spring Boot
- RAG / AI application development
- MCP / MCP server development
- Frontend
- Full-stack development
- Linux / systemd / networking / filesystem 학습
- 나만의 재현 가능한 개발 OS 구축

핵심 목표는 단순히 "Arch를 설치한다"가 아니라:

> **Linux를 이해하면서 필요한 구성요소만 조립하고, 이후 같은 환경을 다시 만들 수 있는 나만의 개발 OS를 만드는 것**

이다.

---

# 2. 사용자/설계 철학

## 핵심 원칙

### KISS

Keep It Simple, Stupid.

- 필요한 것만 설치
- 문제가 없으면 튜닝하지 않음
- 공식 패키지 우선
- 불필요한 daemon / framework 최소화

### Arch를 선택한 이유

- 미니멀함
- KISS 철학
- 공식 ArchWiki의 높은 문서 품질
- 개발자 친화성
- Linux 자체를 공부할 수 있음
- 시스템을 직접 설계할 수 있음
- 최종적으로 "나만의 OS"를 만들고 싶음

### 언어 정책

- 시스템 UI: **English**
- locale: **en_US.UTF-8 + ko_KR.UTF-8 준비**
- 한국어 입력: **Fcitx5 + Hangul**
- 개발/에러/문서 검색은 가능한 English 중심

---

# 3. 하드웨어 정보

## 노트북

**Lenovo IdeaPad Slim 3 14AHP10-83K9000CKR**

| 항목 | 사양 |
|---|---|
| CPU | AMD Ryzen 7 8840HS |
| CPU | 8 Cores / 16 Threads |
| CPU Clock | 3.3GHz, up to 5.1GHz |
| Cache | 16MB |
| GPU | AMD Radeon 780M |
| Display | 14" WUXGA 1920x1200 IPS |
| Display | 300 nits, 45% NTSC |
| RAM (original) | 8GB DDR5-5600 onboard |
| RAM Slot | 1 x SO-DIMM |
| Planned RAM | **16GB SO-DIMM 추가 → 총 24GB** |
| SSD (original) | 512GB M.2 2242 PCIe 4.0 |
| Additional SSD | 추가 M.2 2242 PCIe 4.0 슬롯 사용 예정 가능 |
| Wi-Fi | Wi-Fi 7, 2x2 |
| Bluetooth | Bluetooth 5.4 |
| Camera | FHD 1080p + IR |
| Battery | 60Wh |
| Weight | 약 1.43kg |
| Keyboard | Korean engraving, backlight |
| OS at purchase | FreeDOS |

### 하드웨어 관련 판단

- AMD Radeon 780M → Linux에서는 AMDGPU + Mesa 기본 구성을 사용
- NVIDIA proprietary driver 불필요
- Wi-Fi 7 장치의 Linux 지원은 최신 kernel/driver 상태가 중요하므로 업데이트 상태를 유지
- RAM 24GB가 개발 workload에 적합한 목표 구성
- 내장 GPU가 시스템 RAM을 공유하므로 메모리 여유가 중요

---

# 4. 현재 OS 구조

## Boot

```text
UEFI
  ↓
systemd-boot
  ↓
Arch Linux
```

## Disk

현재 설계:

```text
NVMe 512GB
├── EFI System Partition
│   ├── FAT32
│   └── 약 1 GiB
│       └── /boot
│
└── Root
    ├── ext4
    └── 나머지 전체
        └── /
```

현재 초기 설계에서는:

- `/home` 별도 파티션 없음
- 별도 swap partition 없음
- LVM 미사용
- Btrfs 미사용
- LUKS 미사용

이유: **처음에는 구조를 단순하게 유지하고 Linux 구성요소를 직접 이해하기 위함**.

> 암호화/LUKS/Btrfs는 필요성과 학습 목적이 분명해지는 시점에 별도 설계한다.

---

# 5. 현재 시스템 계층

```text
Arch Linux
│
├── UEFI
│   └── systemd-boot
│
├── Kernel
│   ├── linux
│   ├── linux-firmware
│   └── amd-ucode
│
├── Base services
│   ├── systemd
│   ├── NetworkManager
│   ├── Bluetooth / BlueZ
│   ├── PipeWire / WirePlumber
│   └── power-profiles-daemon
│
├── Graphics
│   ├── AMDGPU
│   ├── Mesa
│   └── Vulkan
│
├── Desktop
│   ├── KDE Plasma
│   ├── Wayland
│   └── SDDM
│
├── Input / Locale
│   ├── en_US.UTF-8
│   ├── ko_KR.UTF-8
│   ├── Fcitx5
│   └── Hangul
│
└── Developer CLI
    ├── Git
    ├── OpenSSH
    ├── Zsh
    ├── tmux
    ├── ripgrep
    ├── fd
    ├── fzf
    ├── Neovim
    └── mise
```

---

# 6. 진행 상태

> 상태 기준: **대화에서 사용자에 의해 완료로 보고된 범위**와 **계획만 잡힌 범위**를 분리한다.
> 실제 상태가 불확실한 항목은 `검증 필요`로 표시한다.

## 6.1 설치/부팅

| 항목 | 상태 | 비고 |
|---|---|---|
| FreeDOS 제거 | ✅ 완료 | Arch 전용 사용 |
| UEFI 부팅 | ✅ 완료 | UEFI 기준 설치 |
| GPT | ✅ 완료 | EFI + root |
| EFI partition | ✅ 완료 | `/boot` |
| ext4 root | ✅ 완료 | `/` |
| pacstrap | ✅ 완료 | base system 구성 |
| fstab | ✅ 완료 | UUID 기반 |
| systemd-boot | ✅ 완료 | `/boot` |
| 첫 부팅 | ✅ 완료 | TTY 로그인까지 도달 |

## 6.2 GUI / 노트북 기본기

| 항목 | 상태 | 비고 |
|---|---|---|
| AMDGPU | ✅ 구성 | Radeon 780M |
| Mesa / Vulkan | ✅ 구성 | 기본 AMDGPU stack |
| KDE Plasma | ✅ 구성 | Plasma + Wayland |
| SDDM | ✅ 구성 | 로그인 화면 |
| Wayland | ✅ 구성 | X11 별도 설치하지 않음 |
| NetworkManager | ✅ 구성 | 기본 네트워크 관리 |
| PipeWire | ✅ 구성 | WirePlumber |
| Bluetooth | ✅ 구성 | BlueZ + Bluedevil |
| Power management | ✅ 구성 | power-profiles-daemon |
| Korean font | ✅ 구성 | Noto CJK 계열 |
| Fcitx5 | ✅ 구성 | Hangul |
| Firefox | ✅ 설치 | 기본 브라우저 |

## 6.3 기본 개발 도구

| 항목 | 상태 | 비고 |
|---|---|---|
| Git | ✅ 구성 | global config 사용 |
| OpenSSH client | ✅ 구성 | GitHub SSH용 |
| Ed25519 key | ✅ 구성 | private key는 Git에 넣지 않음 |
| Zsh | ✅ 구성 | 최소 `.zshrc` |
| tmux | ✅ 구성 | 최소 `tmux.conf` |
| ripgrep | ✅ 구성 | `rg` |
| fd | ✅ 구성 | `fd` |
| fzf | ✅ 구성 | Zsh integration |
| Neovim | ✅ 구성 | 최소 `init.lua` |
| mise | ✅ 구성 | runtime manager |

## 6.4 시스템 유지관리

사용자가 **7번 항목까지 완료했다고 보고함**. 아래는 그 범위의 기록이다.

| 항목 | 상태 | 비고 |
|---|---|---|
| fwupd | ✅ 완료 보고 | 펌웨어 확인/업데이트 체계 |
| fwupd refresh | ✅ 완료 보고 | metadata refresh |
| fstrim timer | ✅ 완료 보고 | SSD TRIM |
| zram | ✅ 완료 보고 | 약 4GiB 목표 구성 |
| firewalld | ✅ 완료 보고 | 기본 방화벽 |
| time sync 확인 | ✅ 완료 보고 | timedatectl 기준 |
| pacman mirror 기본 점검 | ✅ 완료 보고 | 문제 없으면 유지 |

> 주의: 위 항목은 현재 시스템을 직접 조회해서 검증한 것이 아니라, 사용자가 완료했다고 보고한 상태다. 다음 세션에서 변경/문제 진단이 필요하면 아래 검증 명령을 먼저 실행한다.

---

# 7. 현재 사용 중인 핵심 명령

## 시스템 정보

```bash
uname -r
cat /etc/os-release
lsblk -f
findmnt
bootctl status
```

## GUI / Wayland

```bash
echo $XDG_SESSION_TYPE
echo $XDG_CURRENT_DESKTOP
```

## GPU

```bash
lspci -k | grep -A 3 -E 'VGA|3D|Display'
glxinfo -B
vulkaninfo --summary
```

기대값 핵심:

```text
Kernel driver in use: amdgpu
XDG_SESSION_TYPE=wayland
```

## 네트워크

```bash
nmcli device
nmcli connection show
ip addr
```

## 오디오

```bash
wpctl status
systemctl --user status pipewire
systemctl --user status wireplumber
```

## Bluetooth

```bash
systemctl status bluetooth --no-pager
```

## 전원

```bash
powerprofilesctl
systemctl status power-profiles-daemon --no-pager
```

## 실패한 systemd unit

```bash
systemctl --failed
```

---

# 8. Git / SSH 구성

## Git

주요 설정:

```bash
git config --global user.name "YOUR_NAME"
git config --global user.email "YOUR_EMAIL"
git config --global init.defaultBranch main
git config --global core.editor nvim
git config --global fetch.prune true
```

## SSH

기본 파일:

```text
~/.ssh/
├── config
├── id_ed25519
└── id_ed25519.pub
```

GitHub host 설정:

```sshconfig
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes
```

테스트:

```bash
ssh -T git@github.com
```

### 보안 규칙

- `id_ed25519`는 절대 Git에 커밋하지 않는다.
- `id_ed25519.pub`만 외부 서비스에 등록한다.
- `.ssh/config` 권한은 `600`.
- `~/.ssh` 권한은 `700`.

---

# 9. Shell / Terminal

## Zsh

기본 shell:

```text
/usr/bin/zsh
```

설정 파일:

```text
~/.zshrc
```

현재 철학:

- Oh My Zsh 미사용
- framework 최소화
- 필요한 plugin만 추가

현재 주요 기능:

- history
- completion
- `EDITOR=nvim`
- fzf integration
- mise activation
- 최소 alias

## tmux

설정:

```text
~/.config/tmux/tmux.conf
```

개념:

```text
Terminal
└── tmux
    ├── window 1
    ├── window 2
    └── window 3
```

개발환경 구성 후 용도:

```text
backend
frontend
infra
shell
```

## Search tools

```text
rg  → 파일 내용 검색
fd  → 파일명 검색
fzf → fuzzy selection
```

---

# 10. Neovim

설정:

```text
~/.config/nvim/init.lua
```

현재 목표:

- Neovim 기본 동작 이해
- Vim motions 학습
- Lua 설정 학습
- 이후 필요할 때만 plugin manager / LSP / completion 추가

현재 기본 설정 방향:

- line number
- relative number
- indentation
- search
- mouse
- UTF-8
- clipboard
- leader = Space

---

# 11. mise

현재 역할:

> 프로젝트별 runtime/tool version 관리

목표 runtime:

```text
Java
Node.js
Python
Go
Rust
```

현재 중요한 원칙:

- 시스템 전체에 여러 runtime 버전을 무분별하게 설치하지 않는다.
- 프로젝트별 version declaration을 우선한다.
- 실제 개발 runtime 구성은 개발환경 단계에서 진행한다.

---

# 12. 백업 구조 설계

## 목표

백업을 하나로 생각하지 않고 세 계층으로 분리한다.

```text
1. dotfiles
2. secrets
3. actual data
```

## 12.1 dotfiles

목표 구조:

```text
GitHub
└── dotfiles
    ├── .zshrc
    ├── .gitconfig
    └── .config/
        ├── nvim/
        └── tmux/
```

로컬 bare repository 방식:

```text
~/.local/share/dotfiles.git/
```

alias:

```bash
alias dot='git --git-dir=$HOME/.local/share/dotfiles.git --work-tree=$HOME'
```

원칙:

- 설정 파일만 저장
- private key는 저장하지 않음
- 공개 가능하면 public repo도 가능하지만 초기에는 private repo 권장

## 12.2 arch-setup

목표:

> Arch OS를 재설치한 후 동일한 개발 환경을 재현할 수 있게 한다.

구조:

```text
arch-setup/
├── README.md
├── packages/
│   ├── native-explicit.txt
│   ├── foreign-explicit.txt
│   └── orphans.txt
├── scripts/
└── systemd/
```

패키지 목록:

```bash
pacman -Qqent > packages/native-explicit.txt
pacman -Qqem  > packages/foreign-explicit.txt
pacman -Qqdt  > packages/orphans.txt
```

## 12.3 secrets

구조:

```text
GitHub
└── secrets (private)
    ├── .sops.yaml
    └── *.sops.*
```

암호화:

```text
SOPS + age
```

age private key:

```text
~/.config/sops/age/keys.txt
```

### 절대 Git에 넣지 않는 것

```text
~/.config/sops/age/keys.txt
~/.ssh/id_ed25519
```

age private key는 오프라인 recovery copy도 별도로 보관한다.

## 12.4 Actual data

Git으로 실제 데이터를 백업하지 않는다.

예정:

```text
restic
  ↓
외장 SSD / NAS / cloud repository
```

대상 예:

```text
~/workspace
~/Documents
~/Pictures
필요한 ~/.config / ~/.ssh / 시스템 설정
```

제외 예:

```text
~/.cache
node_modules
 target
.venv
각종 build artifact
```

---

# 13. 자동화 정책

## 자동화한다

| 작업 | 자동화 방법 |
|---|---|
| fwupd metadata refresh | systemd timer |
| 패키지 업데이트 확인 | checkupdates + user systemd timer |
| pacman cache cleanup | paccache timer |
| 데이터 백업 | restic + systemd timer |
| backup 결과 알림 | KDE notification |

## 자동화하지 않는다

| 작업 | 이유 |
|---|---|
| 무인 `pacman -Syu` | rolling release + 의존성/문제 확인 필요 |
| 무인 AUR 업데이트 | PKGBUILD / 재빌드 결과 확인 필요 |
| 무인 firmware install | 재부팅/firmware 상태 영향 |
| 비밀정보 자동 평문 생성 | 보안상 부적절 |

### 선호하는 흐름

```text
자동 점검
   ↓
사용자 알림
   ↓
사용자 승인
   ↓
업데이트
   ↓
검증
   ↓
변경 기록
```

---

# 14. 향후 개발환경 로드맵

## Phase A — 기본 시스템 ✅

- Arch base
- UEFI/systemd-boot
- ext4
- NetworkManager
- KDE / Wayland
- AMDGPU / Mesa
- Fcitx5 / Hangul
- PipeWire
- Bluetooth
- Power management

## Phase B — 개발 CLI ✅ / 진행 완료 범위

- Git
- SSH
- Zsh
- tmux
- ripgrep
- fd
- fzf
- Neovim
- mise

## Phase C — 백업/재현 🔄

- dotfiles Git
- arch-setup Git
- SOPS + age
- restic
- systemd timers
- restore test

## Phase D — Language Runtime 예정

- JDK 21
- Gradle / Gradle Wrapper
- Python
- uv
- Node.js
- pnpm
- Go
- Rust

## Phase E — Container / Infra 예정

- Docker Engine
- Docker Compose
- PostgreSQL
- pgvector
- Valkey
- Qdrant (필요 시)
- MinIO (필요 시)

## Phase F — AI / RAG 예정

- Ollama
- embedding model
- reranker
- RAG service
- evaluation
- local / remote LLM integration

## Phase G — MCP 예정

- TypeScript MCP
- Python MCP
- Java/Spring MCP
- MCP server packaging
- MCP client testing

## Phase H — Frontend 예정

- React
- Next.js
- TypeScript
- pnpm workspace

---

# 15. 학습 로드맵

환경 구성과 Linux 공부를 분리하지 않는다.

## Linux fundamentals

1. UEFI / EFI System Partition
2. GPT / partition table
3. filesystem / ext4
4. mount / fstab
5. initramfs
6. kernel / firmware / microcode
7. systemd
8. systemd service / timer
9. journalctl
10. process / signal
11. permission / ownership
12. users / groups / sudo
13. networking / DNS
14. NetworkManager
15. Wayland / compositor / display manager
16. PipeWire / audio stack
17. power management
18. package management / pacman
19. AUR / PKGBUILD
20. backups / restore

## 꼭 직접 이해할 질문

- UEFI가 무엇을 하는가?
- systemd-boot는 kernel을 어떻게 찾는가?
- `root=UUID=...`는 무엇을 의미하는가?
- initramfs는 왜 필요한가?
- systemd가 PID 1인 이유는 무엇인가?
- NetworkManager는 무엇을 관리하는가?
- Wayland와 X11은 무엇이 다른가?
- AMDGPU와 Mesa의 역할 차이는 무엇인가?
- PipeWire / WirePlumber는 각각 무엇을 하는가?
- `pacman -Syu`가 왜 중요한가?
- AUR은 공식 repo와 무엇이 다른가?
- zram은 swap과 어떻게 다른가?
- Git과 backup은 왜 별개인가?
- SOPS/age와 restic은 어떤 문제를 각각 해결하는가?

---

# 16. 다음 에이전트를 위한 작업 규칙

새 에이전트가 이 환경을 이어받는다면 아래 순서로 작업한다.

### 첫 응답 전에 확인

```bash
uname -r
lsblk -f
bootctl status
systemctl --failed
pacman -Q | wc -l
```

그리고 필요한 경우:

```bash
echo $XDG_SESSION_TYPE
echo $XDG_CURRENT_DESKTOP
lspci -k | grep -A 3 -E 'VGA|3D|Display'
nmcli device
```

### 사용자가 명령어 실행 결과를 전달하면

1. 결과를 해석한다.
2. 현재 상태를 이 문서의 어느 단계와 연결한다.
3. 다음 명령을 최소 단위로 제시한다.
4. 위험한 명령(`mkfs`, `fdisk`, `rm -rf`, 파티션 변경 등)은 대상과 결과를 먼저 설명한다.
5. 같은 패키지를 다시 설치하라고 반복하지 않는다.
6. 공식 ArchWiki를 우선 사용한다.

### 중요한 커뮤니케이션 원칙

- 한 번에 너무 많은 변경을 하지 않는다.
- 명령어마다 목적을 설명한다.
- 학습 포인트를 함께 남긴다.
- 오류가 발생하면 먼저 현 상태를 보존하고 원인을 찾는다.
- 정상 동작 중인 부분은 건드리지 않는다.

---

# 17. 변경 이력

## 2026-09

- FreeDOS 기반 Lenovo IdeaPad Slim 3 14AHP10에 Arch Linux 설치
- UEFI + GPT + ext4 + systemd-boot 구성
- KDE Plasma + Wayland 구성
- AMDGPU / Mesa 구성
- 한국어 locale / Fcitx5 / Hangul 구성
- Firefox 설치
- Git / SSH / Zsh / tmux / ripgrep / fd / fzf / Neovim / mise 구성
- fwupd / fstrim / zram / firewalld / time sync / mirror 유지관리 구성
- 백업 아키텍처 설계

---

# 18. 현재 미완료 / 다음 우선순위

## Backup

- [ ] dotfiles 실제 GitHub remote 연결/확인
- [ ] arch-setup GitHub remote 연결/확인
- [ ] SOPS + age 실제 secret file 생성
- [ ] age recovery key 오프라인 백업
- [ ] restic repository 생성
- [ ] restic 첫 백업
- [ ] restore test
- [ ] backup systemd timer

## Development

- [ ] JDK 21
- [ ] Gradle
- [ ] Python + uv
- [ ] Node.js + pnpm
- [ ] Go
- [ ] Rust
- [ ] Docker
- [ ] PostgreSQL + pgvector
- [ ] Valkey
- [ ] Ollama

## Hardening / deeper Linux

- [ ] LUKS 여부 결정
- [ ] secure boot 여부 결정
- [ ] suspend/resume 장기 테스트
- [ ] battery / power tuning
- [ ] firmware 상태 정기 점검

---

# 19. 설계 결정 기록

## 왜 Arch?

KISS, 최소 구성, ArchWiki, Linux 학습, 높은 커스터마이징, 나만의 OS 구축 목적.

## 왜 KDE?

이 노트북에서 디스플레이/전원/네트워크/Bluetooth/한국어/외부 모니터를 직접 조립하는 유지보수 부담을 줄이면서도 Plasma/Wayland 구조를 학습할 수 있기 때문.

## 왜 systemd-boot?

UEFI 전용 노트북에서 단순한 부트 구성을 유지하고 systemd 생태계를 직접 이해하기 위함.

## 왜 ext4?

첫 구성에서는 filesystem 자체의 복잡도를 낮추고 Arch 설치/운영의 기본기를 익히기 위함.

## 왜 별도 swap partition을 만들지 않았는가?

RAM 24GB 목표이며 초기에는 zram을 사용해 구조를 단순화하기 위함. 하이버네이션 필요성이 생기면 다시 설계한다.

## 왜 DB를 host에 직접 설치하지 않는가?

개발 인프라를 Docker로 격리해 OS를 깨끗하게 유지하고 프로젝트별 재현성을 높이기 위함.

## 왜 자동 `pacman -Syu`를 하지 않는가?

Arch rolling release 특성상 변경 내용을 확인하고 업데이트하는 운영 방식을 선호하기 때문.

## 왜 secrets를 별도 Git repo로 분리하는가?

설정과 비밀정보의 생명주기/권한을 분리하고, SOPS + age로 암호화된 상태에서 Git에 저장하기 위함.

---

# 20. 참고 문서

주요 출발점:

- ArchWiki Main Page
- ArchWiki Installation Guide
- ArchWiki AMDGPU
- ArchWiki KDE
- ArchWiki NetworkManager
- ArchWiki PipeWire
- ArchWiki Bluetooth
- ArchWiki Zsh
- ArchWiki Tmux
- ArchWiki Neovim
- ArchWiki ZRAM
- ArchWiki Fwupd
- ArchWiki Restic
- ArchWiki Dotfiles
- ArchWiki SOPS / age 관련 공식 문서

> 외부 문서의 구체적인 명령이나 패키지명은 시스템 업데이트 시 변경될 수 있으므로, 작업 시 현재 ArchWiki/공식 패키지 정보를 다시 확인한다.

---

# 21. One-page handoff summary

```text
MACHINE
Lenovo IdeaPad Slim 3 14AHP10-83K9000CKR
Ryzen 7 8840HS / Radeon 780M / 14" 1920x1200
8GB onboard + 16GB SO-DIMM planned = 24GB
512GB NVMe + possible second M.2 slot

OS
Arch Linux
UEFI
GPT
systemd-boot
ext4
KDE Plasma
Wayland

HARDWARE STACK
AMDGPU
Mesa
NetworkManager
PipeWire / WirePlumber
BlueZ
power-profiles-daemon

KOREAN
ko_KR.UTF-8 prepared
Fcitx5 + Hangul
Noto CJK
System UI remains English

CLI
Git
OpenSSH
Zsh
Tmux
ripgrep
fd
fzf
Neovim
mise

BROWSER
Firefox

MAINTENANCE
fwupd refresh
fstrim timer
zram
firewalld
system time sync
pacman mirror checked

BACKUP DESIGN
Dotfiles      -> Git
OS definition -> arch-setup Git
Secrets       -> SOPS + age
Actual data   -> restic

NOT YET / NEXT
Backup implementation + restore test
JDK 21
Python + uv
Node + pnpm
Go / Rust
Docker
PostgreSQL + pgvector
Valkey
Ollama
Spring / RAG / MCP / Frontend
```
