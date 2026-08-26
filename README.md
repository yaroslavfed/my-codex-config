# Глобальная настройка Codex Skills и AGENTS.md

Этот файл — инструкция по восстановлению глобальной конфигурации Codex после переустановки системы, смены компьютера или настройки нового рабочего места.

## 1. Рекомендуемая структура Git-репозитория

Храни конфигурацию в отдельном приватном репозитории:

```text
my-codex-config/
├── AGENTS.md
├── .codex/
│   └── config.example.toml
└── skills/
    ├── adversarial-review/
    ├── branch-report/
    ├── bug-hunt/
    ├── code-review/
    ├── codebase-design/
    ├── differential-review/
    ├── grilling/
    ├── implementation-final-review/
    ├── learning-mode/
    ├── nestjs-review/
    ├── systematic-debugging/
    ├── task-investigation/
    ├── verification-before-completion/
    └── zoom-out/
        └── SKILL.md
```

Этот репозиторий — source of truth. Все изменения в `AGENTS.md` и skills делай именно в нём.

Codex использует глобальную директорию:

- Linux/macOS: `~/.codex`
- Windows: `%USERPROFILE%\.codex`

Важно: **не заменяй всю папку `~/.codex/skills` symlink'ом**, потому что Codex хранит в ней системные skills в `.system`.

Подключай:

- `AGENTS.md` — одним symlink;
- каждый пользовательский skill — отдельным symlink внутри `~/.codex/skills`.

---

# 2. Linux — Ubuntu 24.04

## 2.1. Клонирование

```bash
mkdir -p ~/Documents
cd ~/Documents
git clone <URL_ПРИВАТНОГО_РЕПОЗИТОРИЯ> my-codex-config
```

Проверка:

```bash
ls -la ~/Documents/my-codex-config
find ~/Documents/my-codex-config/skills -maxdepth 2 -name SKILL.md -print
```

## 2.2. Подготовка Codex

```bash
mkdir -p ~/.codex/skills
```

Не удаляй существующую:

```text
~/.codex/skills/.system
```

## 2.3. Подключение AGENTS.md

Если файл уже существует, сохрани резервную копию:

```bash
if [ -e ~/.codex/AGENTS.md ] || [ -L ~/.codex/AGENTS.md ]; then
  mv ~/.codex/AGENTS.md ~/.codex/AGENTS.md.backup
fi
```

Создай ссылку:

```bash
ln -s ~/Documents/my-codex-config/AGENTS.md ~/.codex/AGENTS.md
```

## 2.4. Подключение всех пользовательских skills

```bash
for skill in ~/Documents/my-codex-config/skills/*; do
  [ -d "$skill" ] || continue
  name="$(basename "$skill")"
  rm -rf "$HOME/.codex/skills/$name"
  ln -s "$skill" "$HOME/.codex/skills/$name"
done
```

## 2.5. Проверка

```bash
ls -la ~/.codex/AGENTS.md
ls -la ~/.codex/skills
readlink -f ~/.codex/AGENTS.md
find -L ~/.codex/skills -maxdepth 2 -type f -name SKILL.md -print
cat ~/.codex/AGENTS.md
```

После этого перезапусти Codex / Codex Desktop / агент в IDE.

---

# 3. macOS

На macOS схема практически такая же, как на Linux.

## 3.1. Клонирование

```bash
mkdir -p ~/Documents
cd ~/Documents
git clone <URL_ПРИВАТНОГО_РЕПОЗИТОРИЯ> my-codex-config
```

## 3.2. Подготовка

```bash
mkdir -p ~/.codex/skills
```

## 3.3. Подключение AGENTS.md

```bash
if [ -e ~/.codex/AGENTS.md ] || [ -L ~/.codex/AGENTS.md ]; then
  mv ~/.codex/AGENTS.md ~/.codex/AGENTS.md.backup
fi

ln -s ~/Documents/my-codex-config/AGENTS.md ~/.codex/AGENTS.md
```

## 3.4. Подключение skills

```bash
for skill in ~/Documents/my-codex-config/skills/*; do
  [ -d "$skill" ] || continue
  name="$(basename "$skill")"
  rm -rf "$HOME/.codex/skills/$name"
  ln -s "$skill" "$HOME/.codex/skills/$name"
done
```

## 3.5. Проверка

```bash
ls -la ~/.codex/AGENTS.md
ls -la ~/.codex/skills
find -L ~/.codex/skills -maxdepth 2 -type f -name SKILL.md -print
cat ~/.codex/AGENTS.md
```

