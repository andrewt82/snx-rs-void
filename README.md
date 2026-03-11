
# Установка и настройка `snx-rs` на Void Linux.

В этой инструкции `snx-rs` устанавливается из исходников, настраивается конфигурационный файл и создаётся удобный способ запуска VPN через `tmux` и алиас `vpn`.

Такой подход удобен тем, что:
- клиент собирается в актуальной версии,
- запуск упрощён до одной команды,
- VPN-сессия не теряется при закрытии  терминала,
- повторное подключение к уже запущенной сессии становится простым.

# 1. Обновление системы и установка зависимостей

```bash
sudo xbps-install -Syu && sudo xbps-install -y git gcc pkg-config openssl-devel sqlite-devel rustup tmux
```

---

# 2. Установка Rust

```bash
rustup-init -y --default-toolchain stable
```

Добавляем окружение Rust:

```bash
echo '[ -f "$HOME/.cargo/env" ] && . "$HOME/.cargo/env"' >> ~/.bash_profile
source ~/.bash_profile
```

Проверка установки:

```bash
rustup --version
rustc --version
cargo --version
```

---

# 3. Сборка `snx-rs`

```bash
git clone https://github.com/ancwrd1/snx-rs.git
cd snx-rs

cargo build --release -p snx-rs -p snxctl
```

---

# 4. Установка бинарников

```bash
sudo install -Dm755 target/release/snx-rs /usr/local/bin/snx-rs
sudo install -Dm755 target/release/snxctl /usr/local/bin/snxctl
```

Проверка:

```bash
snx-rs --version
```

---

# 5. Проверка доступных login types

```bash
snx-rs -m info -s ra.site.ru
```

---

# 6. Создание конфигурации

```bash
mkdir -p ~/.config/snx-rs
nano ~/.config/snx-rs/snx-rs.conf
```

Пример конфигурации:

```ini
server-name=ra.site.ru
login-type=vpn_OTP
user-name=user.name
default-route=false
tunnel-type=ipsec
transport-type=auto
no-keychain=true
```

---

# 7. Скрипт подключения

```bash
nano ~/.config/snx-rs/connect
```

```sh
#!/bin/sh

SESSION=vpn

tmux has-session -t "$SESSION" 2>/dev/null
if [ $? -eq 0 ]; then
    exec tmux attach -t "$SESSION"
fi

exec tmux new-session -s "$SESSION" \
  "printf '========== VPN CONNECT %s ==========\n\n' \"\$(date '+%Y-%m-%d %H:%M:%S')\"; exec sudo snx-rs -m standalone -c $HOME/.config/snx-rs/snx-rs.conf"
```

Сделать скрипт исполняемым:

```bash
chmod +x ~/.config/snx-rs/connect
```

---

# 8. Алиас для запуска

Добавить в `~/.bashrc`:

```bash
alias vpn='sh $HOME/.config/snx-rs/connect'
```

Применить изменения:

```bash
source ~/.bashrc
```

---

# 9. Подключение к VPN

```bash
vpn
```

Если сессия уже существует — произойдёт подключение к ней через `tmux`.

---
