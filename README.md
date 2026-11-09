# practical_02_all.py
"""
Практична робота №2: Розробка та застосування ітераторів та генераторів у Python
Виконано студенткою групи КБ-25: Григор'євська
"""

import sys
import timeit
import itertools
from collections import namedtuple, defaultdict


# =====================================================================
# ЧАСТИНА 1. Завдання 1.1: Власний ітератор CyclicRange
# =====================================================================
class CyclicRange:
    """Ітератор, що циклічно генерує числа у заданому діапазоні."""

    def __init__(self, start: int, stop: int, step: int = 1, repeats: int = 1):
        if step == 0:
            raise ValueError("Крок не може дорівнювати нулю.")
        self._start = start
        self._stop = stop
        self._step = step
        self._repeats = repeats
        
        self._current = start
        self._current_cycle = 1

    def __iter__(self):
        return self

    def __next__(self) -> int:
        if self._current_cycle > self._repeats:
            raise StopIteration

        val = self._current
        self._current += self._step

        if (self._step > 0 and self._current >= self._stop) or (self._step < 0 and self._current <= self._stop):
            self._current_cycle += 1
            self._current = self._start

        return val


def task_1_1():
    print("\n--- Завдання 1.1: Власний ітератор CyclicRange ---")
    print("CyclicRange(1, 4, repeats=3):")
    for num in CyclicRange(1, 4, repeats=3):
        print(num, end=" ")
    print()

    print("\nCyclicRange(0, 10, step=3, repeats=2):")
    for num in CyclicRange(0, 10, step=3, repeats=2):
        print(num, end=" ")
    print()

    result = list(CyclicRange(5, 8, repeats=2))
    print(f"\nЯк список: {result}")

    total = sum(CyclicRange(1, 5, repeats=2))
    print(f"Сума CyclicRange(1, 5, repeats=2): {total}")


# =====================================================================
# ЧАСТИНА 1. Завдання 1.2: Ітерабельний контейнер StudentGroup
# =====================================================================
Student = namedtuple("Student", ["name", "grade"])


class StudentGroupIterator:
    """Ітератор для обходу StudentGroup."""

    def __init__(self, students: list[Student]):
        self._students = students
        self._index = 0

    def __iter__(self):
        return self

    def __next__(self) -> Student:
        if self._index >= len(self._students):
            raise StopIteration
        student = self._students[self._index]
        self._index += 1
        return student


class StudentGroup:
    """Ітерабельний контейнер студентів з підтримкою багаторазового обходу."""

    def __init__(self, group_name: str):
        self._group_name = group_name
        self._students: list[Student] = []

    def add_student(self, name: str, grade: int) -> None:
        self._students.append(Student(name, grade))

    def __iter__(self):
        return StudentGroupIterator(self._students)

    def __len__(self) -> int:
        return len(self._students)

    def top_students(self, n: int = 3):
        sorted_students = sorted(self._students, key=lambda s: s.grade, reverse=True)
        return iter(sorted_students[:n])


def task_1_2():
    print("\n--- Завдання 1.2: Ітерабельний контейнер StudentGroup ---")
    group = StudentGroup("КБ-25")
    group.add_student("Олена", 95)
    group.add_student("Іван", 78)
    group.add_student("Марія", 92)
    group.add_student("Петро", 85)
    group.add_student("Анна", 88)

    print("Перший обхід:")
    for student in group:
        print(f"  {student.name}: {student.grade}")

    print("\nДругий обхід:")
    for student in group:
        print(f"  {student.name}: {student.grade}")

    print("\nВсі пари студентів:")
    for s1 in group:
        for s2 in group:
            if s1.name < s2.name:
                print(f"  {s1.name} та {s2.name}")

    print("\nТоп-3 студенти:")
    for student in group.top_students(3):
        print(f"  {student.name}: {student.grade}")


# =====================================================================
# ЧАСТИНА 2. Завдання 2.1: Функції-генератори
# =====================================================================
def arithmetic_progression(start: float, step: float):
    current = start
    while True:
        yield current
        current += step


def geometric_progression(start: float, ratio: float, limit: float = float("inf")):
    current = start
    while current <= limit:
        yield current
        current *= ratio


def fibonacci(n: int | None = None):
    a, b = 0, 1
    count = 0
    while n is None or count < n:
        yield a
        a, b = b, a + b
        count += 1


def collatz_sequence(n: int):
    current = n
    while current != 1:
        yield current
        if current % 2 == 0:
            current //= 2
        else:
            current = 3 * current + 1
    yield 1


def flatten(nested):
    for item in nested:
        if isinstance(item, list):
            yield from flatten(item)
        else:
            yield item


def task_2_1():
    print("\n--- Завдання 2.1: Функції-генератори та yield ---")
    ap = arithmetic_progression(2, 3)
    first_10 = list(itertools.islice(ap, 10))
    print(f"Арифметична прогресія (2, 3): {first_10}")

    gp = list(geometric_progression(1, 2, limit=1000))
    print(f"Геометрична прогресія (1, 2): {gp}")

    fib_15 = list(fibonacci(15))
    print(f"Фібоначчі (15): {fib_15}")

    coll = list(collatz_sequence(27))
    print(f"Коллатц (27): довжина={len(coll)}, макс={max(coll)}")

    nested = [1, [2, 3], [4, [5, 6]], 7, [8, [9, [10]]]]
    flat = list(flatten(nested))
    print(f"Розгортання: {flat}")

    for fib_num in fibonacci(None):
        if fib_num > 10000:
            print(f"Перше число Фібоначчі > 10000: {fib_num}")
            break


