# Глобальная настройка Codex Skills и AGENTS.md

Этот файл — инструкция по восстановлению глобальной конфигурации Codex после переустановки системы, смены компьютера или настройки нового рабочего места.

## 1. Рекомендуемая структура Git-репозитория

Храни конфигурацию в отдельном приватном репозитории:

```text
my-codex-config/
├── AGENTS.md
├── .codex/
│   └── config.example.toml
├── project-memory/
│   ├── README.md
│   └── projects/
│       ├── Карта сервисов.md
│       ├── api-gateway/
│       │   ├── api-gateway.md
│       │   └── chat-routes.md
│       └── elph-chat-server/
│           ├── elph-chat-server.md
│           └── messages.md
└── skills/
    ├── branch-report/
    ├── code-review/
    ├── codebase-design/
    ├── grilling/
    ├── learning-mode/
    ├── project-memory/
    ├── review-fixes/
    ├── senior-review/
    ├── systematic-debugging/
    ├── task-investigation/
    ├── token-efficiency/
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

Если по целевому пути уже лежит другой файл или ссылка, сохрани его вручную под свободным именем и повтори команду:

```bash
agents_source="$HOME/Documents/my-codex-config/AGENTS.md"
agents_target="$HOME/.codex/AGENTS.md"
if [ -L "$agents_target" ] && [ "$(readlink "$agents_target")" = "$agents_source" ]; then
  :
elif [ -e "$agents_target" ] || [ -L "$agents_target" ]; then
  printf 'Путь занят: %s\n' "$agents_target" >&2
else
  ln -s "$agents_source" "$agents_target"
