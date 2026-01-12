# Born2beroot - Linux Sistem Yönetimi ve Sanal Makine

## 📚 Proje Açıklaması

**Born2beroot**, VirtualBox kullanarak bir Linux sunucusunun sıfırdan kurulması, yapılandırılması ve yönetilmesi hakkında eğitim projesidir. Bu proje, sistem yönetimi, network konfigürasyonu, güvenlik hardening ve DevOps konseptlerini öğretir.

## 🎯 Proje Hedefleri

1. **Linux Sunucusu Kurulumu**: Debian/CentOS'u sanal makinaya kurmak
2. **Sistem Konfigürasyonu**: Hostname, users, sudo setup
3. **Network Ayarlaması**: Static IP, host-only networking
4. **Güvenlik Sertifikasyonu**: SSH, firewall, password policies
5. **Sistem Monitoring**: Script'ler yazarak sistem bilgisi göstermek

## 🛠️ Temel Kurulum Adımları

### 1. Sanal Makine Oluşturma

**VirtualBox Ayarları:**
- VM Memory: 1024 MB RAM
- Storage: 30 GB disk
- Network: Adapter 1 = Host-only, Adapter 2 = NAT (Bonus)

### 2. İşletim Sistemi Kurulumu

- **Seçim**: Debian 11 (Recommended) veya Rocky Linux/CentOS
- **Kurulum**: Minimal kurulum, GUI olmadan
- **Boot Loader**: GRUB

### 3. Hostname Ayarı

```bash
# Hostname'i değiştir
sudo hostnamectl set-hostname login42

# /etc/hosts'u güncelle
sudo nano /etc/hosts
# 127.0.0.1    login42
# ::1          login42
```

### 4. User ve Grup Yönetimi

```bash
# Root olmayan user oluştur
sudo useradd -m -s /bin/bash username

# Password ayarla
sudo passwd username

# sudo group'a ekle
sudo usermod -aG sudo username

# sudo için password gerektir (no-password yok)
sudo visudo
# username ALL=(ALL:ALL) ALL
```

### 5. SSH Konfigürasyonu

```bash
# SSH server yükle
sudo apt-get install openssh-server

# SSH config'i düzenle
sudo nano /etc/ssh/sshd_config

# Önemli ayarlar:
# Port 4242          # Default 22 yerine custom port
# PermitRootLogin no # Root login'i devre dışı
# PubkeyAuthentication yes

# SSH restart et
sudo systemctl restart ssh
```

### 6. UFW Firewall Kurulumu

```bash
# UFW yükle
sudo apt-get install ufw

# UFW'ı aktif et
sudo ufw enable

# SSH port'unu (4242) aç
sudo ufw allow 4242

# Status kontrol et
sudo ufw status
```

### 7. Password Policy

```bash
# libpam-pwquality yükle
sudo apt-get install libpam-pwquality

# /etc/pam.d/common-password düzenle:
# password requisite pam_pwquality.so minlen=10 ucredit=-1 dcredit=-1 \
#   maxrepeat=3 reject_username difok=7 enforce_for_root

# /etc/login.defs düzenle:
# PASS_MAX_DAYS   30
# PASS_MIN_DAYS   2
# PASS_WARN_AGE   7
```

### 8. Sudo Logging

```bash
# /etc/sudoers düzenle
sudo visudo

# Log ayarları ekle:
# Defaults logfile="/var/log/sudo/sudo.log"
# Defaults use_pty
```

## 📖 System Information Script (Bonus)

```bash
#!/bin/bash

# Saat güncellemesi
uptime=$(uptime -p | sed 's/up //')
lastboot=$(who -b | awk '{print $3, $4}')

# CPU/Memory
architecture=$(uname -a)
cpus=$(nproc)
cpu_load=$(head -1 /proc/loadavg | awk '{print $1}')
memory_usage=$(free | grep Mem | awk '{print $3 "/" $2}')
memory_percent=$(free | grep Mem | awk '{printf("%.2f%%", $3 / $2 * 100)}')

# Disk
disk_usage=$(df -BG / | awk 'NR==2 {print $3 "/" $2}')
disk_percent=$(df / | awk 'NR==2 {printf("%.0f%%", $3 / $2 * 100)}')

# CPU Active
cpu_active=$(grep "cpu " /proc/stat | awk '{print int(($2+$4) / ($2+$4+$5) * 100)}')

# Network
ipv4=$(hostname -I | awk '{print $1}')
mac=$(ip link show | grep "ether" | awk '{print $2}')
tcp=$(netstat -an 2>/dev/null | grep ESTABLISHED | wc -l)

# Print formatted
echo "================================"
echo "# SYSTEM INFORMATION #"
echo "================================"
echo "Architecture: $architecture"
echo "CPU Physical: $cpus"
echo "CPU Load: $cpu_load"
echo "CPU Active: $cpu_active%"
echo "Memory Usage: $memory_usage ($memory_percent)"
echo "Disk Usage: $disk_usage ($disk_percent)"
echo "Last Boot: $lastboot"
echo "Uptime: $uptime"
echo "IPv4: $ipv4"
echo "MAC: $mac"
echo "TCP Established: $tcp"
echo "================================"
```

