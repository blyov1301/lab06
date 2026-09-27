# Лабораторная работа №6: Изучение средств пакетирования на примере CPack

**Студент:** Литошенко Григорий

**GitHub Username:** blyov1301

Данная лабораторная работа посвещена изучению средств пакетирования на примере CPack; настройка CI так, чтобы при появлении тега автоматически собирались пакеты (.deb, .rpm, .tar.gz, .msi, .dmg) и прикреплялись к GitHub Release.

## Сборка проекта и локальная генерация пакетов

*cmake -H. -B_build*

Вывод:

```bash
-- Configuring done (0.0s)
-- Generating done (0.0s)
-- Build files have been written to: /home/vboxuser/workspace/lab06/_build
```
*cmake --build _build*

Вывод:

```bash
[ 18%] Built target formatter
[ 36%] Built target formatter_ex
[ 54%] Built target solver_lib
[ 72%] Built target solver
[100%] Built target banking
```
*cd _build*

После успешной компиляции была выполнена локальная проверка работоспособности утилиты cpack для создания архивов формата .tar.gz, .deb и .rpm.

*cpack -G "TGZ"*

Вывод:

```bash
CPack: Create package using TGZ
CPack: Install projects
CPack: - Run preinstall target for: lab06
CPack: - Install project: lab06 []
CPack: Create package
CPack: - package: /home/vboxuser/workspace/lab06/_build/lab06-0.1.0.0-Linux.tar.gz generated.
```

*cpack -G "DEB"*

Вывод:

```bash
CPack: Create package using DEB
CPack: Install projects
CPack: - Run preinstall target for: lab06
CPack: - Install project: lab06 []
CPack: Create package
-- CPACK_DEBIAN_PACKAGE_DEPENDS not set, the package will have no dependencies.
CPack: - package: /home/vboxuser/workspace/lab06/_build/lab06-0.1.0.0-Linux.deb generated.
```

*cpack -G "RPM"*

Вывод:

```bash
CPack: Create package using RPM
CPack: Install projects
CPack: - Run preinstall target for: lab06
CPack: - Install project: lab06 []
CPack: Create package
CPackRPM: Will use GENERATED spec file: /home/vboxuser/workspace/lab06/_build/_CPack_Packages/Linux/RPM/SPECS/solver-devel.spec
CPack: - package: /home/vboxuser/workspace/lab06/_build/lab06-0.1.0.0-Linux.rpm generated.
```
*cd ..*

## Инициализация и очистка сценариев автоматизации

*mkdir -p .github/workflows*

*ls -la .github/workflows/*

Вывод:

```bash
итого 16
drwxrwxr-x 2 vboxuser vboxuser 4096 сен 26 19:32 .
drwxrwxr-x 3 vboxuser vboxuser 4096 сен 26 18:38 ..
-rw-rw-r-- 1 vboxuser vboxuser 1076 сен 26 18:38 ci.yml
-rw-rw-r-- 1 vboxuser vboxuser 2118 сен 26 19:32 release.yml
```

## Настройка конфигурации GitHub Actions

Содержимое *.github/workflows/release.yml*:

```yaml
name: Release Packages

on:
  push:
    tags:
      - 'v*'

jobs:
  build-deb-rpm:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Install tools
        run: sudo apt-get update && sudo apt-get install -y rpm

      - name: Configure
        run: cmake -S . -B _build -DCPACK_GENERATOR="DEB;RPM;TGZ"

      - name: Build
        run: cmake --build _build

      - name: Package
        run: cmake --build _build --target package

      - name: Upload artifacts
        uses: actions/upload-artifact@v4
        with:
          name: linux-packages
          path: |
            _build/*.deb
            _build/*.rpm
            _build/*.tar.gz

      - name: Upload to Release
        uses: softprops/action-gh-release@v2
        with:
          files: |
            _build/*.deb
            _build/*.rpm
            _build/*.tar.gz
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  build-dmg:
    runs-on: macos-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Configure
        run: cmake -S . -B _build -DCPACK_GENERATOR="DragNDrop"

      - name: Build
        run: cmake --build _build

      - name: Package
        run: cmake --build _build --target package

      - name: Upload to Release
        uses: softprops/action-gh-release@v2
        with:
          files: _build/*.dmg
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  build-msi:
    runs-on: windows-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive

      - name: Configure
        run: cmake -S . -B _build -DCPACK_GENERATOR="WIX" -G "Visual Studio 17 2022" -A x64

      - name: Build
        run: cmake --build _build --config Release

      - name: Package
        run: cmake --build _build --config Release --target package

      - name: Upload to Release
        uses: softprops/action-gh-release@v2
        with:
          files: _build/*.msi
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```
