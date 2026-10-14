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