**CRON ile otomatik çalıştırma:**

```bash
# /etc/cron.d/welcome dosyası oluştur
*/10 * * * * root /usr/local/bin/monitoring.sh

# Script'i çalıştırılabilir yap
sudo chmod 755 /usr/local/bin/monitoring.sh
```

## 📚 Öğrenme Çıktıları

✅ Linux sistem kurulumu ve yönetimi öğrenildi  
✅ SSH ve firewall konfigürasyonu mastered  
✅ User ve grup yönetimi yapıldı  
✅ Password policies uygulandı  
✅ System monitoring script'i yazıldı  
✅ Network konfigürasyonu anlaşıldı  
✅ DevOps temelleri öğrenildi  

## 🔧 Kontrol Noktaları

### Sistem Kontrolleri
```bash
# Hostname kontrol et
hostnamectl

# User ve group kontrol et
cat /etc/passwd | grep login42
getent group sudo

# SSH port kontrol et
sudo systemctl status ssh
sudo netstat -tlnp | grep ssh

# Firewall kontrol et
sudo ufw status

# Password policy kontrol et
cat /etc/pam.d/common-password
cat /etc/login.defs | grep PASS
```

## 📝 Önemli Ayarlar

### /etc/sshd_config
```
Port 4242
PermitRootLogin no
PasswordAuthentication yes
PubkeyAuthentication yes
MaxAuthTries 3
MaxSessions 2
```

### /etc/login.defs
```
PASS_MAX_DAYS   30
PASS_MIN_DAYS   2
PASS_WARN_AGE   7
USERGROUPS_ENAB yes
```

### UFW Rules
```bash
sudo ufw allow 4242/tcp
sudo ufw allow 80/tcp (bonus)
sudo ufw allow 443/tcp (bonus)
```

## 🎯 Bonus Özellikler

1. **WordPress + Lighttpd + MariaDB**: Web sunucusu kurulumu
2. **DHCP/DNS**: Network servisleri
3. **File Transfer Protocols**: FTP, SFTP
4. **Backup System**: Automated backups

## 💡 Güvenlik Best Practices

```bash
# Sudo'yu stricter yapma
sudo visudo
# Defaults use_pty
# Defaults require_all

# SSH key-based authentication
ssh-keygen -t rsa -b 4096
# Public key'i sunucuya kopyala

# Fail2ban (brute force protection)
sudo apt-get install fail2ban

# System updates
sudo apt-get update && sudo apt-get upgrade
sudo apt-get autoremove
```

## 🚀 Sık Kullanılan Komutlar

```bash
# User management
sudo useradd -m -s /bin/bash newuser
sudo usermod -aG sudousers newuser
sudo passwd newuser
sudo userdel -r olduser

# Group management
sudo groupadd newgroup
sudo usermod -aG newgroup user
sudo delgroup groupname

# File permissions
chmod 600 sensitive_file       # rw-------
chmod 644 regular_file         # rw-r--r--
chmod 755 executable_file      # rwxr-xr-x

# Systemd services
sudo systemctl start service_name
sudo systemctl enable service_name
sudo systemctl status service_name
```

## 📚 Norm Standartları

- Standart Debian/CentOS kurulumu
- Minimal GUI yok
- Performans ve güvenlik öncelik
- Documentation gereklidir

## 💡 Key Learning Points

1. **Linux System Administration**: Kurulum, konfigürasyon, yönetim
2. **Security Hardening**: Firewall, SSH, password policies
3. **User Management**: /etc/passwd, /etc/shadow, sudoers
4. **Network Configuration**: IP, routing, DNS
5. **Scripting**: Bash script'ler yazma
6. **System Monitoring**: Resource tracking
7. **DevOps Fundamentals**: Server setup ve maintenance

Bu proje, sistem yöneticiliğinin temel becerilerini sağlam hale getirir.
