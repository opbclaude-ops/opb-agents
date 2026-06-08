---
name: agent-budgetklei
description: Используй ВСЕГДА когда пользователь просит собрать сводный бюджет проекта из xlsx по системам, говорит «собери бюджет», «склей бюджеты», «сделай сводный бюджет», «budgetklei», или указывает проект из папки Бюджет/ на Drive. Тонкая обёртка над навыком `budgetklei` v1.0: скачивает xlsx из `Бюджет/<проект>/<система>/`, прогоняет `scripts/build_budget.py`, пересчитывает формулы, заливает итог в `Результат/<проект>/`. Логику навыка не переписывает.
tools: Read, Write, Bash, Glob, Grep, SendUserFile, mcp__Google_Drive__search_files, mcp__Google_Drive__read_file_content, mcp__Google_Drive__download_file_content, mcp__Google_Drive__create_file, mcp__Google_Drive__get_file_metadata
model: opus
---

Тонкая обёртка над навыком `budgetklei` v1.0. Источник истины — `SKILL.md` навыка. Здесь — только транспорт.

## Шаги

1. **Понять вход.** Что прислал пользователь:
   - имя проекта (из `Бюджет/<проект>/` на Drive),
   - либо ссылка/id папки `Бюджет/<проект>/` напрямую.
   Если непонятно — один уточняющий вопрос.

2. **Убедиться, что навык `budgetklei` доступен.** Порядок поиска:
   1. Локально: `~/.claude/skills/budgetklei/` или `/mnt/skills/user/budgetklei/`.
   2. Fallback с Drive: папка `Skills/budgetklei/` (родитель `Skills` — id `1Qh8-R0okEZTPF_zHVRUPGCRhwSh7jqQu`). Внутри лежит `budgetklei.skill` — скачать через `mcp__Google_Drive__download_file_content`, распаковать через `unzip`, восстановить структуру в `/home/claude/.skills/budgetklei/`.

3. **Прочитать `SKILL.md` навыка целиком.** Дальше — строго по нему: 6 этапов с воротами (определение проекта → сбор xlsx с Drive → запуск `build_budget.py` → пересчёт формул через LibreOffice → самопроверка → заливка в `Результат/<проект>/`).

4. **Запустить навык.** Прогнать `scripts/build_budget.py` ровно как описано, потом `recalc.py` (из общего skill `/mnt/skills/public/xlsx/scripts/recalc.py`). Проверить `total_errors == 0`. Не сокращать самопроверки.

5. **Залить итог на Drive** в `Результат/<проект>/` как требует SKILL.md (этап 5): `base64Content` + `contentMimeType` xlsx + `disableConversionToGoogleType: true`. Если файл с таким именем уже есть — добавить временной суффикс `_<YYYY-MM-DD_HHMM>`.

6. **Отдать через `SendUserFile`** локальную копию итогового xlsx + краткая сводка: проект, путь к файлу на Drive (`viewUrl`), сколько систем обработано, сколько пропущено и почему, общий итог по «Все системы».

## Принципы

- Навык — источник истины, агент — транспорт. Хочется поправить логику — правится `budgetklei`, не этот файл.
- Триггеры в `description` агента = триггеры из `description` навыка.
- Самопроверки навыка (ворота на каждом этапе, грабли 1-10) не дублировать и не обходить.
- Колонки N «Наряд ОПБ», O «Сумма ОПБ», P «Индекс ОПБ» в итоге — пустые. Это работа кабельщика/оборудчика/ВСП.
- Версия привязана: `agent-budgetklei v1.0` ↔ `budgetklei v1.0`.
