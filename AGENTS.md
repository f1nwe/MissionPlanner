# AGENTS.md — локальні нотатки цього клону (f1nwe)

Remotes: `origin` = форк `f1nwe/MissionPlanner`, `upstream` = `ArduPilot/MissionPlanner` (не пушити).
Гілка `master` тут чиста і стежить за `upstream/master`, усі свої правки — на гілці
`fix-restore-size` (цей файл теж). Правила upstream-коду див. `CLAUDE.md`.

## Як це запускається на цій машині

- Збірка: `mission-planner-build` (`~/.local/bin`) → `docker run` образу `missionplanner-build:1`
  (Dockerfile `~/.cache/missionplanner-docker`: `mcr.microsoft.com/dotnet/sdk:10.0-noble` + mono-devel),
  який виконує upstream-овий `Linux/build.sh`. NuGet-кеш у `~/.cache/missionplanner-nuget`.
  Результат: `bin/linux/MissionPlanner-linux-x86_64/`, запуск `Linux/run.sh`. На хості потрібен лише `mono`.
- Запуск: `mission-planner` (`~/.local/bin`) і ярлик `~/.local/share/applications/mission-planner.desktop`.
  За замовчуванням через gamescope: `-W 3840 -H 2400 -w 1920 -h 1200 -F nearest -f`, тобто рендер у
  1920×1200 і цілочисельний 2× апскейл. `mission-planner --no-hidpi` (або дія ярлика) — напряму.
- Smoke-тест upstream: `DISPLAY=:0 ./Linux/run.sh --self-test` → `LINUX_SMOKE_TEST_PASS`.
- Скріншот того, що бачить gamescope: `GAMESCOPE_WAYLAND_DISPLAY=gamescope-0 gamescopectl screenshot /tmp/x.png`
  (номер — з логу `Running compositor on wayland display`).

## Чому gamescope, а не фікс DPI у коді

GNOME 51 масштабує XWayland-клієнтів нативно: X-програми бачать 3840×2400 і `Xft.dpi: 192`.
Mono WinForms DPI ігнорує, у MP `Properties/app.manifest` має `dpiAware=false`, контроли —
`AutoScaleMode.None`, розкладки абсолютні. Нативний HiDPI у коді — велика окрема робота.

## Оновлення з upstream

```sh
git checkout master && git pull upstream master && git push origin master
git checkout fix-restore-size && git rebase master && git push --force-with-lease
mission-planner-build
```

Клон зроблений з `--filter=blob:none` (історія є, блоби тягнуться за потреби).
`.local/` у корені створює збірка в Docker, він у `.git/info/exclude`.

## Журнал

- **2026-10-08** — пакет `ardupilot-mission-planner` замінено на цю збірку. Фікс `MainV2.cs`:
  збережений `MainWidth/MainHeight` обмежується робочою областю екрана, бо після звичайного запуску
  в `~/.local/share/Mission Planner/config.xml` лежало 3848×2303, і всередині gamescope (1920×1200)
  Mono робив вікно більшим за екран — саме через це «HiDPI-варіант не працював». Кандидат на PR в upstream.
- Відомі косметичні проблеми на Linux (не наші): вкладка Quick без підписів, написи HUD накладаються
  при вузькій панелі.