После этого перезапусти Codex / IDE-agent.

---

# 4. Windows 11

Рекомендуемый путь для репозитория:

```text
C:\Users\<USER>\Documents\my-codex-config
```

Глобальная директория Codex:

```text
%USERPROFILE%\.codex
```

или в PowerShell:

```text
$HOME\.codex
```

## 4.1. Клонирование

PowerShell:

```powershell
Set-Location "$HOME\Documents"
git clone <URL_ПРИВАТНОГО_РЕПОЗИТОРИЯ> my-codex-config
```

## 4.2. Подготовка Codex

```powershell
New-Item -ItemType Directory -Force -Path "$HOME\.codex\skills" | Out-Null
```

Не удаляй:

```text
%USERPROFILE%\.codex\skills\.system
```

## 4.3. Разрешение symbolic links

На Windows создание symlink без запуска терминала от администратора обычно требует включённого **Developer Mode**:

```text
Settings
→ System
→ For developers
→ Developer Mode
```

Альтернатива — запуск PowerShell/Windows Terminal от имени администратора.

## 4.4. Подключение AGENTS.md

```powershell
$agents = "$HOME\.codex\AGENTS.md"

if (Test-Path $agents) {
    Move-Item $agents "$HOME\.codex\AGENTS.md.backup" -Force
}

New-Item `
  -ItemType SymbolicLink `
  -Path "$HOME\.codex\AGENTS.md" `
  -Target "$HOME\Documents\my-codex-config\AGENTS.md"
```

## 4.5. Подключение всех skills

```powershell
$source = "$HOME\Documents\my-codex-config\skills"
$target = "$HOME\.codex\skills"

