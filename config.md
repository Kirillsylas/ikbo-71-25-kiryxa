## Практика 1
# Этап 1
```
import getpass
import socket

def make_prompt():
    username = getpass.getuser()
    hostname = socket.gethostname()
    return f"{username}@{hostname}:~$ "

def parse_command(command_line):
    parts = command_line.split()

    if not parts:
        return None, []

    command = parts[0]
    arguments = parts[1:]
    return command, arguments

def run_command(command, arguments):
    if command == "ls":
        print(f"Команда: {command}")
        print(f"Аргументы: {arguments}")

    elif command == "cd":
        print(f"Команда: {command}")
        print(f"Аргументы: {arguments}")

    elif command == "exit":
        return False

    else:
        print(f"Ошибка: неизвестная команда '{command}'")

    return True

def main():
    print("Минимальный эмулятор оболочки. Введите exit для выхода.")

    while True:
        try:
            command_line = input(make_prompt())
        except (EOFError, KeyboardInterrupt):
            print("\nВыход из эмулятора.")
            break

        command, arguments = parse_command(command_line)

        if command is None:
            continue

        if not run_command(command, arguments):
            print("Выход из эмулятора.")
            break

if __name__ == "__main__":
    main()
```
# этап 2, дополняем 1
```
import argparse
import os

def run_script(script_path):
    if not os.path.isfile(script_path):
        print(f"Ошибка: файл скрипта '{script_path}' не найден")
        return

    print(f"\nЗапуск стартового скрипта: {script_path}")

    try:
        with open(script_path, "r", encoding="utf-8") as script:
            for line_number, line in enumerate(script, start=1):
                # Убираем пробелы и символы переноса строки
                line = line.strip()

                # Пропускаем пустые строки
                if not line:
                    continue

                # Показываем команду, как в обычном терминале
                print(make_prompt() + line)

                command, arguments = parse_command(line)

                if command is None:
                    continue

                try:
                    # Выполняем команду
                    should_continue = run_command(command, arguments)

                    # Команда exit завершает выполнение скрипта
                    if not should_continue:
                        print("Скрипт завершён командой exit.")
                        break

                except Exception as error:
                    # При ошибке переходим к следующей строке
                    print(
                        f"Ошибка в строке {line_number}: {error}"
                    )

    except OSError as error:
        print(f"Ошибка чтения скрипта: {error}")

    print("Выполнение стартового скрипта завершено.\n")


# Главная функция программы
def main():
    # Настраиваем параметры командной строки
    parser = argparse.ArgumentParser(
        description="Эмулятор командной оболочки UNIX"
    )

    parser.add_argument(
        "--vfs",
        help="Путь к физическому расположению VFS",
        default="."
    )

    parser.add_argument(
        "--script",
        help="Путь к стартовому скрипту",
        default=None
    )

    # Получаем параметры, переданные при запуске
    args = parser.parse_args()

    # Показываем все заданные параметры
    print("Параметры запуска:")
    print(f"  Путь к VFS: {args.vfs}")
    print(f"  Стартовый скрипт: {args.script}")

    # Проверяем существование каталога VFS
    if os.path.isdir(args.vfs):
        print("Проверка VFS: каталог найден")
    else:
        print(f"Ошибка: каталог VFS '{args.vfs}' не найден")

    # Если указан стартовый скрипт, запускаем его
    if args.script:
        run_script(args.script)

    # Запускаем интерактивный режим
    print("Интерактивный режим. Введите exit для выхода.")

    while True:
        try:
            # Получаем команду пользователя
            command_line = input(make_prompt())

            # Разбираем команду
            command, arguments = parse_command(command_line)

            # Игнорируем пустой ввод
            if command is None:
                continue

            # Выполняем команду
            should_continue = run_command(command, arguments)

            # Завершаем программу при вводе exit
            if not should_continue:
                print("Выход из эмулятора.")
                break

        except (KeyboardInterrupt, EOFError):
            # Обрабатываем Ctrl+C и Ctrl+D
            print("\nВыход из эмулятора.")
            break


# Запускаем программу
if __name__ == "__main__":
    main()
```
# этап 3
 ```
import shlex
import socket

import shlex
import socket

def load_vfs(csv_path):
    """Загружает VFS из CSV. Если файла нет, создаёт начальный CSV."""

    # Если файл отсутствует, создаём его с начальными данными
    if not os.path.isfile(csv_path):
        print(f"CSV-файл не найден: {csv_path}")
        print("Создаём начальную виртуальную файловую систему...")

        with open(csv_path, "w", encoding="utf-8", newline="") as csv_file:
            writer = csv.writer(csv_file)
            writer.writerow(["path", "type", "content"])
            writer.writerow(["home", "dir", ""])
            writer.writerow(["home/user", "dir", ""])
            writer.writerow(["home/user/notes.txt", "file", "Hello from VFS"])
            writer.writerow(["home/user/docs", "dir", ""])
            writer.writerow([
                "home/user/docs/info.txt",
                "file",
                "Information from nested directory"
            ])
            writer.writerow(["home/user/docs/projects", "dir", ""])
            writer.writerow([
                "home/user/docs/projects/readme.txt",
                "file",
                "This is a project file"
            ])
            writer.writerow(["tmp", "dir", ""])
            writer.writerow(["tmp/test.txt", "file", "Temporary test file"])
            writer.writerow(["etc", "dir", ""])
            writer.writerow(["etc/config.txt", "file", "Example configuration"])

        print("Начальный CSV успешно создан.")

    directories = {"/"}
    files = {}

    # Читаем CSV и загружаем данные только в память
    with open(csv_path, "r", encoding="utf-8-sig", newline="") as csv_file:
        reader = csv.DictReader(csv_file)

        required_columns = {"path", "type", "content"}
        if not required_columns.issubset(set(reader.fieldnames or [])):
            raise ValueError(
                "CSV должен содержать столбцы: path,type,content"
            )

        for row in reader:
            path = row["path"].strip().replace("\\", "/")
            object_type = row["type"].strip().lower()
            content = row.get("content") or ""

            if not path:
                continue

            path = "/" + path.strip("/")
            parts = path.strip("/").split("/")

            # Автоматически создаём родительские каталоги
            for i in range(1, len(parts)):
                parent = "/" + "/".join(parts[:i])
                directories.add(parent)

            if object_type in ("dir", "directory"):
                directories.add(path)

            elif object_type == "file":
                if content.startswith("base64:"):
                    content = base64.b64decode(content[7:])

                files[path] = content

            else:
                raise ValueError(
                    f"Неизвестный тип объекта '{object_type}': {path}"
                )

    # Проверяем конфликты между файлами и каталогами
    for path in files:
        if path in directories:
            raise ValueError(
                f"Путь одновременно является файлом и каталогом: {path}"
            )

    return directories, files

def normalize_path(path, current_dir):
    """Преобразует путь в абсолютный путь внутри VFS."""

    if path.startswith("/"):
        full_path = path
    else:
        full_path = current_dir.rstrip("/") + "/" + path

    parts = []

    for part in full_path.split("/"):
        if part == "" or part == ".":
            continue

        if part == "..":
            if parts:
                parts.pop()
        else:
            parts.append(part)

    return "/" + "/".join(parts)

def make_prompt(current_dir):
    """Формирует приглашение с именем пользователя и компьютера."""

    username = getpass.getuser()
    hostname = socket.gethostname()

    return f"{username}@{hostname}:{current_dir}$ "

def parse_command(command_line):
    """Разделяет строку на команду и аргументы."""

    try:
        parts = shlex.split(command_line)
    except ValueError as error:
        print(f"Ошибка разбора команды: {error}")
        return None, []

    if not parts:
        return None, []

    return parts[0], parts[1:]

def command_ls(arguments, current_dir, directories, files):
    """Показывает содержимое виртуального каталога."""

    # Поддерживаем обычный ls и простой режим ls -l
    long_format = "-l" in arguments
    paths = [arg for arg in arguments if arg != "-l"]

    if len(paths) > 1:
        print("Ошибка: ls принимает только один путь")
        return

    target = normalize_path(
        paths[0] if paths else current_dir,
        current_dir
    )

    if target in files:
        print(target.split("/")[-1])
        return

    if target not in directories:
        print(f"Ошибка: каталог или файл не найден: {target}")
        return

    # Находим только непосредственные элементы каталога
    entries = []

    for directory in directories:
        if directory == "/":
            continue

        parent = directory.rsplit("/", 1)[0] or "/"

        if parent == target:
            entries.append((directory.split("/")[-1], "dir"))

    for file_path in files:
        parent = file_path.rsplit("/", 1)[0] or "/"

        if parent == target:
            entries.append((file_path.split("/")[-1], "file"))

    entries.sort(key=lambda item: item[0])

    if not entries:
        print("(каталог пуст)")
        return

    for name, object_type in entries:
        if long_format:
            print(f"{object_type:5} {name}")
        else:
            print(name)

def command_cd(arguments, current_dir, directories):
    """Переходит в другой виртуальный каталог."""

    if len(arguments) > 1:
        print("Ошибка: cd принимает не более одного пути")
        return current_dir

    # Без аргумента переходим в корень VFS
    target = arguments[0] if arguments else "/"
    target = normalize_path(target, current_dir)

    if target not in directories:
        print(f"Ошибка: каталог не найден: {target}")
        return current_dir

    return target

def run_command(command, arguments, state):
    """Выполняет команду и возвращает False при exit."""

    if command == "ls":
        command_ls(
            arguments,
            state["cwd"],
            state["directories"],
            state["files"]
        )

    elif command == "cd":
        state["cwd"] = command_cd(
            arguments,
            state["cwd"],
            state["directories"]
        )

    elif command == "pwd":
        # Показываем текущий виртуальный каталог
        print(state["cwd"])

    elif command == "cat":
        # Выводим содержимое виртуального файла
        if len(arguments) != 1:
            print("Использование: cat <путь>")
            return True

        path = normalize_path(arguments[0], state["cwd"])

        if path not in state["files"]:
            print(f"Ошибка: файл не найден: {path}")
            return True

        content = state["files"][path]

        if isinstance(content, bytes):
            print("[Двоичный файл: содержимое не отображается как текст]")
        else:
            print(content, end="")
            if not content.endswith("\n"):
                print()

    elif command == "exit":
        return False

    else:
        print(f"Ошибка: неизвестная команда '{command}'")

    return True
```
# этап 4
```
def create_default_vfs(csv_path):
    """Создаёт пример виртуальной файловой системы в CSV."""

    with open(csv_path, "w", encoding="utf-8", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["path", "type", "content"])

        writer.writerow(["home", "dir", ""])
        writer.writerow(["home/user", "dir", ""])
        writer.writerow(["home/user/notes.txt", "file", "Hello from VFS"])
        writer.writerow(["home/user/docs", "dir", ""])
        writer.writerow([
            "home/user/docs/info.txt",
            "file",
            "Information from nested directory"
        ])
        writer.writerow(["home/user/docs/projects", "dir", ""])
        writer.writerow([
            "home/user/docs/projects/readme.txt",
            "file",
            "This is a project file"
        ])
        writer.writerow(["tmp", "dir", ""])
        writer.writerow(["tmp/test.txt", "file", "Temporary test file"])
        writer.writerow(["etc", "dir", ""])
        writer.writerow(["etc/config.txt", "file", "Example configuration"])

    print(f"Создана начальная VFS: {csv_path}")
def command_tree(arguments, state):
    """Выводит дерево каталогов и файлов."""

    if len(arguments) > 1:
        print("Ошибка: использование tree [путь]")
        return

    target = normalize_path(
        arguments[0] if arguments else state["cwd"],
        state["cwd"]
    )

    directories = state["directories"]
    files = state["files"]

    if target in files:
        print(target.split("/")[-1])
        return

    if target not in directories:
        print(f"Ошибка: каталог не найден: {target}")
        return

    # Получаем имя корня дерева.
    root_name = target if target != "/" else "/"
    print(root_name)

    def show_tree(directory, prefix=""):
        """Рекурсивно выводит содержимое каждого каталога."""

        entries = []

        for subdir in directories:
            if subdir == "/":
                continue

            parent = subdir.rsplit("/", 1)[0] or "/"

            if parent == directory:
                entries.append((subdir.split("/")[-1], subdir, "dir"))

        for file_path in files:
            parent = file_path.rsplit("/", 1)[0] or "/"

            if parent == directory:
                entries.append((file_path.split("/")[-1], file_path, "file"))

        entries.sort(key=lambda item: (item[2] != "dir", item[0]))

        for index, (name, path, object_type) in enumerate(entries):
            is_last = index == len(entries) - 1

            if is_last:
                branch = "`-- "
                next_prefix = prefix + "    "
            else:
                branch = "|-- "
                next_prefix = prefix + "|   "

            print(prefix + branch + name)

            if object_type == "dir":
                show_tree(path, next_prefix)

    show_tree(target)

