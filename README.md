# Arch laptop Ansible

Плейбук для чистого Arch Linux: ставит `paru`, `greetd` + ReGreet, `niri`, Noctalia Shell, Ghostty, Firefox и Fnott.

## Быстрый старт

```bash
# из пользователя с sudo
sudo pacman -Syu --needed ansible git python sudo

# или из root-сессии на совсем чистой системе
pacman -Syu --needed ansible git python sudo

ansible-galaxy collection install -r requirements.yml
```

Отредактируйте `inventory/hosts.yml` и задайте своего пользователя:

```yaml
laptop_user: your_user
```

Если такого пользователя ещё нет, плейбук создаст его, но без пароля. Пароль лучше задать заранее (`passwd your_user`) или передать SHA-512 hash через `laptop_user_password_hash`.

Запуск локально из пользователя с sudo:

```bash
ansible-playbook site.yml --ask-become-pass
sudo reboot
```

Если sudo ещё не настроен, запускайте плейбук из root-сессии без `--ask-become-pass`.

## Что делает

- Ставит официальные пакеты Arch: `greetd`, `greetd-regreet`, `cage`, `niri`, `xwayland-satellite`, `ghostty`, `firefox`, `fnott`, PipeWire, порталы, шрифты и утилиты для Wayland/Noctalia.
- Собирает и ставит `paru` из AUR, пишет `/etc/paru.conf`.
- Ставит `noctalia-shell` из AUR через `paru`.
- Настраивает `/etc/greetd/config.toml`: ReGreet запускается в Cage, а после логина можно выбрать сессию `niri`.
- Пишет `~/.config/niri/config.kdl`: автозапуск `qs -c noctalia-shell`, `xwayland-satellite`, clipboard watcher и polkit agent. Терминал по умолчанию — Ghostty.
- Пишет базовые конфиги `~/.config/ghostty/config` и `~/.config/fnott/fnott.ini`.

По умолчанию `greetd` только включается в автозагрузку. Если нужно стартовать его сразу, добавьте в inventory:

```yaml
laptop_start_greetd_now: true
```

## Уведомления

У `niri` своего notification daemon нет. У Noctalia он есть, поэтому `fnott` установлен как fallback, но не автозапускается, чтобы не конфликтовать за D-Bus-имя `org.freedesktop.Notifications`. Если хотите использовать Fnott вместо Noctalia-уведомлений, включите:

```yaml
fnott_enable: true
```

## Важно про sudo

Пользователь добавляется в `wheel`, а для `wheel` создаётся правило sudo с паролем: `%wheel ALL=(ALL:ALL) ALL`.

Для неинтерактивной установки AUR-пакетов дополнительно создаётся правило:

```sudoers
your_user ALL=(root) NOPASSWD: /usr/bin/pacman
```

После первичной установки можно отключить его:

```yaml
laptop_enable_passwordless_pacman_for_aur: false
```

и повторно запустить плейбук.

## Отладка

```bash
journalctl -u greetd -b
regreet --demo
qs -c noctalia-shell
niri --config ~/.config/niri/config.kdl
```

Если greeter не поднялся, переключитесь на TTY: `Ctrl+Alt+F2`.
