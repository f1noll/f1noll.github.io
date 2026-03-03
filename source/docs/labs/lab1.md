# Лабораторная работа №1 — Шаблонизатор

## Цель
Разработать консольную утилиту на C++, которая генерирует текстовый документ на основе шаблона с метками `{{ variable_name }}` и файла данных формата `KEY=VALUE`.

## Задание
- Реализовать поддержку аргументов командной строки:
  - `-t / --template` — путь к файлу шаблона (обязательно)
  - `-d / --data` — путь к файлу данных (обязательно)
  - `-o / --output` — путь к выходному файлу (опционально)
- Заменять все метки в шаблоне на соответствующие значения.
- Обрабатывать ошибки и возвращать заданные коды завершения.

## Код
```cpp
#include <iostream>
#include <fstream>
#include <cstring>

const int kMaxKey = 100;
const int kMaxVal = 100;
const int kMaxPairs = 1024;
const int kMaxBuf = 65536;

struct KeyValue {
    char key[kMaxKey];
    char val[kMaxVal];
};

bool IsValidIdentifier(const char* str) {
    for (int i = 0; str[i] != '\0'; ++i) {
        char c = str[i];
        bool is_letter = (c >= 'a' && c <= 'z') || (c >= 'A' && c <= 'Z');
        bool is_digit = (c >= '0' && c <= '9');
        if (!(is_letter || is_digit || c == '_')) {
            return false;
        }
    }
    return true;
}

int LoadData(const char* path, KeyValue* pairs) {
    std::ifstream in(path);
    if (!in.is_open()) return -1;

    char line[kMaxKey + kMaxVal + 10];
    int count = 0;

    while (in.getline(line, sizeof(line))) {
        if (line[0] == '#' || (line[0] == '/' && line[1] == '/')) continue;

        char* eq = std::strchr(line, '=');
        if (!eq) continue;

        *eq = '\0';
        char* key = line;
        char* val = eq + 1;

        while (*key == ' ') key++;
        char* end_key = key + std::strlen(key) - 1;
        while (end_key >= key && *end_key == ' ') *end_key-- = '\0';

        while (*val == ' ') val++;
        char* end_val = val + std::strlen(val) - 1;
        while (end_val >= val && *end_val == ' ') *end_val-- = '\0';

        if (!IsValidIdentifier(key) || !IsValidIdentifier(val)) continue;

        bool found = false;
        for (int i = 0; i < count; ++i) {
            if (std::strcmp(pairs[i].key, key) == 0) {
                std::strncpy(pairs[i].val, val, kMaxVal);
                pairs[i].val[kMaxVal - 1] = '\0';
                found = true;
                break;
            }
        }

        if (!found && count < kMaxPairs) {
            std::strncpy(pairs[count].key, key, kMaxKey);
            pairs[count].key[kMaxKey - 1] = '\0';
            std::strncpy(pairs[count].val, val, kMaxVal);
            pairs[count].val[kMaxVal - 1] = '\0';
            count++;
        }
    }
    return count;
}

int ProcessTemplate(const char* template_path, const KeyValue* pairs, int pair_count, std::ostream& out) {
    std::ifstream in(template_path);
    if (!in.is_open()) return 3;

    char c;
    while (in.get(c)) {
        if (c == '{') {
            char next;
            if (!in.get(next)) return 4;

            if (next == '{') {
                char var[kMaxKey];
                int idx = 0;
                bool closed = false;

                while (in.get(c)) {
                    if (c == '}' && in.peek() == '}') {
                        in.get();
                        var[idx] = '\0';
                        closed = true;
                        break;
                    }
                    if (c != ' ') {
                        if (idx < kMaxKey - 1) var[idx++] = c;
                    }
                }

                if (!closed) return 4;

                bool found = false;
                for (int i = 0; i < pair_count; ++i) {
                    if (std::strcmp(pairs[i].key, var) == 0) {
                        out << pairs[i].val;
                        found = true;
                        break;
                    }
                }

                if (!found) return 1;
            } else {
                out << '{' << next;
            }
        } else {
            out << c;
        }
    }
    return 0;
}

int main(int argc, char* argv[]) {
    const char* template_file = nullptr;
    const char* data_file = nullptr;
    const char* output_file = nullptr;

    for (int i = 1; i < argc; ++i) {
        if (std::strcmp(argv[i], "-t") == 0 && i + 1 < argc) {
            template_file = argv[++i];
        } else if (std::strncmp(argv[i], "--template=", 11) == 0) {
            template_file = argv[i] + 11;
        } else if (std::strcmp(argv[i], "-d") == 0 && i + 1 < argc) {
            data_file = argv[++i];
        } else if (std::strncmp(argv[i], "--data=", 7) == 0) {
            data_file = argv[i] + 7;
        } else if (std::strcmp(argv[i], "-o") == 0 && i + 1 < argc) {
            output_file = argv[++i];
        } else if (std::strncmp(argv[i], "--output=", 9) == 0) {
            output_file = argv[i] + 9;
        }
    }

    if (!template_file || !data_file) return 2;

    KeyValue pairs[kMaxPairs];
    int pair_count = LoadData(data_file, pairs);
    if (pair_count < 0) return 3;

    int result;
    if (output_file) {
        std::ofstream out(output_file);
        if (!out.is_open()) return 3;
        result = ProcessTemplate(template_file, pairs, pair_count, out);
    } else {
        result = ProcessTemplate(template_file, pairs, pair_count, std::cout);
    }

    return result;
}
```

## Вывод
В ходе работы реализована утилита генерации текста по шаблону с поддержкой аргументов командной строки и обработкой ошибок. Закреплены навыки работы с текстовыми файлами и разбором входных данных в C++.

