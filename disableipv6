#!/bin/bash

# Скрипт для автоматического отключения IPv6 на Ubuntu через sysctl

set -e

# Функция для вывода сообщений
log() {
    echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $*"
}

# Проверка запуска от root
if [[ "$EUID" -ne 0 ]]; then
    log "Ошибка: Скрипт должен быть запущен от имени root. Используйте sudo."
    exit 1
fi

log "Начинаем процесс отключения IPv6..."

# Шаг 1: Создание конфигурационного файла sysctl для отключения IPv6
SYSCTL_CONF="/etc/sysctl.d/99-disable-ipv6.conf"

log "Создаём/обновляем файл $SYSCTL_CONF для отключения IPv6..."
cat <<EOL > "$SYSCTL_CONF"
# Отключение IPv6
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOL

# Установка правильных прав доступа
chmod 644 "$SYSCTL_CONF"
chown root:root "$SYSCTL_CONF"

log "Файл $SYSCTL_CONF создан и права доступа установлены."

# Шаг 2: Исправление прав доступа к файлам Netplan
NETPLAN_DIR="/etc/netplan"
if [[ -d "$NETPLAN_DIR" ]]; then
    log "Исправляем права доступа к файлам Netplan в $NETPLAN_DIR..."
    chmod 644 "$NETPLAN_DIR"/*.yaml 2>/dev/null || true
    chown root:root "$NETPLAN_DIR"/*.yaml 2>/dev/null || true
    log "Права доступа к файлам Netplan исправлены."
else
    log "Каталог Netplan ($NETPLAN_DIR) не найден. Пропускаем этот шаг."
fi

# Шаг 3: Применение настроек sysctl
log "Применяем настройки sysctl..."
sysctl --system

log "Настройки sysctl применены."

# Шаг 4: Проверка состояния IPv6
log "Проверяем статус IPv6..."
if ip a | grep -q inet6; then
    log "Предупреждение: IPv6 всё ещё активен. Возможно, требуется дополнительная настройка."
else
    log "IPv6 успешно отключён."
fi

# Шаг 5: Дополнительное отключение IPv6 через GRUB (опционально)
read -p "Хотите также отключить IPv6 через параметры ядра в GRUB? (y/N): " disable_grub
if [[ "$disable_grub" =~ ^[Yy]$ ]]; then
    GRUB_CONF="/etc/default/grub"
    log "Добавляем параметр отключения IPv6 в $GRUB_CONF..."
    
    # Проверка, добавлен ли уже параметр
    if grep -q "ipv6.disable=1" "$GRUB_CONF"; then
        log "Параметр ipv6.disable=1 уже присутствует в $GRUB_CONF."
    else
        sed -i 's/GRUB_CMDLINE_LINUX_DEFAULT="/&ipv6.disable=1 /' "$GRUB_CONF"
        log "Параметр ipv6.disable=1 добавлен в GRUB_CMDLINE_LINUX_DEFAULT."
        
        log "Обновляем конфигурацию GRUB..."
        update-grub
        log "Конфигурация GRUB обновлена."
    fi
    
    log "Перезагрузка системы для применения изменений GRUB..."
    read -p "Перезагрузить сейчас? (y/N): " reboot_now
    if [[ "$reboot_now" =~ ^[Yy]$ ]]; then
        reboot
    else
        log "Не забудьте перезагрузить систему позже, чтобы изменения GRUB вступили в силу."
    fi
fi

# Шаг 6: Блокировка модуля IPv6 (опционально)
read -p "Хотите также заблокировать модуль IPv6 через modprobe? (y/N): " blacklist_ipv6
if [[ "$blacklist_ipv6" =~ ^[Yy]$ ]]; then
    BLACKLIST_CONF="/etc/modprobe.d/disable-ipv6.conf"
    log "Создаём/обновляем файл $BLACKLIST_CONF для блокировки модуля IPv6..."
    echo "blacklist ipv6" > "$BLACKLIST_CONF"
    chmod 644 "$BLACKLIST_CONF"
    chown root:root "$BLACKLIST_CONF"
    log "Файл $BLACKLIST_CONF создан и права доступа установлены."
    
    log "Обновляем initramfs..."
    update-initramfs -u
    log "initramfs обновлён."
    
    log "Перезагрузка системы для применения изменений..."
    read -p "Перезагрузить сейчас? (y/N): " reboot_now_blacklist
    if [[ "$reboot_now_blacklist" =~ ^[Yy]$ ]]; then
        reboot
    else
        log "Не забудьте перезагрузить систему позже, чтобы изменения вступили в силу."
    fi
fi

log "Процесс отключения IPv6 завершён."

exit 0