fi
```

## 2.4. Подключение всех пользовательских skills

```bash
for skill in ~/Documents/my-codex-config/skills/*; do
  [ -d "$skill" ] || continue
  skill_target="$HOME/.codex/skills/$(basename "$skill")"
  if [ -L "$skill_target" ] && [ "$(readlink "$skill_target")" = "$skill" ]; then
    continue
  fi
  if [ -e "$skill_target" ] || [ -L "$skill_target" ]; then
    printf 'Путь занят: %s\n' "$skill_target" >&2
    continue
  fi
  ln -s "$skill" "$skill_target"
done
```

При сообщении о занятом пути проверь его содержимое и перенеси вручную под свободным именем, если хочешь заменить его ссылкой

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
agents_source="$HOME/Documents/my-codex-config/AGENTS.md"
agents_target="$HOME/.codex/AGENTS.md"
if [ -L "$agents_target" ] && [ "$(readlink "$agents_target")" = "$agents_source" ]; then
  :
elif [ -e "$agents_target" ] || [ -L "$agents_target" ]; then
  printf 'Путь занят: %s\n' "$agents_target" >&2
else
  ln -s "$agents_source" "$agents_target"
fi
```

## 3.4. Подключение skills

```bash
for skill in ~/Documents/my-codex-config/skills/*; do
  [ -d "$skill" ] || continue
  skill_target="$HOME/.codex/skills/$(basename "$skill")"
  if [ -L "$skill_target" ] && [ "$(readlink "$skill_target")" = "$skill" ]; then
    continue
  fi
  if [ -e "$skill_target" ] || [ -L "$skill_target" ]; then
    printf 'Путь занят: %s\n' "$skill_target" >&2
    continue
  fi
  ln -s "$skill" "$skill_target"
done
```

При сообщении о занятом пути проверь его содержимое и перенеси вручную под свободным именем, если хочешь заменить его ссылкой

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
$agentsSource = "$HOME\Documents\my-codex-config\AGENTS.md"
$agentsTarget = "$HOME\.codex\AGENTS.md"
$existingAgents = Get-Item -LiteralPath $agentsTarget -Force -ErrorAction SilentlyContinue

if ($null -eq $existingAgents) {
    New-Item -ItemType SymbolicLink -Path $agentsTarget -Target $agentsSource | Out-Null
} elseif ($existingAgents.LinkType -ne 'SymbolicLink' -or $existingAgents.Target -ne $agentsSource) {
    Write-Warning "Путь занят: $agentsTarget"
}
```

При сообщении о занятом пути сохрани существующий файл или ссылку вручную под свободным именем и повтори команду

## 4.5. Подключение всех skills

```powershell
$source = "$HOME\Documents\my-codex-config\skills"
$target = "$HOME\.codex\skills"

Get-ChildItem $source -Directory | ForEach-Object {
    $link = Join-Path $target $_.Name
    $existingLink = Get-Item -LiteralPath $link -Force -ErrorAction SilentlyContinue

    if ($null -eq $existingLink) {
        New-Item -ItemType SymbolicLink -Path $link -Target $_.FullName | Out-Null
    } elseif ($existingLink.LinkType -ne 'SymbolicLink' -or $existingLink.Target -ne $_.FullName) {
        Write-Warning "Путь занят: $link"
    }
}
```

При сообщении о занятом пути проверь содержимое и перенеси его вручную под свободным именем, если хочешь заменить ссылкой

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
branch-report
code-review
codebase-design
grilling
learning-mode
review-fixes
project-memory
senior-review
systematic-debugging
task-investigation
token-efficiency
verification-before-completion
zoom-out
```

Для проверки `AGENTS.md`:

```text
На каком языке ты должен отвечать мне согласно глобальным инструкциям?
```

Если в `AGENTS.md` задан русский язык, Codex должен это подтвердить.

## Как выбирать skills

В репозитории 13 пользовательских skills. Для review нужны только три:

- `code-review` — обычное ревью корректности и качества кода, дублирования, шаблонного кода, применимости GRASP, SOLID и GoF, а также достаточности unit, integration и e2e тестов для бизнес-требований. Проверяет, что тесты действительно выявляют поломку поведения и не фиксируют историю изменений вместо функциональности
- `senior-review` — строгое ревью: включает `code-review`, поднимает доступную инфраструктуру в контейнерах и проверяет фичу сквозными сценариями по бизнес-требованиям
- `review-fixes` — разбирает замечания ревью, проверяет их обоснованность, исправляет подтверждённые проблемы по запросу и готовит ответы

Остальные skills выбирай по задаче: `zoom-out` и `task-investigation` для изучения системы; `grilling` для выяснения важных решений пользователя до реализации; `codebase-design` для сравнения архитектурных вариантов; `systematic-debugging` для расследования дефектов; `learning-mode` для совместной учебной разработки; `verification-before-completion` для проверки результата; `token-efficiency` для экономии контекста; `branch-report` для текста MR и Redmine.

`project-memory` хранит Markdown-заметки по репозиториям в соседней папке `project-memory/projects/`. У каждого репозитория своя папка с короткой обзорной заметкой и тематическими заметками по устойчивым сценариям. Скилл лежит в `skills/project-memory/` и находит память по относительному пути `../../project-memory` от своего реального каталога. Он помогает ответить на вопросы об устройстве сервиса, ручках и связях, а также даёт контекст для ревью, исследования задачи, расследования дефекта и `branch-report`. Агент сначала читает обзор и нужные темы, сверяет существенные сведения с текущим кодом, исправляет устаревший пункт вместо добавления противоречащего и создаёт папку для нового репозитория при первом подтверждённом знании. Если папка памяти отсутствует, основная задача продолжается.

Папку `project-memory/projects/` можно открыть как хранилище Obsidian. [Карта сервисов](project-memory/projects/Карта%20сервисов.md), обзорные заметки `[[имя-репозитория]]` и тематические ссылки `[[имя-репозитория/тема|название]]` показывают связи в графе; раздел `Связи` объясняет их назначение. Существующую ссылку `~/.codex/skills/project-memory`, если она указывает на старый отдельный скилл, замени на `~/Documents/my-codex-config/skills/project-memory` перед перезапуском Codex. Старый репозиторий памяти можно сохранить как архив, но текущие заметки находятся здесь.

`grilling` вызывай явно, когда требования или архитектурные решения ещё открыты и ответ зависит от намерения пользователя, а не от уже существующего кода. Например, нужно решить, какой сервис владеет данными или что делать при частичном отказе. Агент сначала изучает доступные факты, затем задаёт по одному важному вопросу, предлагает варианты и рекомендует решение. Результат — согласованные решения до реализации. Если требования уже определены, этот skill не нужен. `task-investigation` исследует текущую систему, а `codebase-design` сравнивает варианты устройства кода при известных требованиях.

`learning-mode` включается только по прямому запросу на учебный режим или совместную пошаговую разработку. Пользователь пишет существенную реализацию реального приложения, а агент по умолчанию полностью готовит тесты, берёт на себя рутинные правки и ведёт работу небольшими проверяемыми этапами: контракт, тест с ожидаемым падением, реализация, проверка и ревью. Агент опирается на действующий план проекта и даёт готовую реализацию по запросу пользователя.

`branch-report` выдаёт название и описание MR отдельно для каждого затронутого репозитория, а для всей задачи — один текст Redmine без раздела «Как проверить». Skill не создаёт MR и не публикует эти тексты.

Общее правило `AGENTS.md` требует сомневаться в собственных решениях и выводах, искать подтверждения и контрпримеры в требованиях, контрактах, коде и тестах и явно обозначать неопределённость при недостатке доказательств.

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

Если skill удалён из репозитория, его прежняя ссылка в `~/.codex/skills` не исчезнет автоматически. Удаляй только эту ссылку после проверки, что она указывает на удалённый каталог данного репозитория.

---

# 8. Быстрое восстановление — Linux/macOS

```bash
mkdir -p ~/Documents
cd ~/Documents
git clone <URL_ПРИВАТНОГО_РЕПОЗИТОРИЯ> my-codex-config

mkdir -p ~/.codex/skills

agents_source="$HOME/Documents/my-codex-config/AGENTS.md"
agents_target="$HOME/.codex/AGENTS.md"
if [ -L "$agents_target" ] && [ "$(readlink "$agents_target")" = "$agents_source" ]; then
  :
elif [ -e "$agents_target" ] || [ -L "$agents_target" ]; then
  printf 'Путь занят: %s\n' "$agents_target" >&2
else
  ln -s "$agents_source" "$agents_target"
fi

for skill in ~/Documents/my-codex-config/skills/*; do
  [ -d "$skill" ] || continue
  skill_target="$HOME/.codex/skills/$(basename "$skill")"
  if [ -L "$skill_target" ] && [ "$(readlink "$skill_target")" = "$skill" ]; then
    continue
  fi
  if [ -e "$skill_target" ] || [ -L "$skill_target" ]; then
    printf 'Путь занят: %s\n' "$skill_target" >&2
    continue
  fi
  ln -s "$skill" "$skill_target"
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

$agentsSource = "$HOME\Documents\my-codex-config\AGENTS.md"
$agentsTarget = "$HOME\.codex\AGENTS.md"
$existingAgents = Get-Item -LiteralPath $agentsTarget -Force -ErrorAction SilentlyContinue

if ($null -eq $existingAgents) {
    New-Item -ItemType SymbolicLink -Path $agentsTarget -Target $agentsSource | Out-Null
} elseif ($existingAgents.LinkType -ne 'SymbolicLink' -or $existingAgents.Target -ne $agentsSource) {
    Write-Warning "Путь занят: $agentsTarget"
}

$source = "$HOME\Documents\my-codex-config\skills"
$target = "$HOME\.codex\skills"

Get-ChildItem $source -Directory | ForEach-Object {
    $link = Join-Path $target $_.Name
    $existingLink = Get-Item -LiteralPath $link -Force -ErrorAction SilentlyContinue

    if ($null -eq $existingLink) {
        New-Item -ItemType SymbolicLink -Path $link -Target $_.FullName | Out-Null
    } elseif ($existingLink.LinkType -ne 'SymbolicLink' -or $existingLink.Target -ne $_.FullName) {
        Write-Warning "Путь занят: $link"
    }
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
    ├── branch-report
    ├── code-review
    ├── codebase-design
    ├── grilling
    ├── learning-mode
    ├── review-fixes
    ├── senior-review
    ├── systematic-debugging
    ├── task-investigation
    ├── token-efficiency
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
