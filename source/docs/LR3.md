# Лабораторная работа 3  
## CI/CD для статического сайта

---

## Выполненные действия

Создан сайт на MkDocs.  
Настроен CI/CD на двух платформах:
- GitHub через GitHub Actions;
- SourceCraft.

В одном репозитории настроены два remote:
- origin (GitHub)
- sourcecraft (SourceCraft)

При каждом push выполняется автоматическая сборка и публикация сайта.

---

## Организация репозитория

- source/docs — исходники сайта  
- source/mkdocs.yml — конфигурация  
- docs — готовый сайт  
- .github/workflows — GitHub Actions  
- .sourcecraft — CI/CD SourceCraft  

---

## Как выполнить деплой

```bash
python -m mkdocs build -f source/mkdocs.yml
git push origin main
git push sourcecraft main
```

---

## Настройки GitHub
- включен GitHub Pages
- выбран Source: GitHub Actions

---

## Настройки SourceCraft
- добавлен remote sourcecraft
- настроены ci.yaml и sites.yaml