# Инструкции для Claude — сервер 170-168-91-248.swtest.ru

## Общие правила работы

### Git workflow
- Meaningful commit messages на английском
- Не батчить несвязанные изменения
- Co-Authored-By: Claude Sonnet 4.5 <noreply@anthropic.com>

### Безопасность
- ⛔ НИКОГДА не коммитить файлы с токенами и ключами (.env)
- ✅ Проверять .gitignore перед каждым commit

### Экономия токенов
- ✅ Отвечать по делу, без вводных фраз и пересказа задачи
- ✅ Объединять независимые Tool-вызовы в один параллельный блок
- ✅ Не читать один и тот же файл повторно в рамках одной сессии
- ✅ Предпочитать прямые Glob/Grep вместо Task/subagent для простых поисков
- ⛔ Не запускать субагентов, если задача решается за 1–2 вызова инструментов
- ⛔ Не дублировать поиск: если субагент уже ищет — не искать то же самое параллельно

---

## Работа со скилами (Claude Code Skills)

Репозиторий: `https://github.com/artwist-polyakov/polyakov-claude-skills`
Скилы установлены в `~/.claude/skills/`

| Скил | Что делает | Внешние зависимости |
|------|-----------|---------------------|
| `agent-deck` | Запускает дочерние Claude-сессии через agent-deck CLI | `agent-deck` CLI, tmux |
| `codex-review` | Кросс-агентное review: Claude пишет, Codex (GPT) ревьюит | `codex` CLI |
| `ssh-remote-connection` | SSH к удалённому серверу, Docker, логи | SSH-ключ, `config/.env` |
| `scrapedo-web-scraper` | Скрапинг с обходом Cloudflare/CAPTCHA; авто при 403/429 | `SCRAPER_API_KEY` |
| `docx-contracts` | Заполняет .docx шаблоны (Jinja2 `{{VAR}}`) | Python, `docxtpl` |
| `genome-analizer` | Анализирует VCF-файлы генома, SNP, личный отчёт | VCF файл |
| `yandex-wordstat` | Анализ поискового спроса, до 2000 запросов, CSV | `YANDEX_WORDSTAT_TOKEN` |
| `fal-ai-image` | text-to-image и image-edit через fal.ai, до 4K | `FAL_KEY` |
| `humanizer` | Убирает AI-маркеры из текста (rule of three, em dash, клише) | — |

### Правила применения
- ✅ Для агент-режима создавать кастомный агент `.claude/agents/*.md` с явным `skills:` полем
- ✅ Проверять наличие внешних зависимостей (API ключи, CLI) перед использованием
- ⛔ Встроенные агенты (Explore, Plan, general-purpose) НЕ наследуют скилы
- ⛔ Не указывать скилы, не установленные в `~/.claude/skills/`

### Шаблон кастомного агента `.claude/agents/{name}.md`

```markdown
---
name: agent-name
description: Когда использовать этот агент.
tools: Read, Glob, Grep, Bash
skills:
  - skill-name
---

Инструкции для агента.
```

---

## Инфраструктура

- **ОС:** Ubuntu 24.04, 2 CPU, 4 GB RAM, 40 GB диск
- **Рабочая директория:** `/root`
- **Docker-контейнеры:**
  - `paradox_therapy_bot` — Telegram-бот (Python, `/app/main.py`)
  - `docker-app-node-app-1` — Node.js-приложение (порт 3000)

---

## Fail2ban

Установлен и запущен. Конфиг: `/etc/fail2ban/jail.local`

### Параметры джейла sshd
- Окно наблюдения: 10 минут
- Макс. попыток: 3
- Время бана: 24 часа
- Backend: systemd journald

### Управление

```bash
# Статус сервиса
systemctl status fail2ban

# Перезапуск (после изменения конфига)
systemctl restart fail2ban

# Статус SSH-джейла (заблокированные IP, статистика)
fail2ban-client status sshd

# Список всех джейлов
fail2ban-client status

# Разбанить IP вручную
fail2ban-client set sshd unbanip <IP>

# Забанить IP вручную
fail2ban-client set sshd banip <IP>

# Проверить синтаксис конфига без перезапуска
fail2ban-client -t

# Посмотреть логи fail2ban
journalctl -u fail2ban -n 50 --no-pager

# Включить/отключить джейл
fail2ban-client start sshd
fail2ban-client stop sshd
```

### Изменение конфига

Редактировать только `/etc/fail2ban/jail.local` (не `jail.conf`).
После изменений: `systemctl restart fail2ban`

---

## Диагностика сервера

```bash
# Текущая нагрузка CPU и памяти
top -bn1 | head -15

# Топ процессов по CPU
ps aux --sort=-%cpu | head -15

# Нагрузка на диск
iostat -x 1 3

# Заполненность дисков
df -h

# Системные ошибки за последний час
journalctl --since "1 hour ago" -p err --no-pager | grep -v sshd
```

---

## Docker

```bash
# Статус контейнеров
docker ps -a

# Ресурсы контейнеров
docker stats --no-stream

# Логи контейнера
docker logs <name> --tail 50

# Перезапуск контейнера
docker restart <name>
```