# =====================================================================
# ЧАСТИНА 2. Завдання 2.2: Конвеєр генераторів
# =====================================================================
SAMPLE_DATA = [
    "Олена,Математика,95",
    "Іван,Математика,78",
    "",
    "Марія,Фізика,92",
    "Петро,Математика,INVALID",
    "Анна,Фізика,88",
    "Олена,Фізика,90",
    "Іван,Програмування,85",
    "Марія,Програмування,91",
    "Петро,Фізика,76",
    "Анна,Математика,93",
    "Олена,Програмування,97",
    "",
    "Марія,Математика,89",
    "Іван,Фізика,72",
    "Петро,Програмування,80",
    "Анна,Програмування,86",
]


def data_source(records: list[str]):
    for record in records:
        yield record


def filter_empty(records):
    for record in records:
        if record.strip():
            yield record


def parse_records(records):
    for record in records:
        parts = record.split(",")
        if len(parts) == 3:
            student, subject, grade_str = parts
            try:
                grade = int(grade_str)
                yield {"student": student, "subject": subject, "grade": grade}
            except ValueError:
                continue


def filter_passed(records, min_grade: int = 60):
    for record in records:
        if record["grade"] >= min_grade:
            yield record


def format_output(records):
    for record in records:
        yield f"[{record['subject']}] {record['student']}: {record['grade']} балів"


def task_2_2():
    print("\n--- Завдання 2.2: Конвеєр генераторів для обробки даних ---")
    pipeline = format_output(
        filter_passed(
            parse_records(
                filter_empty(
                    data_source(SAMPLE_DATA)
                )
            ),
            min_grade=75,
        )
    )

    print("=== Результати (>= 75 балів) ===")
    for line in pipeline:
        print(f"  {line}")

    print("\n=== Середній бал по предметах ===")
    subject_grades = defaultdict(list)
    for record in parse_records(filter_empty(data_source(SAMPLE_DATA))):
        subject_grades[record["subject"]].append(record["grade"])

    for subject, grades in sorted(subject_grades.items()):
        avg = sum(grades) / len(grades)
        print(f"  {subject}: {avg:.1f} (з {len(grades)} оцінок)")


# =====================================================================
# ЧАСТИНА 3. Завдання 3.1: Порівняння пам'яті та продуктивності
# =====================================================================
def task_3_1():
    print("\n--- Завдання 3.1: Порівняння list comprehension та generator expression ---")
    
    print("\n[1] Порівняння використання пам'яті:")
    sizes = [1_000, 10_000, 100_000, 1_000_000]
    print(f"{'N':>12} | {'list (байт)':>14} | {'gen (байт)':>12} | {'Ratio':>8}")
    print("-" * 52)
    for n in sizes:
        data_list = [x ** 2 for x in range(n)]
        data_gen = (x ** 2 for x in range(n))
        list_size = sys.getsizeof(data_list)
        gen_size = sys.getsizeof(data_gen)
        print(f"{n:>12,} | {list_size:>14,} | {gen_size:>12,} | {list_size / gen_size:>7.0f}×")

    print("\n[2] Порівняння продуктивності (sum):")
    n_test = 500_000
    time_list = timeit.timeit(lambda: sum([x ** 2 for x in range(n_test)]), number=5)
    time_gen = timeit.timeit(lambda: sum(x ** 2 for x in range(n_test)), number=5)
    print(f"  list comprehension: {time_list:.4f} с")
    print(f"  generator expression: {time_gen:.4f} с")

    print("\n[3] Практичні приклади використання generator expression:")
    words = ["Python", "ітератор", "генератор", "yield", "comprehension", "пам'ять", "конвеєр"]
    longest = max(words, key=lambda w: len(w))
    print(f"  Найдовше слово: '{longest}'")
    
    even_squares_sum = sum(x ** 2 for x in range(1, 101) if x % 2 == 0)
    print(f"  Сума квадратів парних від 1 до 100: {even_squares_sum}")


# =====================================================================
# ГОЛОВНЕ МЕНЮ ПРОГРАМИ
# =====================================================================
def main():
    while True:
        print("\n" + "=" * 65)
        print(" ГОЛОВНЕ МЕНЮ — ПРАКТИЧНА РОБОТА №2 (Григор'євська, КБ-25)")
        print("=" * 65)
        print(" 1. Завдання 1.1 (Власний ітератор CyclicRange)")
        print(" 2. Завдання 1.2 (Ітерабельний контейнер StudentGroup)")
        print(" 3. Завдання 2.1 (Функції-генератори та yield)")
        print(" 4. Завдання 2.2 (Конвеєр генераторів для обробки даних)")
        print(" 5. Завдання 3.1 (Порівняння пам'яті та продуктивності)")
        print(" 0. Вихід з програми")
        print("=" * 65)

        choice = input(" Оберіть номер завдання (0-5): ").strip()

        if choice == '1':
            task_1_1()
        elif choice == '2':
            task_1_2()
        elif choice == '3':
            task_2_1()
        elif choice == '4':
            task_2_2()
        elif choice == '5':
            task_3_1()
        elif choice == '0':
            print("\nРоботу програми завершено. До побачення!")
            break
        else:
            print("\nНевірний вибір! Введіть число від 0 до 5.")


if __name__ == "__main__":
    main()
