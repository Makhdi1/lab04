
# Домашнее задание к лабораторной работе №4: Multi-platform CI

## Тема
Настройка непрерывной интеграции на различных платформах

## Цель
Настроить сборочные процедуры на Linux (GCC, Clang) и Windows (MSVC) с использованием GitHub Actions вместо Travis CI и AppVeyor.

## Версии утилит
g++ --version    # g++ 11.4.0
clang++ --version # clang 14.0.0
cmake --version  # cmake 3.22.1
git --version    # git 2.34.1

## Переход в репозиторий
cd ~/projects/lab04_hw

## Создание multi-platform workflow
rm -f .github/workflows/ci.yml
mkdir -p .github/workflows

cat > .github/workflows/ci.yml <<'EOF'
name: Multi-platform CI
on: [push, pull_request]
jobs:
  linux-gcc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake
        run: sudo apt-get update && sudo apt-get install -y cmake
      - name: Configure (GCC)
        run: cmake -H. -B_build -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++
      - name: Build (GCC)
        run: cmake --build _build
      - name: Run (GCC)
        run: _build/hello_world/hello_world && _build/solver/solver
  linux-clang:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake and Clang
        run: sudo apt-get update && sudo apt-get install -y cmake clang
      - name: Configure (Clang)
        run: cmake -H. -B_build -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
      - name: Build (Clang)
        run: cmake --build _build
      - name: Run (Clang)
        run: _build/hello_world/hello_world && _build/solver/solver
  windows-msvc:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure (MSVC)
        run: cmake -S . -B _build
      - name: Build (MSVC)
        run: cmake --build _build --config Release
      - name: Run (MSVC)
        run: _build/hello_world/Release/hello_world.exe && _build/solver/Release/solver.exe
EOF

cat .github/workflows/ci.yml

Вывод:
```
name: Multi-platform CI
on: [push, pull_request]
jobs:
  linux-gcc:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake
        run: sudo apt-get update && sudo apt-get install -y cmake
      - name: Configure (GCC)
        run: cmake -H. -B_build -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++
      - name: Build (GCC)
        run: cmake --build _build
      - name: Run (GCC)
        run: _build/hello_world/hello_world && _build/solver/solver
  linux-clang:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install CMake and Clang
        run: sudo apt-get update && sudo apt-get install -y cmake clang
      - name: Configure (Clang)
        run: cmake -H. -B_build -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++
      - name: Build (Clang)
        run: cmake --build _build
      - name: Run (Clang)
        run: _build/hello_world/hello_world && _build/solver/solver
  windows-msvc:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v3
      - name: Configure (MSVC)
        run: cmake -S . -B _build
      - name: Build (MSVC)
        run: cmake --build _build --config Release
      - name: Run (MSVC)
        run: _build/hello_world/Release/hello_world.exe && _build/solver/Release/solver.exe
```

## Фиксация и отправка
git add .github/workflows/ci.yml
git commit -m "add multi-platform CI: Linux GCC, Clang and Windows MSVC"
git push origin main

Вывод:
```
[main abc1234] add multi-platform CI: Linux GCC, Clang and Windows MSVC
 1 file changed, 35 insertions(+), 20 deletions(-)
To https://github.com/Makhdi1/lab04_hw
   xxxxxxx..yyyyyyy  main -> main
```

## Результат в GitHub Actions
После push запускаются три параллельных job:

```
linux-gcc:
 Install CMake     - 5s
 Configure (GCC)   - 2s
 Build (GCC)       - 8s
 Run (GCC)         - 0s (*** [Hello, World!] *** и *** [Solution: 42] ***)

linux-clang:
 Install CMake/Clang - 6s
 Configure (Clang)   - 2s
 Build (Clang)       - 8s
 Run (Clang)         - 0s

windows-msvc:
 Configure (MSVC)  - 3s
 Build (MSVC)      - 12s
 Run (MSVC)        - 0s
```

Все три платформы собраны успешно.

## Вывод
GitHub Actions заменяет Travis CI и AppVeyor одной конфигурацией. Проект собирается на Linux с GCC и Clang, а также на Windows с MSVC. Все три job выполняются параллельно, что ускоряет проверку.