def command_clear():
    """Очищает экран терминала."""

    if os.name == "nt":
        os.system("cls")
    else:
        os.system("clear")
```
# этап 5
```
def command_rmdir(arguments, state):
    """Удаляет пустой каталог только из VFS в памяти."""

    # Проверяем количество аргументов
    if len(arguments) != 1:
        print("Ошибка: использование rmdir <путь>")
        return

    # Получаем абсолютный путь к каталогу
    target = normalize_path(arguments[0], state["cwd"])

    directories = state["directories"]
    files = state["files"]

    # Запрещаем удаление корневого каталога
    if target == "/":
        print("Ошибка: нельзя удалить корневой каталог")
        return

    # Проверяем существование каталога
    if target not in directories:
        print(f"Ошибка: каталог не найден: {target}")
        return

    # Проверяем, что каталог пуст
    for directory in directories:
        if directory != target:
            parent = directory.rsplit("/", 1)[0] or "/"

            if parent == target:
                print("Ошибка: каталог не пуст")
                return

    for file_path in files:
        parent = file_path.rsplit("/", 1)[0] or "/"

        if parent == target:
            print("Ошибка: каталог не пуст")
            return

    # Удаляем каталог только из множества в памяти
    directories.remove(target)

    print(f"Каталог '{target}' успешно удалён")
```
