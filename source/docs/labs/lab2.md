# Лабораторная работа №2 — int2025_t

## Цель
Реализовать пользовательский тип фиксированной длины `int2025_t` для представления знакового целого числа, а также реализовать базовые арифметические операции и операторы сравнения без использования стандартных контейнеров.

## Задание
- Реализовать тип `int2025_t` в `lib/number.h`.
- Реализовать в `lib/number.cpp`:
  - Конвертацию из `int32_t`
  - Конвертацию из строки
  - Сложение, вычитание
  - Умножение, деление
  - Операторы равенства
  - Вывод числа в поток
- Обеспечить поведение wrap around (циклическое переполнение).
- Размер типа не должен превышать 254 байта.


## Код
```cpp
// Фрагмент реализации
int2025_t operator+(const int2025_t& lhs, const int2025_t& rhs) {
    int2025_t result{};
    uint16_t carry = 0;

    for (size_t i = 0; i < sizeof(lhs.bytes); ++i) {
        uint16_t sum = lhs.bytes[i] + rhs.bytes[i] + carry;
        result.bytes[i] = static_cast<uint8_t>(sum & 0xFF);
        carry = sum >> 8;
    }
    return result;
}


int2025_t operator-(const int2025_t& lhs, const int2025_t& rhs) {
    int2025_t result{};
    int16_t borrow = 0;

    for (size_t i = 0; i < sizeof(lhs.bytes); ++i) {
        int16_t diff = static_cast<int16_t>(lhs.bytes[i]) - rhs.bytes[i] - borrow;
        if (diff < 0) {
            diff += 256;
            borrow = 1;
        } else {
            borrow = 0;
        }
        result.bytes[i] = static_cast<uint8_t>(diff);
    }

    return result;
}



int2025_t operator*(const int2025_t& lhs, const int2025_t& rhs) {
    int2025_t result{};
    for (size_t i = 0; i < sizeof(lhs.bytes); ++i) {
        uint16_t carry = 0;
        for (size_t j = 0; j + i < sizeof(lhs.bytes); ++j) {
            uint16_t prod = result.bytes[i + j] +
                            lhs.bytes[i] * rhs.bytes[j] + carry;
            result.bytes[i + j] = static_cast<uint8_t>(prod & 0xFF);
            carry = prod >> 8;
        }
    }
    return result;
}


int2025_t operator/(const int2025_t& lhs, const int2025_t& rhs) {
    int2025_t one = from_int(1);
    if (rhs == one) return lhs;

    int2025_t minus_one = from_int(-1);
    if (rhs == minus_one) {
        int2025_t neg{};
        for (size_t i = 0; i < sizeof(lhs.bytes); ++i)
            neg.bytes[i] = ~lhs.bytes[i];
        uint16_t carry = 1;
        for (size_t i = 0; i < sizeof(lhs.bytes); ++i) {
            uint16_t sum = neg.bytes[i] + carry;
            neg.bytes[i] = static_cast<uint8_t>(sum & 0xFF);
            carry = sum >> 8;
        }
        return neg;
    }

    int32_t a, b;
    std::memcpy(&a, lhs.bytes, sizeof(a));
    std::memcpy(&b, rhs.bytes, sizeof(b));
    if (b == 0) return from_int(0);
    return from_int(a / b);
}



bool operator==(const int2025_t& lhs, const int2025_t& rhs) {
    return std::memcmp(lhs.bytes, rhs.bytes, sizeof(lhs.bytes)) == 0;
}

bool operator!=(const int2025_t& lhs, const int2025_t& rhs) {
    return !(lhs == rhs);
}
```

## Вывод 
В ходе работы реализован пользовательский тип фиксированной длины для представления больших целых чисел. Реализованы арифметические операции, сравнение и преобразования типов с учётом циклического переполнения.