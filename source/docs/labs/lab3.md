# Лабораторная работа №3 — Библиотека парсинга аргументов

## Цель
Спроектировать и реализовать библиотеку для парсинга аргументов командной строки без использования контейнеров, строк, шаблонов и классов.

## Задание
- Реализовать библиотеку `argparser`.
- Поддержать следующие типы аргументов:
  - целые числа
  - вещественные числа
  - флаги
  - строки (фиксированной максимальной длины)
- Обеспечить прохождение всех тестов GoogleTest.
- Реализовать динамическое выделение памяти.
- Избежать дублирования кода.

## Код

```cpp
// Основная структура аргумента

enum ValueKind { kValueInt, kValueFloat, kValueString, kValueFlag };

struct ArgSpec {
    char* short_opt;
    char* long_opt;
    void* target;
    ValueKind kind;
    NargsType nargs_mode;
    int appearances;
};

// Структура парсера

struct ArgumentParserStruct {
    char* prog_name;
    ArgSpec* specs;
    int spec_count;
    int spec_capacity;
};

// Регистрация аргумента (динамическое расширение массива)

static void EnsureSpecCapacity(ArgumentParser parser) {
    if (parser->spec_count < parser->spec_capacity) return;

    int new_cap = (parser->spec_capacity == 0 ? 8 : parser->spec_capacity * 2);
    ArgSpec* tmp = new ArgSpec[new_cap];
    std::memcpy(tmp, parser->specs, sizeof(ArgSpec) * parser->spec_count);
    delete[] parser->specs;

    parser->specs = tmp;
    parser->spec_capacity = new_cap;
}

// Основной цикл парсинга

bool Parse(ArgumentParser parser, int argc, const char* argv[]) {
    for (int i = 1; i < argc; ++i) {
        const char* tok = argv[i];

        if (tok[0] == '-') {
            int idx = FindSpecByOption(parser, tok);
            if (idx < 0) return false;

            ArgSpec& spec = parser->specs[idx];

            if (spec.kind == kValueFlag) {
                *static_cast<bool*>(spec.target) = true;
                spec.appearances++;
            }
            // обработка чисел, строк и повторяющихся значений
        }
    }
    return true;
}
```

## Вывод
В ходе работы реализована библиотека для парсинга аргументов командной строки с поддержкой различных типов данных и повторяющихся параметров.