Get-ChildItem $source -Directory | ForEach-Object {
    $link = Join-Path $target $_.Name

    if (Test-Path $link) {
        Remove-Item $link -Recurse -Force
    }

    New-Item `
      -ItemType SymbolicLink `
      -Path $link `
      -Target $_.FullName | Out-Null
}
```

## 4.6. Проверка

```powershell
Get-Item "$HOME\.codex\AGENTS.md" | Format-List *
Get-Content "$HOME\.codex\AGENTS.md"
Get-ChildItem "$HOME\.codex\skills" -Force
Get-ChildItem "$HOME\.codex\skills" -Recurse -Filter SKILL.md -Force |
    Select-Object FullName
```

После настройки перезапусти Codex / Codex Desktop / IDE-agent.

---

# 5. Проверка внутри Codex

После перезапуска создай новую сессию и спроси:

```text
Какие skills тебе доступны?
```

В ответе должны присутствовать твои skills, например:

```text
adversarial-review
branch-report
bug-hunt
code-review
codebase-design
differential-review
grilling
implementation-final-review
learning-mode
nestjs-review
systematic-debugging
task-investigation
verification-before-completion
zoom-out
```

Для проверки `AGENTS.md`:

```text
На каком языке ты должен отвечать мне согласно глобальным инструкциям?
```

Если в `AGENTS.md` задан русский язык, Codex должен это подтвердить.

---

# 6. WebStorm MCP

Skills и MCP — разные механизмы:

```text
Skill = как агент должен выполнять задачу
MCP   = какими возможностями IDE агент может пользоваться
```

Для WebStorm MCP отдельно включи:

```text
WebStorm
→ Settings
→ Tools
→ MCP Server
→ Enable MCP Server
```

Для Codex лучше использовать **Clients Auto-Configuration**.

После настройки в глобальном `config.toml` должна появиться секция примерно такого вида:

```toml
[mcp_servers.webstorm]
url = "http://127.0.0.1:<PORT>/stream"
```

Порт лучше не задавать вручную — используй Auto-Configuration WebStorm. Файл `.codex/config.example.toml` в этом репозитории служит только примером и **не должен заменять** локальный `~/.codex/config.toml`: порт MCP локален для конкретной установки IDE и может измениться после переустановки или перенастройки WebStorm.

Расположение файла:

- Linux/macOS: `~/.codex/config.toml`
- Windows: `%USERPROFILE%\.codex\config.toml`

## Проверка Linux/macOS

```bash
grep -n -A 5 -B 2 "mcp_servers.webstorm" ~/.codex/config.toml
```

## Проверка Windows PowerShell

```powershell
Select-String `
  -Path "$HOME\.codex\config.toml" `
  -Pattern "mcp_servers.webstorm" `
  -Context 2,5
```

Функциональный тест:

```text
Используй WebStorm MCP.

1. Покажи доступные Run Configurations.
2. Найди usages указанного класса.
3. Проверь IDE inspections для текущего файла.

Ничего не изменяй.
В конце перечисли использованные WebStorm MCP tools.
```

---

# 7. Обновление конфигурации

Все изменения делай в `my-codex-config`.

После изменения:

```bash
git add .
git commit -m "Update Codex configuration"
git push
```

На другой машине:

```bash
cd ~/Documents/my-codex-config
git pull
```

Windows PowerShell:

```powershell
Set-Location "$HOME\Documents\my-codex-config"
git pull
```

Так как `~/.codex` использует symlink, после `git pull` существующие skills и `AGENTS.md` обновятся автоматически.

Если добавлен **новый** skill, нужно создать symlink для него. Самый простой вариант — повторно выполнить цикл подключения skills для своей ОС.

---

# 8. Быстрое восстановление — Linux/macOS

```bash
mkdir -p ~/Documents
cd ~/Documents
git clone <URL_ПРИВАТНОГО_РЕПОЗИТОРИЯ> my-codex-config

mkdir -p ~/.codex/skills

if [ -e ~/.codex/AGENTS.md ] || [ -L ~/.codex/AGENTS.md ]; then
  mv ~/.codex/AGENTS.md ~/.codex/AGENTS.md.backup
fi

ln -s ~/Documents/my-codex-config/AGENTS.md ~/.codex/AGENTS.md

for skill in ~/Documents/my-codex-config/skills/*; do
  [ -d "$skill" ] || continue
  name="$(basename "$skill")"
  rm -rf "$HOME/.codex/skills/$name"
  ln -s "$skill" "$HOME/.codex/skills/$name"
done

find -L ~/.codex/skills -maxdepth 2 -type f -name SKILL.md -print
```

Перезапусти Codex.

---

# 9. Быстрое восстановление — Windows 11

PowerShell:

```powershell
Set-Location "$HOME\Documents"
git clone <URL_ПРИВАТНОГО_РЕПОЗИТОРИЯ> my-codex-config

New-Item -ItemType Directory -Force -Path "$HOME\.codex\skills" | Out-Null

$agents = "$HOME\.codex\AGENTS.md"

if (Test-Path $agents) {
    Move-Item $agents "$HOME\.codex\AGENTS.md.backup" -Force
}

New-Item `
  -ItemType SymbolicLink `
  -Path "$HOME\.codex\AGENTS.md" `
  -Target "$HOME\Documents\my-codex-config\AGENTS.md"

$source = "$HOME\Documents\my-codex-config\skills"
$target = "$HOME\.codex\skills"

Get-ChildItem $source -Directory | ForEach-Object {
    $link = Join-Path $target $_.Name

    if (Test-Path $link) {
        Remove-Item $link -Recurse -Force
    }

    New-Item `
      -ItemType SymbolicLink `
      -Path $link `
      -Target $_.FullName | Out-Null
}

Get-ChildItem "$HOME\.codex\skills" -Recurse -Filter SKILL.md -Force |
    Select-Object FullName
```

Если создание `SymbolicLink` выдаёт ошибку доступа, включи Windows Developer Mode или запусти PowerShell от имени администратора.

Перезапусти Codex.

---

# 10. Итоговая схема

```text
Private Git repository
        │
        │ git clone / git pull
        ▼
~/Documents/my-codex-config
├── AGENTS.md
├── .codex/config.example.toml
└── skills/*
        │
        │ symbolic links
        ▼
~/.codex
├── AGENTS.md
├── config.toml              ← локальный, создаётся Auto-Configuration WebStorm
└── skills/
    ├── .system/
    ├── adversarial-review
    ├── branch-report
    ├── bug-hunt
    ├── code-review
    ├── codebase-design
    ├── differential-review
    ├── grilling
    ├── implementation-final-review
    ├── learning-mode
    ├── nestjs-review
    ├── systematic-debugging
    ├── task-investigation
    ├── verification-before-completion
    └── zoom-out
        │
        ▼
      Codex
        │
        ├── global instructions
        ├── custom skills
        └── WebStorm MCP
```

Главное правило: **source of truth — Git-репозиторий `my-codex-config`; директория `~/.codex` содержит ссылки и системные данные Codex.**
