# 🛡️ CYBERPATRIOT — DEBIAN 12 SCRIPT LIBRARY

![Debian 12](https://img.shields.io/badge/OS-Debian%2012-A81D33?style=for-the-badge&logo=debian&logoColor=white)
![Bash](https://img.shields.io/badge/LANGUAGE-Bash-4EAA25?style=for-the-badge&logo=gnubash&logoColor=white)
![Scripts](https://img.shields.io/badge/SCRIPTS-24-2563EB?style=for-the-badge)
![Competition](https://img.shields.io/badge/USE-CyberPatriot-7C3AED?style=for-the-badge)

> **Purpose:** A collection of individual Debian/Linux audit, hardening, and forensics scripts for CyberPatriot preparation.
>
> **Important:** The competition README for the actual VM always takes priority. This library does not guarantee that every listed item is scored in every round. Review findings before removing accounts, packages, services, or files.

---

# 📚 SCRIPT INDEX

| ID | Script | Main purpose | Type |
|:--:|---|---|---|
| 00 | [`preflight.sh`](#-script-00--preflightsh) | Initial inventory, backups, and environment report | 🟢 Audit |
| 01 | [`accounts.sh`](#-script-01--accountssh) | Authorized administrator and standard-user groups | 🟠 Changes |
| 02 | [`unauthorized_users.sh`](#-script-02--unauthorized_userssh) | Find and interactively review unlisted accounts | 🔵 Interactive |
| 03 | [`password_audit.sh`](#-script-03--password_auditsh) | Empty password fields and aging information | 🟢 Audit |
| 04 | [`password_policy.sh`](#-script-04--password_policysh) | Password aging and PAM quality settings | 🟠 Changes |
| 05 | [`ssh_audit.sh`](#-script-05--ssh_auditsh) | SSH configuration and root-login protection | 🟠 Changes |
| 06 | [`firewall.sh`](#-script-06--firewallsh) | UFW rules and SSH allowance | 🟠 Changes |
| 07 | [`bookworm_updates.sh`](#-script-07--bookworm_updatessh) | Debian 12 Bookworm package updates | 🟠 Changes |
| 08 | [`automatic_updates.sh`](#-script-08--automatic_updatessh) | Automatic package-list refresh and downloads | 🟠 Changes |
| 09 | [`chromium.sh`](#-script-09--chromiumsh) | Default browser and popup policy | 🟠 Changes |
| 10 | [`lightdm.sh`](#-script-10--lightdmsh) | LightDM and guest-login setting | 🟠 Changes |
| 11 | [`apache.sh`](#-script-11--apachesh) | Apache service review and optional disable | 🔵 Interactive |
| 12 | [`media_scan.sh`](#-script-12--media_scansh) | Find potential prohibited media | 🟢 Audit |
| 13 | [`software_scan.sh`](#-script-13--software_scansh) | Find potentially prohibited applications | 🟢 Audit |
| 14 | [`service_audit.sh`](#-script-14--service_auditsh) | Running services, ports, timers, and cron | 🟢 Audit |
| 15 | [`permission_audit.sh`](#-script-15--permission_auditsh) | SUID/SGID, writable, and unowned files | 🟢 Audit |
| 16 | [`backdoor_audit.sh`](#-script-16--backdoor_auditsh) | Startup persistence, SSH keys, and suspicious locations | 🟢 Audit |
| 17 | [`forensics_search.sh`](#-script-17--forensics_searchsh) | Desktop questions, hidden files, and clues | 🟣 Forensics |
| 18 | [`file_forensics.sh`](#-script-18--file_forensicssh) | File type, magic bytes, hashes, and metadata | 🟣 Forensics |
| 19 | [`login_forensics.sh`](#-script-19--login_forensicssh) | Login, SSH, sudo, and shell-history evidence | 🟣 Forensics |
| 20 | [`final_audit.sh`](#-script-20--final_auditsh) | Final README configuration verification | 🟢 Audit |
| 21 | [`decode_clue.sh`](#-script-21--decode_cluesh) | Decode Base64, hex, ROT13, and URL text | 🟣 Forensics |
| 22 | [`optional_server_audit.sh`](#-script-22--optional_server_auditsh) | Inspect optional web/database server software | 🟢 Audit |
| 23 | [`package_history.sh`](#-script-23--package_historysh) | Package activity and repository-origin review | 🟢 Audit |

## 🎨 LABEL KEY

![Audit](https://img.shields.io/badge/GREEN-READ--ONLY%20AUDIT-16A34A?style=flat-square)
![Changes](https://img.shields.io/badge/ORANGE-MAKES%20SYSTEM%20CHANGES-F59E0B?style=flat-square)
![Interactive](https://img.shields.io/badge/BLUE-ASKS%20BEFORE%20ACTION-2563EB?style=flat-square)
![Forensics](https://img.shields.io/badge/PURPLE-FORENSICS%20HELPER-7C3AED?style=flat-square)

**GitHub display note:** GitHub does not reliably render arbitrary Markdown text colors or custom CSS. These colored Shields.io badges, emoji, large headings, and horizontal dividers are used to make each section visually distinct.

---

---

# 🚀 SCRIPT 00 — `preflight.sh`

![AUDIT / BACKUP](https://img.shields.io/badge/TYPE-AUDIT%20/%20BACKUP-16A34A?style=for-the-badge)

![READ ONLY](https://img.shields.io/badge/ACCESS-READ--ONLY-16A34A?style=flat-square)

**🎯 Task:** Inspect the Debian version, accounts, groups, repositories, services, ports and important configuration. Saves a protected report and copies important configuration files before changes.

```bash

#!/usr/bin/env bash
set -u
umask 077

[[ $EUID -eq 0 ]] || { echo "Use sudo bash preflight.sh"; exit 1; }
source /etc/os-release

STAMP="$(date -u +%Y%m%d-%H%M%S)"
DIR="/root/CP-PREFLIGHT-$STAMP"
mkdir -p "$DIR"
chmod 700 "$DIR"

REPORT="$DIR/report.txt"
exec > >(tee "$REPORT") 2>&1

echo "=== OPERATING SYSTEM ==="
cat /etc/os-release
uname -a

echo "=== IMPORTANT CONFIG BACKUP ==="
for f in /etc/passwd /etc/shadow /etc/group /etc/gshadow \
         /etc/login.defs /etc/ssh/sshd_config \
         /etc/sudoers /etc/pam.d/common-password \
         /etc/hosts /etc/fstab; do
    [[ -f "$f" ]] && cp -a --parents "$f" "$DIR"
done

echo "=== USERS ==="
getent passwd
echo "=== GROUPS ==="
getent group
echo "=== SUDOERS ==="
visudo -c 2>&1 || true
grep -RHE '^[[:space:]]*[^#].*(ALL|NOPASSWD)' \
    /etc/sudoers /etc/sudoers.d 2>/dev/null || true

echo "=== UID 0 ==="
awk -F: '$3==0 {print $1,$3,$7}' /etc/passwd

echo "=== LISTENING PORTS ==="
ss -tulpn

echo "=== RUNNING SERVICES ==="
systemctl list-units --type=service --state=running --no-pager

echo "=== ENABLED SERVICES ==="
systemctl list-unit-files --type=service --state=enabled --no-pager

echo "=== CRON ==="
ls -la /etc/cron.* /etc/crontab 2>/dev/null || true
systemctl list-timers --all --no-pager

echo "=== APT SOURCES ==="
grep -RhsE '^[[:space:]]*(deb |URIs:|Suites:)' \
    /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null || true

echo "=== FIREWALL ==="
ufw status verbose 2>&1 || true

echo "=== SSH ==="
sshd -t 2>&1 || true
systemctl status ssh --no-pager 2>&1 || true

echo "=== REPORT COMPLETE ==="
echo "$REPORT"

```bash


---

# 🚀 SCRIPT 01 — `accounts.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Ensure the seven authorized administrators have sudo membership, and the thirteen authorized standard users don't have sudo/admin/wheel/lpadmin privileges. It does not delete accounts.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1

ADMINS=(pparker tstark ppotts hhogan nfury mhill pcoulson)
USERS=(bbanner swilson sstrange slang cdanvers okoye kbishop ybelova jfoster nleeds mjwatson jtorres pcarter)

getent group sudo >/dev/null || groupadd sudo

for u in "${ADMINS[@]}"; do
    if id "$u" >/dev/null 2>&1; then
        usermod -aG sudo "$u"
        echo "Admin sudo membership ensured: $u"
    else
        echo "MISSING ADMIN: $u"
    fi
done

for u in "${USERS[@]}"; do
    if ! id "$u" >/dev/null 2>&1; then
        echo "MISSING USER: $u"
        continue
    fi

    for g in sudo admin wheel lpadmin; do
        if id -nG "$u" | tr ' ' '\n' | grep -qx "$g"; then
            gpasswd -d "$u" "$g"
            echo "Removed $u from $g"
        fi
    done
done

echo
echo "=== FINAL SUDO GROUP ==="
getent group sudo
echo
echo "=== SUDOERS FILES ==="
visudo -c || true
grep -RHE '^[[:space:]]*[^#].*(ALL|NOPASSWD|!authenticate)' \
    /etc/sudoers /etc/sudoers.d 2>/dev/null || true

```bash


---

# 🚀 SCRIPT 02 — `unauthorized_users.sh`

![INTERACTIVE REVIEW](https://img.shields.io/badge/TYPE-INTERACTIVE%20REVIEW-2563EB?style=for-the-badge)

**🎯 Task:** Find human accounts missing from the README. You must type `REMOVE` before an account record is deleted. Home directories are kept. It protects your current account and automatic-login account.

```bash

#!/usr/bin/env bash
set -u

[[ $EUID -eq 0 ]] || exit 1

AUTHORIZED=(
pparker tstark ppotts hhogan nfury mhill pcoulson
bbanner swilson sstrange slang cdanvers okoye kbishop
ybelova jfoster nleeds mjwatson jtorres pcarter
)

CURRENT="${SUDO_USER:-}"
AUTO="$(grep -RhsE '^[[:space:]]*autologin-user[[:space:]]*=' \
    /etc/lightdm 2>/dev/null | tail -1 | cut -d= -f2- | xargs 2>/dev/null || true)"

echo "=== UNLISTED INTERACTIVE USERS ==="

while IFS=: read -r name _ uid _ _ home shell; do
    [[ "$uid" =~ ^[0-9]+$ ]] || continue
    (( uid >= 1000 && uid < 65534 )) || continue
    [[ "$name" != nobody ]] || continue
    [[ "$shell" != */nologin && "$shell" != */false ]] || continue

    found=0
    for a in "${AUTHORIZED[@]}"; do
        [[ "$name" == "$a" ]] && found=1
    done
    (( found )) && continue

    echo "UNLISTED: $name | UID=$uid | HOME=$home"

    if [[ "$name" == "$CURRENT" || "$name" == "$AUTO" ]]; then
        echo "Protected current/automatic-login account."
        continue
    fi

    read -rp "Remove ONLY account record for $name? Type REMOVE: " answer
    if [[ "$answer" == REMOVE ]]; then
        userdel "$name"
        echo "Removed account record. Home was retained."
    else
        echo "Kept $name."
    fi
done < /etc/passwd

```bash


---

# 🚀 SCRIPT 03 — `password_audit.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Check empty password fields, account lock state, minimum/maximum age, expiration and current password-aging values. It never prints password hashes.

```bash

#!/usr/bin/env bash
set -u

[[ $EUID -eq 0 ]] || exit 1

echo "=== EMPTY PASSWORD FIELDS ==="
awk -F: '$2 == "" {print "EMPTY: " $1}' /etc/shadow

echo
echo "=== UID 0 ACCOUNTS ==="
awk -F: '$3 == 0 {print $1,$3,$7}' /etc/passwd

echo
echo "=== LOGIN.DEFS ==="
grep -E '^[[:space:]]*PASS_(MAX|MIN)_DAYS|^[[:space:]]*PASS_WARN_AGE' \
    /etc/login.defs

echo
echo "=== AUTHORIZED ACCOUNTS ==="

for u in pparker tstark ppotts hhogan nfury mhill pcoulson \
         bbanner swilson sstrange slang cdanvers okoye kbishop \
         ybelova jfoster nleeds mjwatson jtorres pcarter; do
    id "$u" >/dev/null 2>&1 || continue
    echo "--- $u ---"
    passwd -S "$u"
    chage -l "$u"
done

```bash


---

# 🚀 SCRIPT 04 — `password_policy.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Apply your historical Round 1 policy: minimum age 7, maximum age 90, warning 5. Configure password complexity if Debian's pwquality module is installed. Skips the automatic-login account.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1

BACKUP="/root/CP-PAM-BACKUP-$(date -u +%Y%m%d-%H%M%S)"
mkdir -p "$BACKUP"
chmod 700 "$BACKUP"

cp -a /etc/login.defs "$BACKUP/"
cp -a /etc/pam.d "$BACKUP/pam.d"

AUTO="$(grep -RhsE '^[[:space:]]*autologin-user[[:space:]]*=' \
    /etc/lightdm 2>/dev/null | tail -1 | cut -d= -f2- | xargs 2>/dev/null || true)"

setval() {
    local key="$1" val="$2"
    if grep -qE "^[[:space:]]*$key[[:space:]]+" /etc/login.defs; then
        sed -i -E "s|^[[:space:]]*$key[[:space:]]+.*|$key $val|" /etc/login.defs
    else
        echo "$key $val" >> /etc/login.defs
    fi
}

setval PASS_MAX_DAYS 90
setval PASS_MIN_DAYS 7
setval PASS_WARN_AGE 5

USERS=(pparker tstark ppotts hhogan nfury mhill pcoulson bbanner swilson sstrange slang cdanvers okoye kbishop ybelova jfoster nleeds mjwatson jtorres pcarter)

for u in "${USERS[@]}"; do
    id "$u" >/dev/null 2>&1 || continue
    if [[ "$u" == "$AUTO" ]]; then
        echo "Skipping auto-login account: $u"
        continue
    fi
    chage -m 7 -M 90 -W 5 "$u"
done

if [[ -f /usr/share/pam-configs/pwquality ]] &&
   command -v pam-auth-update >/dev/null 2>&1; then

    CONF=/etc/security/pwquality.conf
    cp -a "$CONF" "$BACKUP/" 2>/dev/null || touch "$CONF"

    setq() {
        local key="$1" val="$2"
        if grep -qE "^[[:space:]]*$key[[:space:]]*=" "$CONF"; then
            sed -i -E "s|^[[:space:]]*$key[[:space:]]*=.*|$key = $val|" "$CONF"
        else
            echo "$key = $val" >> "$CONF"
        fi
    }

    setq minlen 12
    setq difok 3
    setq ucredit -1
    setq lcredit -1
    setq dcredit -1
    setq ocredit -1
    setq maxrepeat 3

    if ! grep -q pam_pwquality.so /etc/pam.d/common-password; then
        pam-auth-update --enable pwquality || {
            echo "PAM update failed. Restore from $BACKUP/pam.d if needed."
            exit 1
        }
    fi
else
    echo "pwquality profile unavailable. Complexity configuration skipped."
fi

echo "Password aging configured."
grep -E 'PASS_(MAX|MIN)_DAYS|PASS_WARN_AGE' /etc/login.defs
grep -n 'pam_pwquality\|pam_pwhistory\|pam_unix' /etc/pam.d/common-password
echo "Backup: $BACKUP"

```bash

This does not guarantee password-history enforcement. Check the active PAM stack and scoring report separately. Don't append old `pam_tally2` or `pam_cracklib` lines from outdated Ubuntu checklists to Debian 12.


---

# 🚀 SCRIPT 05 — `ssh_audit.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Secure SSH, disallow root login, preserve password authentication for authorized remote users and validate configuration before reload. Does not restrict authorized usernames with `AllowUsers`.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1
command -v sshd >/dev/null || { echo "sshd not installed"; exit 1; }

CFG=/etc/ssh/sshd_config
DROP=/etc/ssh/sshd_config.d/00-cp-security.conf
BACKUP="/root/CP-SSH-$(date -u +%Y%m%d-%H%M%S)"

mkdir -p "$BACKUP"
cp -a "$CFG" "$BACKUP/"
mkdir -p /etc/ssh/sshd_config.d
[[ -e "$DROP" ]] && cp -a "$DROP" "$BACKUP/" || true

if ! grep -qE '^[[:space:]]*Include[[:space:]]+/etc/ssh/sshd_config.d/' "$CFG"; then
    sed -i '1i Include /etc/ssh/sshd_config.d/*.conf' "$CFG"
fi

cat > "$DROP" <<'EOF'
PermitRootLogin no
PermitEmptyPasswords no
PasswordAuthentication yes
UsePAM yes
X11Forwarding no
MaxAuthTries 4
IgnoreRhosts yes
HostbasedAuthentication no
LogLevel VERBOSE
EOF

chmod 600 "$DROP"

if sshd -t &&
   sshd -T | grep -qx 'permitrootlogin no' &&
   sshd -T | grep -qx 'passwordauthentication yes'; then

    systemctl enable ssh
    systemctl reload ssh
    echo "SSH validated and reloaded."
else
    echo "SSH validation failed; restoring configuration."
    cp -a "$BACKUP/sshd_config" "$CFG"
    if [[ -f "$BACKUP/00-cp-security.conf" ]]; then
        cp -a "$BACKUP/00-cp-security.conf" "$DROP"
    else
        rm -f "$DROP"
    fi
    systemctl restart ssh || true
    exit 1
fi

sshd -T | grep -E \
'^(permitrootlogin|passwordauthentication|permitemptypasswords|usepam|maxauthtries|x11forwarding) '
systemctl is-active ssh

```bash


---

# 🚀 SCRIPT 06 — `firewall.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Configure UFW, allow the current SSH port first, set default deny incoming and allow outgoing. It does not reset existing firewall rules.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1
command -v ufw >/dev/null || { echo "Install UFW from verified Bookworm sources first."; exit 1; }
command -v sshd >/dev/null || exit 1

echo "Current rules:"
ufw status numbered || true

if [[ -n "${SSH_CONNECTION:-}" ]]; then
    echo "Detected SSH session. Run firewall changes from the VM console."
    exit 1
fi

PORTS="$(sshd -T | awk '$1=="port" {print $2}' | sort -un)"
[[ -n "$PORTS" ]] || PORTS=22

while read -r port; do
    [[ "$port" =~ ^[0-9]+$ ]] || continue
    ufw allow "$port/tcp"
done <<< "$PORTS"

ufw default deny incoming
ufw default allow outgoing
ufw logging low

read -rp "Enable UFW with the displayed SSH allowance? Type ENABLE: " answer
[[ "$answer" == ENABLE ]] || exit 0

ufw --force enable
ufw status verbose

```bash


---

# 🚀 SCRIPT 07 — `bookworm_updates.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Update the current Debian 12 packages only. Refuses to proceed if it detects a moving stable alias or another release. Does not use `dist-upgrade`, `full-upgrade` or package removal.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1
source /etc/os-release

[[ "$ID" == debian && "$VERSION_ID" == 12 &&
   "$VERSION_CODENAME" == bookworm ]] || {
    echo "STOP: Debian 12 Bookworm required."
    exit 1
}

echo "=== APT SOURCES ==="
grep -RhsE '^[[:space:]]*(deb |URIs:|Suites:)' \
    /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null || true

SOURCES="$(grep -RhsE '^[[:space:]]*(deb |URIs:|Suites:)' \
    /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null || true)"

if grep -Eiq '(^|[[:space:]])(stable|oldstable|trixie|forky|testing|unstable|sid)([[:space:]]|$)' <<< "$SOURCES"; then
    echo "STOP: Non-Bookworm or moving release suite found."
    exit 1
fi

grep -Eiq 'bookworm' <<< "$SOURCES" || {
    echo "STOP: Could not confirm Bookworm source."
    exit 1
}

read -rp "Confirm these are trusted official Debian 12 Bookworm sources. Type BOOKWORM: " answer
[[ "$answer" == BOOKWORM ]] || exit 1

apt-get update

echo "=== SIMULATING SAFE UPGRADE ==="
apt-get -s --no-remove upgrade

read -rp "Install available Bookworm upgrades without removals? Type UPDATE: " answer
[[ "$answer" == UPDATE ]] || exit 0

DEBIAN_FRONTEND=noninteractive apt-get --no-remove upgrade -y

echo "=== KEY PACKAGE VERSIONS ==="
dpkg-query -W -f='${binary:Package} ${Version}\n' \
    openssh-server chromium systemd ufw 2>/dev/null || true

echo "No release upgrade was performed."

```bash


---

# 🚀 SCRIPT 08 — `automatic_updates.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Configure automatic package-list refresh and download of available packages. It does not automatically install them.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1

source /etc/os-release
[[ "$ID" == debian && "$VERSION_ID" == 12 ]] || exit 1

mkdir -p /etc/apt/apt.conf.d

cat > /etc/apt/apt.conf.d/99-cp-periodic <<'EOF'
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Download-Upgradeable-Packages "1";
APT::Periodic::Unattended-Upgrade "0";
EOF

systemctl enable --now apt-daily.timer

echo "=== TIMER ==="
systemctl status apt-daily.timer --no-pager
systemctl list-timers apt-daily.timer
echo "Automatic installation remains disabled."

```bash


---

# 🚀 SCRIPT 09 — `chromium.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Make Chromium the default browser and configure managed popup blocking. Does not install a browser from an unverified repository.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1

if ! command -v chromium >/dev/null 2>&1; then
    echo "Chromium is missing. Check Bookworm package availability."
    exit 1
fi

mkdir -p /etc/chromium/policies/managed

cat > /etc/chromium/policies/managed/10-cp.json <<'EOF'
{
  "DefaultPopupsSetting": 2
}
EOF

chmod 644 /etc/chromium/policies/managed/10-cp.json

update-alternatives --install /usr/bin/x-www-browser \
    x-www-browser /usr/bin/chromium 200

update-alternatives --set x-www-browser /usr/bin/chromium

mkdir -p /etc/xdg
touch /etc/xdg/mimeapps.list

if ! grep -q '^\[Default Applications\]' /etc/xdg/mimeapps.list; then
    echo '[Default Applications]' >> /etc/xdg/mimeapps.list
fi

for mime in x-scheme-handler/http x-scheme-handler/https text/html application/xhtml+xml; do
    if grep -q "^$mime=" /etc/xdg/mimeapps.list; then
        sed -i "s|^$mime=.*|$mime=chromium.desktop;|" /etc/xdg/mimeapps.list
    else
        sed -i "/^\[Default Applications\]/a $mime=chromium.desktop;" /etc/xdg/mimeapps.list
    fi
done

chromium --version
echo "Check chrome://policy inside Chromium."

```bash


---

# 🚀 SCRIPT 10 — `lightdm.sh`

![SYSTEM CHANGES](https://img.shields.io/badge/TYPE-SYSTEM%20CHANGES-F59E0B?style=for-the-badge)

**🎯 Task:** Preserve LightDM and disable guest login. Does not alter the primary automatic-login account.

```bash

#!/usr/bin/env bash
set -Eeuo pipefail

[[ $EUID -eq 0 ]] || exit 1

DM="$(cat /etc/X11/default-display-manager 2>/dev/null || true)"

if ! grep -qi lightdm <<< "$DM"; then
    echo "LightDM is not the current display manager. No changes."
    exit 1
fi

mkdir -p /etc/lightdm/lightdm.conf.d

cat > /etc/lightdm/lightdm.conf.d/99-cp-guest.conf <<'EOF'
[Seat:*]
allow-guest=false
EOF

echo "LightDM retained."
echo "Guest login disabled."
echo "Automatic login settings left unchanged."
cat /etc/X11/default-display-manager

```bash


---

# 🚀 SCRIPT 11 — `apache.sh`

![INTERACTIVE REVIEW](https://img.shields.io/badge/TYPE-INTERACTIVE%20REVIEW-2563EB?style=for-the-badge)

**🎯 Task:** Investigate and optionally disable Apache. This is based on your Round 1 scoring notes. The package remains installed.

```bash

#!/usr/bin/env bash
set -u

[[ $EUID -eq 0 ]] || exit 1

if ! systemctl list-unit-files apache2.service --no-legend 2>/dev/null |
    grep -q apache2; then
    echo "Apache service not installed."
    exit 0
fi

systemctl status apache2 --no-pager || true

read -rp "Stop and disable Apache while retaining its package? Type DISABLE: " answer

if [[ "$answer" == DISABLE ]]; then
    systemctl disable --now apache2.service
    echo "Apache service disabled."
else
    echo "No changes made."
fi

```bash


---

# 🚀 SCRIPT 12 — `media_scan.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Find possible non-work-related images, audio and video files. This is an investigation script; it never deletes anything.

```bash

#!/usr/bin/env bash
set -u

[[ $EUID -eq 0 ]] || exit 1

DIRS=(/home /root /tmp /var/tmp /opt /srv /mnt /media /var/www)

EXISTING=()
for d in "${DIRS[@]}"; do
    [[ -d "$d" ]] && EXISTING+=("$d")
done

echo "=== MEDIA CANDIDATES ==="

find "${EXISTING[@]}" -type f \( \
    -iname '*.mp3' -o -iname '*.mp4' -o \
    -iname '*.m4a' -o -iname '*.flac' -o \
    -iname '*.wav' -o -iname '*.ogg' -o \
    -iname '*.avi' -o -iname '*.mkv' -o \
    -iname '*.mov' -o -iname '*.webm' -o \
    -iname '*.jpg' -o -iname '*.jpeg' -o \
    -iname '*.png' -o -iname '*.gif' -o \
    -iname '*.bmp' -o -iname '*.webp' \
    \) -printf '%u:%g | %s bytes | %p\n' 2>/dev/null

echo
echo "Review each file against the scenario before deleting it."

```bash


---

# 🚀 SCRIPT 13 — `software_scan.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Find potential prohibited applications, games, network utilities and hacking tools. It does not uninstall anything.

```bash

#!/usr/bin/env bash
set -u

echo "=== POSSIBLE PROHIBITED PACKAGES ==="

PATTERN='nmap|zenmap|nmapsi4|ophcrack|john|hydra|aircrack|wireshark|tshark|tcpdump|netcat|nikto|metasploit|ettercap|kismet|medusa|truecrack|cryptcat|samba|apache2|nginx|lighttpd|telnet|xinetd|vuze|frostwire|minetest|freeciv'

dpkg-query -W -f='${binary:Package}\t${Version}\n' 2>/dev/null |
    grep -Ei "$PATTERN" || true

echo
echo "=== POSSIBLE EXECUTABLES ==="

for app in nmap zenmap ophcrack john hydra aircrack-ng \
           wireshark tshark tcpdump nc netcat nikto \
           msfconsole ettercap kismet telnet; do
    command -v "$app" 2>/dev/null || true
done

echo
echo "=== INSTALLED GAMES ==="
dpkg-query -W -f='${binary:Package}\n' 2>/dev/null |
    grep -Ei 'game|minetest|freeciv|frozen-bubble|supertux|aisleriot' || true

echo
echo "NO APPLICATIONS WERE REMOVED."

```bash


---

# 🚀 SCRIPT 14 — `service_audit.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Find unexpected running services, enabled services, listening ports, scheduled tasks and network processes. Especially useful for discovering an unexpected web server.

```bash

#!/usr/bin/env bash
set -u

echo "=== RUNNING SERVICES ==="
systemctl list-units --type=service --state=running --no-pager

echo
echo "=== ENABLED SERVICES ==="
systemctl list-unit-files --type=service --state=enabled --no-pager

echo
echo "=== LISTENING PORTS ==="
sudo ss -tulpn

echo
echo "=== SOCKET UNITS ==="
systemctl list-sockets --all --no-pager

echo
echo "=== SYSTEMD TIMERS ==="
systemctl list-timers --all --no-pager

echo
echo "=== PROCESSES ==="
ps auxf

echo
echo "=== USER CRONTABS ==="
while IFS=: read -r user _; do
    echo "--- $user ---"
    crontab -u "$user" -l 2>/dev/null || true
done < /etc/passwd

echo
echo "=== SYSTEM CRON ==="
ls -la /etc/cron* /var/spool/cron 2>/dev/null || true

echo
echo "=== CCS CHECK - NEVER DISABLE ==="
systemctl list-units --all --no-pager | grep -i ccs || true
pgrep -a -fi ccs || true

```bash


---

# 🚀 SCRIPT 15 — `permission_audit.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Search for suspicious permissions, unusual SUID/SGID binaries, world-writable files/directories and files without valid owners. It does not change permissions.

```bash

#!/usr/bin/env bash
set -u

echo "=== SUID/SGID FILES ==="
find / -xdev -type f -perm /6000 \
    -printf '%m %u:%g %p\n' 2>/dev/null

echo
echo "=== WORLD-WRITABLE DIRECTORIES WITHOUT STICKY BIT ==="
find / -xdev -type d -perm -0002 ! -perm -1000 \
    -printf '%m %u:%g %p\n' 2>/dev/null

echo
echo "=== WORLD-WRITABLE FILES ==="
find / -xdev -type f -perm -0002 \
    -printf '%m %u:%g %p\n' 2>/dev/null

echo
echo "=== FILES WITHOUT USER/GROUP ==="
find / -xdev \( -nouser -o -nogroup \) \
    -printf '%u:%g %m %p\n' 2>/dev/null

echo
echo "=== CRITICAL FILE PERMISSIONS ==="
stat -c '%a %U:%G %n' \
    /etc/passwd /etc/shadow /etc/group /etc/gshadow /etc/sudoers \
    2>/dev/null || true

```bash


---

# 🚀 SCRIPT 16 — `backdoor_audit.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

![READ ONLY](https://img.shields.io/badge/ACCESS-READ--ONLY-16A34A?style=flat-square)

**🎯 Task:** Look for common persistence and backdoor locations. This is one of the most useful extra scripts because unexpected programs are not always installed as ordinary packages.

It checks:

- SSH authorized keys.
- Unexpected SSH configuration.
- Systemd units and timers.
- Cron persistence.
- Startup scripts.
- Shell startup configuration.
- Hidden files.
- Recent changes to system locations.
- Listening processes.
- Loaded kernel modules.

```bash

#!/usr/bin/env bash
set -u

[[ $EUID -eq 0 ]] || { echo "Use sudo bash backdoor_audit.sh"; exit 1; }

echo "=== LISTENING PROCESSES ==="
ss -tulpn

echo
echo "=== PROCESS TREE ==="
ps auxf

echo
echo "=== SSH AUTHORIZED KEYS ==="
find /root /home -type f -path '*/.ssh/authorized_keys' \
    -print 2>/dev/null

while IFS= read -r f; do
    [[ -f "$f" ]] || continue
    echo "--- $f ---"
    ssh-keygen -lf "$f" 2>/dev/null || true
done < <(find /root /home -type f -path '*/.ssh/authorized_keys' 2>/dev/null)

echo
echo "=== SSH CONFIGURATION ==="
sshd -t 2>&1 || true
sshd -T 2>/dev/null | grep -E \
'^(permitrootlogin|passwordauthentication|allowusers|denyusers|port) ' || true

echo
echo "=== CUSTOM SYSTEMD UNITS ==="
find /etc/systemd/system /usr/local/lib/systemd \
    -type f \( -name '*.service' -o -name '*.timer' -o -name '*.path' \) \
    -printf '%TY-%Tm-%Td %TH:%TM %u:%g %p\n' 2>/dev/null

echo
echo "=== SYSTEMD TIMERS ==="
systemctl list-timers --all --no-pager

echo
echo "=== CRON CONTENTS ==="
for f in /etc/crontab /etc/cron.d/* /var/spool/cron/crontabs/*; do
    [[ -f "$f" ]] || continue
    echo "--- $f ---"
    sed -n '1,150p' "$f"
done

echo
echo "=== STARTUP FILES ==="
for f in /etc/rc.local /etc/profile /etc/bash.bashrc; do
    [[ -f "$f" ]] || continue
    echo "--- $f ---"
    sed -n '1,180p' "$f"
done

echo
echo "=== PROFILE.D ==="
find /etc/profile.d -maxdepth 1 -type f -print -exec ls -l {} \; 2>/dev/null

echo
echo "=== USER STARTUP FILE INVENTORY ==="
find /home /root -maxdepth 4 -type f \( \
    -name '.bashrc' -o -name '.profile' -o \
    -name '.bash_profile' -o -path '*/.config/autostart/*' \
    \) -printf '%TY-%Tm-%Td %TH:%TM %u:%g %p\n' 2>/dev/null

echo
echo "=== SHELL STARTUP SUSPICIOUS COMMANDS ==="
grep -nEi \
'curl|wget|/dev/tcp|nc -e|bash -i|base64 -d|systemctl enable|crontab' \
/etc/profile /etc/bash.bashrc /etc/profile.d/* \
/root/.bashrc /root/.profile /home/*/.bashrc /home/*/.profile \
2>/dev/null || true

echo
echo "=== RECENT FILE CHANGES: LAST 7 DAYS ==="
find /etc /usr/local /opt /var/spool /home /tmp /var/tmp \
    -xdev -type f -mtime -7 \
    -printf '%TY-%Tm-%Td %TH:%TM %u:%g %p\n' 2>/dev/null | head -300

echo
echo "=== HIDDEN FILES ==="
find /home /root /tmp /opt -maxdepth 5 -type f -name '.*' \
    -printf '%u:%g %m %p\n' 2>/dev/null

echo
echo "=== LOADED KERNEL MODULES ==="
lsmod

echo
echo "=== RECENT SYSTEM LOG WARNINGS ==="
journalctl -p warning -b --no-pager 2>/dev/null | tail -100

echo
echo "AUDIT COMPLETE. Nothing was deleted or modified."

```bash


---

# 🚀 SCRIPT 17 — `forensics_search.sh`

![FORENSICS HELPER](https://img.shields.io/badge/TYPE-FORENSICS%20HELPER-7C3AED?style=for-the-badge)

**🎯 Task:** Find Desktop forensic questions, hidden files, clues, suspicious filenames and recently changed documents.

Your Round 1 PDF specifically described a disguised PDF/executable question, so this also identifies files whose real type doesn't match their extension.&#x20;

round 1.pdf



```bash

#!/usr/bin/env bash
set -u

echo "=== DESKTOPS ==="
for d in /home/*/Desktop /root/Desktop; do
    [[ -d "$d" ]] || continue
    echo "--- $d ---"
    ls -la "$d"
done

echo
echo "=== FORENSIC QUESTION FILES ==="
find /home /root -type f \( \
    -iname '*forensic*' -o \
    -iname '*question*' -o \
    -iname '*answer*' -o \
    -iname '*readme*' \
    \) -print 2>/dev/null

echo
echo "=== POSSIBLE CLUE NAMES ==="
find /home /root /tmp /var/tmp /opt -type f \( \
    -iname '*secret*' -o -iname '*hidden*' -o \
    -iname '*flag*' -o -iname '*clue*' \
    \) -print 2>/dev/null

echo
echo "=== HIDDEN FILES ==="
find /home /root -type f -name '.*' -print 2>/dev/null

echo
echo "=== RECENTLY MODIFIED FILES: LAST 24 HOURS ==="
find /home /root /tmp /var/tmp /opt -type f -mmin -1440 \
    -printf '%TY-%Tm-%Td %TH:%TM %p\n' 2>/dev/null | head -300

echo
echo "=== TEXT FILES ON DESKTOPS ==="
find /home/*/Desktop /root/Desktop -type f \
    \( -iname '*.txt' -o -iname '*.md' -o -iname '*.log' \) \
    -print 2>/dev/null

echo
echo "=== OPTIONAL SEARCH FOR A WORD ==="
read -rp "Enter a keyword to search for (blank to skip): " WORD

if [[ -n "$WORD" ]]; then
    grep -RIlF --binary-files=without-match "$WORD" \
        /home /root /tmp /var/tmp /opt 2>/dev/null | head -200
fi

echo
echo "No evidence was modified."

```bash


---

# 🚀 SCRIPT 18 — `file_forensics.sh`

![FORENSICS HELPER](https://img.shields.io/badge/TYPE-FORENSICS%20HELPER-7C3AED?style=for-the-badge)

**🎯 Task:** Investigate a specific file when a forensic question asks:

- What is its actual file type?
- What are its magic bytes?
- What is its SHA-256, SHA-1 or MD5 hash?
- What text or metadata does it contain?
- When was it modified?

Usage:

```bash
sudo bash file_forensics.sh "/home/pparker/Desktop/suspicious.pdf"
```bash

```bash

#!/usr/bin/env bash
set -u

FILE="${1:-}"

if [[ -z "$FILE" ]]; then
    read -rp "Enter full file path: " FILE
fi

if [[ ! -f "$FILE" ]]; then
    echo "File not found."
    exit 1
fi

echo "=== FILE NAME ==="
printf '%s\n' "$FILE"

echo
echo "=== REAL FILE TYPE ==="
file -k "$FILE"

echo
echo "=== FIRST 32 MAGIC BYTES ==="
if command -v xxd >/dev/null; then
    xxd -l 32 "$FILE"
else
    od -An -tx1 -N32 "$FILE"
fi

echo
echo "=== MIME TYPE ==="
file --mime-type "$FILE"

echo
echo "=== SHA-256 ==="
sha256sum "$FILE"

echo
echo "=== SHA-1 ==="
sha1sum "$FILE"

echo
echo "=== MD5 ==="
md5sum "$FILE"

echo
echo "=== FILE METADATA ==="
stat "$FILE"

echo
echo "=== EXTRACTED TEXT ==="
if command -v strings >/dev/null; then
    strings -n 6 "$FILE" | head -100
fi

echo
echo "=== IMAGE/PDF METADATA IF AVAILABLE ==="
if command -v exiftool >/dev/null; then
    exiftool "$FILE"
fi
if command -v pdfinfo >/dev/null; then
    pdfinfo "$FILE" 2>/dev/null || true
fi

echo
echo "No file contents were changed."

```bash


---

# 🚀 SCRIPT 19 — `login_forensics.sh`

![FORENSICS HELPER](https://img.shields.io/badge/TYPE-FORENSICS%20HELPER-7C3AED?style=for-the-badge)

**🎯 Task:** Investigate who logged in, failed login attempts, SSH activity, sudo usage and potentially suspicious command history.

```bash

#!/usr/bin/env bash
set -u

[[ $EUID -eq 0 ]] || exit 1

echo "=== RECENT SUCCESSFUL LOGINS ==="
last -Fai | head -60

echo
echo "=== FAILED LOGINS ==="
if command -v lastb >/dev/null; then
    lastb -Fai | head -60
else
    echo "lastb unavailable."
fi

echo
echo "=== SSH LOGS: LAST 7 DAYS ==="
journalctl -u ssh --since "7 days ago" --no-pager 2>/dev/null | tail -150
journalctl _COMM=sshd --since "7 days ago" --no-pager 2>/dev/null | tail -150

echo
echo "=== SUDO LOGS ==="
journalctl _COMM=sudo --since "7 days ago" --no-pager 2>/dev/null | tail -150

echo
echo "=== AUTHENTICATION LOG ==="
if [[ -f /var/log/auth.log ]]; then
    grep -Ei 'Accepted|Failed|Invalid user|sudo|session opened|session closed' \
        /var/log/auth.log | tail -150
fi

echo
echo "=== SHELL HISTORY FILES ==="
find /home /root -maxdepth 3 -type f \
    \( -name '.bash_history' -o -name '.zsh_history' \) \
    -printf '%u:%g %p\n' 2>/dev/null

echo
echo "=== SUSPICIOUS HISTORY COMMAND CANDIDATES ==="
grep -nEi \
'wget|curl|/dev/tcp|nc -e|useradd|usermod|crontab|systemctl enable|chmod [467][0-7]{3}|base64 -d' \
/home/*/.bash_history /root/.bash_history \
2>/dev/null | tail -100 || true

echo
echo "=== RECENT PACKAGE INSTALLATION EVENTS ==="
grep -iE ' install | upgrade ' /var/log/dpkg.log 2>/dev/null | tail -100

```bash


---

# 🚀 SCRIPT 20 — `final_audit.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Final verification of your README requirements. It does not make changes.

```bash

#!/usr/bin/env bash
set -u

echo "=== OPERATING SYSTEM ==="
cat /etc/os-release

echo
echo "=== UID 0 ACCOUNTS ==="
awk -F: '$3==0 {print $1,$3}' /etc/passwd

echo
echo "=== SUDO GROUP ==="
getent group sudo

echo
echo "=== AUTHORIZED USERS ==="
for u in pparker tstark ppotts hhogan nfury mhill pcoulson \
         bbanner swilson sstrange slang cdanvers okoye kbishop \
         ybelova jfoster nleeds mjwatson jtorres pcarter; do
    id "$u" 2>/dev/null || echo "MISSING: $u"
done

echo
echo "=== PASSWORD AGING ==="
grep -E 'PASS_(MAX|MIN)_DAYS|PASS_WARN_AGE' /etc/login.defs

echo
echo "=== SSH ==="
sshd -t 2>&1 || true
sshd -T 2>/dev/null | grep -E \
'^(permitrootlogin|passwordauthentication|permitemptypasswords|usepam) ' || true
systemctl is-active ssh 2>/dev/null || true
systemctl is-enabled ssh 2>/dev/null || true

echo
echo "=== UFW ==="
ufw status verbose 2>&1 || true

echo
echo "=== LIGHTDM ==="
cat /etc/X11/default-display-manager 2>/dev/null || true
grep -Rni 'allow-guest' /etc/lightdm 2>/dev/null || true

echo
echo "=== CHROMIUM ==="
chromium --version 2>/dev/null || true
cat /etc/chromium/policies/managed/10-cp.json 2>/dev/null || true

echo
echo "=== APACHE ==="
systemctl is-active apache2 2>/dev/null || true
systemctl is-enabled apache2 2>/dev/null || true

echo
echo "=== CCS: MUST REMAIN ACTIVE ==="
systemctl list-units --all --no-pager | grep -i ccs || true
pgrep -a -fi ccs || true

echo
echo "=== AUTOMATIC UPDATE TIMER ==="
systemctl is-enabled apt-daily.timer 2>/dev/null || true
systemctl is-active apt-daily.timer 2>/dev/null || true

echo
echo "=== END OF AUDIT ==="

```bash


---

# 🚀 SCRIPT 21 — `decode_clue.sh`

![FORENSICS HELPER](https://img.shields.io/badge/TYPE-FORENSICS%20HELPER-7C3AED?style=for-the-badge)

**🎯 Task:** Help with common encoded-message forensic questions. Supports Base64, hexadecimal, ROT13 and URL encoding. It doesn't execute decoded content.

```bash

#!/usr/bin/env bash
set -u

echo "=== FORENSIC MESSAGE DECODER ==="
echo "1. Base64"
echo "2. Hexadecimal"
echo "3. ROT13"
echo "4. URL encoding"

read -rp "Choose encoding: " MODE
read -rp "Paste encoded text: " DATA

case "$MODE" in
    1)
        printf '%s' "$DATA" | base64 -d
        echo
        ;;
    2)
        printf '%s' "$DATA" | tr -d '[:space:]' | xxd -r -p
        echo
        ;;
    3)
        printf '%s' "$DATA" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
        echo
        ;;
    4)
        python3 -c 'import sys,urllib.parse; print(urllib.parse.unquote(sys.argv[1]))' "$DATA"
        ;;
    *)
        echo "Unknown encoding."
        exit 1
        ;;
esac

echo "Inspect the result. Do not execute decoded content."

```bash


---

# 🚀 SCRIPT 22 — `optional_server_audit.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Investigate the Apache, MySQL/MariaDB and PHP configurations from your fourth checklist. Your current README does not require these applications, so this script only investigates what is installed.

```bash

#!/usr/bin/env bash
set -u

echo "=== WEB AND DATABASE PACKAGES ==="

dpkg-query -W -f='${binary:Package} ${Version}\n' 2>/dev/null |
    grep -Ei 'apache2|nginx|mysql|mariadb|php[0-9.]' || true

echo
echo "=== WEB AND DATABASE SERVICES ==="

for svc in apache2 nginx mysql mariadb; do
    echo "--- $svc ---"
    systemctl status "$svc" --no-pager 2>&1 | head -20 || true
done

echo
echo "=== LISTENING PORTS ==="
ss -tulpn | grep -E ':(80|443|3306|9000)([[:space:]]|$)' || true

echo
echo "=== CONFIGURATION FILE INVENTORY ==="

for d in /etc/apache2 /etc/nginx /etc/mysql /etc/php; do
    if [[ -d "$d" ]]; then
        echo "--- $d ---"
        find "$d" -maxdepth 3 -type f -printf '%p\n'
    fi
done

echo
echo "No database or web-server configuration was modified."

```bash


---

# 🚀 SCRIPT 23 — `package_history.sh`

![READ-ONLY AUDIT](https://img.shields.io/badge/TYPE-READ-ONLY%20AUDIT-16A34A?style=for-the-badge)

**🎯 Task:** Find when software was installed or updated, investigate package ownership and check Debian repository origins.

```bash

#!/usr/bin/env bash
set -u

echo "=== RECENT PACKAGE ACTIVITY ==="

if [[ -f /var/log/dpkg.log ]]; then
    grep -Ei ' install | upgrade | remove | purge ' \
        /var/log/dpkg.log | tail -150
fi

echo
echo "=== APT HISTORY ==="

if [[ -f /var/log/apt/history.log ]]; then
    tail -150 /var/log/apt/history.log
fi

echo
echo "=== IMPORTANT PACKAGE VERSIONS ==="

for p in chromium openssh-server systemd ufw sudo bash; do
    echo "--- $p ---"
    apt-cache policy "$p" 2>/dev/null || true
done

echo
echo "=== REPOSITORIES ==="

grep -RhsE '^[[:space:]]*(deb |URIs:|Suites:)' \
    /etc/apt/sources.list /etc/apt/sources.list.d 2>/dev/null || true

echo
echo "=== PACKAGE OWNER EXAMPLE ==="
read -rp "Enter a full executable path to investigate (blank to skip): " PATHNAME

if [[ -n "$PATHNAME" ]]; then
    dpkg -S "$PATHNAME" 2>&1 || true
fi

```bash


---

# 🧭 WHICH SCRIPTS SHOULD YOU ACTUALLY USE?

You don't need to run all 23. Here's my priority recommendation for your Debian competition.

## ⭐ HIGHEST-PRIORITY BACKUP SCRIPTS

Use when the matching task hasn't already been completed by lrt.sh.

00

Initial inspection and backups

01

Fix authorized admin and user roles

03–04

Check and configure password policies

05–06

SSH and UFW

07–08

Updates and automatic refresh

16

Find persistence and backdoors

17–19

Investigate forensics questions

20

Verify the README requirements


---

# 🔎 COMMON FORENSIC QUESTIONS — QUICK REFERENCE

| Question                             | Script/command              |
| ------------------------------------ | --------------------------- |
| What is the actual file type?        | `file_forensics.sh`         |
| What are the first 16 magic bytes?   | `xxd -l 16 FILE`            |
| What is the file hash?               | `sha256sum FILE`            |
| Who last logged in?                  | `last -F`                   |
| Who attempted to log in?             | `lastb -F`                  |
| Which port is suspicious?            | `ss -tulpn`                 |
| What program owns that port?         | `readlink -f /proc/PID/exe` |
| What changed recently?               | `forensics_search.sh`       |
| What scheduled a suspicious program? | `backdoor_audit.sh`         |
| Who has sudo access?                 | `accounts.sh`               |
| What package installed a file?       | `dpkg -S FILE`              |
| What's inside a suspicious file?     | `strings FILE`              |
| Is there a hidden message?           | `decode_clue.sh`            |

These question patterns are drawn from the community forensic guide, which also stresses answering questions in the exact requested format and preserving evidence before making changes.&#x20;

GitHub+1




---

# 📌 HOW TO ORGANIZE THIS IN GITHUB

Make a single GitHub Gist named:

`CyberPatriot-Debian12-Script-Library`

Create one file per script, using the filenames above. Keep the notes and task descriptions in a `README.md`.

Don't put the actual generated passwords, scoring answers, private keys or competition credentials into a public Gist.

A final distinction: These scripts cover a broad set of recurring Linux competition tasks, but no historical checklist can guarantee every point in a new image. Your supplied Round 1 PDF documents specific past scoring discoveries; it does not prove those exact vulnerabilities will recur. Your current Debian README takes priority over every generic checklist.&#x20;

round 1.pdf

GitHub+1



My biggest recommendation is to keep `16_backdoor_audit.sh`, `17_forensics_search.sh`, `18_file_forensics.sh` and `19_login_forensics.sh` handy. Those give you investigation tools that the general hardening script cannot replace.
