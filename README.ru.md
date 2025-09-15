ripgrep (rg)
------------
ripgrep - это инструмент строкового поиска, который рекурсивно выполняет поиск шаблона регулярного
выражения в текущей директории. По умолчанию ripgrep будет учитывать gitignore правила
и автоматически пропускать скрытые файлы/директории и бинарные файлы. (Чтобы отключить
все автоматические фильтры по умолчанию, используйте `rg -uuu`.) ripgrep поддерживает
Windows, macOS и Linux, с доступными бинарными сборками для [каждого
релиза](https://github.com/BurntSushi/ripgrep/releases). ripgrep похож на другие
популярные поисковые инструменты, такие как The Silver Searcher, ack и grep.

[![Статус сборки](https://github.com/BurntSushi/ripgrep/workflows/ci/badge.svg)](https://github.com/BurntSushi/ripgrep/actions)
[![Crates.io](https://img.shields.io/crates/v/ripgrep.svg)](https://crates.io/crates/ripgrep)
[![Статус пакетирования](https://repology.org/badge/tiny-repos/ripgrep.svg)](https://repology.org/project/ripgrep/badges)

Двойная лицензия MIT или [UNLICENSE](https://unlicense.org).


### CHANGELOG

Пожалуйста, ознакомьтесь с [CHANGELOG](CHANGELOG.md) для истории релизов.

### Краткие ссылки на содержание документации

* [Установка](#installation)
* [Руководство пользователя](GUIDE.md)
* [Часто задаваемые вопросы](FAQ.md)
* [Синтаксис регулярных выражений](https://docs.rs/regex/1/regex/#syntax)
* [Файлы конфигурации](GUIDE.md#configuration-file)
* [Автодополнение для оболочки](FAQ.md#complete)
* [Сборка](#building)
* [Переводы](#translations)


### Скриншот результатов поиска

[![Скриншот примера поиска с помощью ripgrep](https://burntsushi.net/stuff/ripgrep1.png)](https://burntsushi.net/stuff/ripgrep1.png)


### Примеры для сравнения инструментов

В этом примере выполняется поиск по всему
[Дереву исходного кода ядра Linux](https://github.com/BurntSushi/linux)
(после выполнения `make defconfig && make -j8`) в поисках `[A-Z]+_SUSPEND`, где
все совпадения должны быть словами. Тайминги были собраны на системе с процессором Intel
i9-12900K 5.2 GHz.

Пожалуйста, помните, что одного бенчмарка никогда не бывает достаточно!
Ознакомьтесь с [постом про ripgrep](https://blog.burntsushi.net/ripgrep/)
для максимально детального сравнения бенчмарков и анализа.

| Инструмент | Команда | Количество строк | Время |
| ---- | ------- | ---------- | ---- |
| ripgrep (Unicode) | `rg -n -w '[A-Z]+_SUSPEND'` | 536 | **0.082s** (1.00x) |
| [hypergrep](https://github.com/p-ranav/hypergrep) | `hgrep -n -w '[A-Z]+_SUSPEND'` | 536 | 0.167s (2.04x) |
| [git grep](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html) | `git grep -P -n -w '[A-Z]+_SUSPEND'` | 536 | 0.273s (3.34x) |
| [The Silver Searcher](https://github.com/ggreer/the_silver_searcher) | `ag -w '[A-Z]+_SUSPEND'` | 534 | 0.443s (5.43x) |
| [ugrep](https://github.com/Genivia/ugrep) | `ugrep -r --ignore-files --no-hidden -I -w '[A-Z]+_SUSPEND'` | 536 | 0.639s (7.82x) |
| [git grep](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html) | `LC_ALL=C git grep -E -n -w '[A-Z]+_SUSPEND'` | 536 | 0.727s (8.91x) |
| [git grep (Unicode)](https://www.kernel.org/pub/software/scm/git/docs/git-grep.html) | `LC_ALL=en_US.UTF-8 git grep -E -n -w '[A-Z]+_SUSPEND'` | 536 | 2.670s (32.70x) |
| [ack](https://github.com/beyondgrep/ack3) | `ack -w '[A-Z]+_SUSPEND'` | 2677 | 2.935s (35.94x) |

Вот ещё один бенчмарк на той же основе, что и выше, который игнорирует файлы gitignore
и выполняет поиск с использованием белого списка. Основа такая же как и в предыдущем
тесте, а флаги, передаваемые каждой команде, гарантируют, что они выполняют одинаковую работу.

| Инструмент | Команда | Количество строк | Время |
| ---- | ------- | ---------- | ---- |
| ripgrep | `rg -uuu -tc -n -w '[A-Z]+_SUSPEND'` | 447 | **0.063s** (1.00x) |
| [ugrep](https://github.com/Genivia/ugrep) | `ugrep -r -n --include='*.c' --include='*.h' -w '[A-Z]+_SUSPEND'` | 447 | 0.607s (9.62x) |
| [GNU grep](https://www.gnu.org/software/grep/) | `grep -E -r -n --include='*.c' --include='*.h' -w '[A-Z]+_SUSPEND'` | 447 | 0.674s (10.69x) |

Теперь перейдём к поиску по одному большому файлу. Вот пример
сравнения между ripgrep, ugrep и GNU grep на кэшированном в памяти файле
(~13GB, [`OpenSubtitles.raw.en.gz`](http://opus.nlpl.eu/download.php?f=OpenSubtitles/v2018/mono/OpenSubtitles.raw.en.gz), decompressed):

| Инструмент | Команда | Количество строк | Время |
| ---- | ------- | ---------- | ---- |
| ripgrep (Unicode) | `rg -w 'Sherlock [A-Z]\w+'` | 7882 | **1.042s** (1.00x) |
| [ugrep](https://github.com/Genivia/ugrep) | `ugrep -w 'Sherlock [A-Z]\w+'` | 7882 | 1.339s (1.28x) |
| [GNU grep (Unicode)](https://www.gnu.org/software/grep/) | `LC_ALL=en_US.UTF-8 egrep -w 'Sherlock [A-Z]\w+'` | 7882 | 6.577s (6.31x) |

В бенчмарке выше, использование флага `-n` (для отображения номеров строк)
увеличивает время до `1.664s` для ripgrep и `9.484s` для GNU grep.
Флаг `-n` не влияет на время upgrep.

Остерегайтесь резких скачков производительности:

| Инструмент | Команда | Количество строк | Время |
| ---- | ------- | ---------- | ---- |
| ripgrep (Unicode) | `rg -w '[A-Z]\w+ Sherlock [A-Z]\w+'` | 485 | **1.053s** (1.00x) |
| [GNU grep (Unicode)](https://www.gnu.org/software/grep/) | `LC_ALL=en_US.UTF-8 grep -E -w '[A-Z]\w+ Sherlock [A-Z]\w+'` | 485 | 6.234s (5.92x) |
| [ugrep](https://github.com/Genivia/ugrep) | `ugrep -w '[A-Z]\w+ Sherlock [A-Z]\w+'` | 485 | 28.973s (27.51x) |

Кроме того, производительность может резко снизиться по всем параметрам
при поиске шаблона в больших файлах без каких-либо возможностей для оптимизации.

| Инструмент | Команда | Количество строк | Время |
| ---- | ------- | ---------- | ---- |
| ripgrep | `rg '[A-Za-z]{30}'` | 6749 | **15.569s** (1.00x) |
| [ugrep](https://github.com/Genivia/ugrep) | `ugrep -E '[A-Za-z]{30}'` | 6749 | 21.857s (1.40x) |
| [GNU grep](https://www.gnu.org/software/grep/) | `LC_ALL=C grep -E '[A-Za-z]{30}'` | 6749 | 32.409s (2.08x) |
| [GNU grep (Unicode)](https://www.gnu.org/software/grep/) | `LC_ALL=en_US.UTF-8 grep -E '[A-Za-z]{30}'` | 6795 | 8m30s (32.74x) |

Наконец, большое количество совпадение также способствует повышению производительности
и нивелированию различий между инструментами (посколько производительность определяется тем,
насколько быстро можно обработать совпадение, а не алгоритмом, используемым для
обнаружение совпадения):

| Инструмент | Команда | Количество строк | Время |
| ---- | ------- | ---------- | ---- |
| ripgrep | `rg the` | 83499915 | **6.948s** (1.00x) |
| [ugrep](https://github.com/Genivia/ugrep) | `ugrep the` | 83499915 | 11.721s (1.69x) |
| [GNU grep](https://www.gnu.org/software/grep/) | `LC_ALL=C grep the` | 83499915 | 15.217s (2.19x) |

### Почему стоит использовать ripgrep?

* Он может заменить многие функции, представляемые другими поисковыми инструментами
  (Ознакомьтесь с [FAQ](FAQ.md#posix4ever) для получения более подробной информации
  о том, может ли ripgrep заменить grep.)
* Как и другие инструменты, специализирующиеся на поиске кода, ripgrep по умолчанию
  использует [рекурсивный поиск](GUIDE.md#recursive-search) и выполняет [автоматическую
  фильтрацию](GUIDE.md#automatic-filtering). А, именно ripgrep не будет искать файлы
  игнорируемые вашими `.gitignore`/`.ignore`/`.rgignore` файлами, не будет искать
  скрытые файлы и не будет искать бинарные файлы. Автоматическую фильтрацию можно
  отключить при помощи `rg -uuu`.
* ripgrep может выполнять [поиск по определённым типам файлов](GUIDE.md#manual-filtering-file-types).
  Например, `rg -tpy foo` ограничевает поиск только по Python файлам, а `rg -Tjs
  foo` исключает JavaScript файлы из поиска. ripgrep может распознать и другие
  типы файлов с помощью пользовательских правил сопоставления.
* ripgrep поддерживает многие функции найденные в `grep`, такие как отображение контекста
  результатов поиска, поиск по нескольким шаблонам, выделение совпадений цветом
  и полная поддержка Unicode. В отличие от GNU grep, ripgrep остаётся быстрым,
  поддерживая Unicode (поддержка которого всегда включена).
* ripgrep имеет дополнительную поддержку для переключения своего движка regex
  на использование PCRE2. Среди прочего, это позволяет использовать look-around и
  backreferences в ваших шаблонах, которые не поддерживаются в движке
  regex ripgrep по умолчанию. PCRE2 можно включить при помощи `-P/--pcre2` (использовать PCRE2
  всегда) или `--auto-hybrid-regex` (использовать PCRE2 только при необходимости). Можно
  использовать и альтернативный синтаксис `--engine (default|pcre2|auto)`.
* ripgrep имеет [базовую поддержку замен](GUIDE.md#replacements),
  которая позволяет переписывать выходные данные на основе того, что было сопоставлено.
* ripgrep поддерживает [поиск в других текстовых кодировках](GUIDE.md#file-encoding)
  отличных от UTF-8, таких как UTF-16, latin-1, GBK, EUC-JP, Shift_JIS и других.
  (Предусмотрена некоторая поддержка автоматическог определения UTF-16. Другие
  кодировки должны быть специально указаны с помощью флага `-E/--encoding`.)
* ripgrep поддерживает поиск файлов, сжатых в стандартные форматы (brotli,
  bzip2, gzip, lz4, lzma, xz, or zstandard) с помощью флага `-z/--search-zip`.
* ripgrep поддерживает
  [произвольные фильтры обработки входных данных](GUIDE.md#preprocessor)
  которые могут быть использованы для извлечения текста PDF, 
  менее поддерживаемых типов сжатых файлов, расшифровки,
  автоматического распознования кодировки текста и прочего.
* ripgrep может быть сконфигурирован при помощи
  [configuration file](GUIDE.md#configuration-file).

Другими словами, используйте ripgrep, если вам нравится скорость, фильтрация
по умолчанию, меньшее количество багов и поддержка Unicode.

### Почему не стоит использовать ripgrep?

Не смотря на то что изначально в ripgrep не хотели добавлять все возможные функции,
со временем ripgrep расширил поддержку большинства функций, которые можно найти
в других инструментах поиска. Это включает в себя поиск результатов по нескольким
строкам и поддержку движка PCRE2, которая обеспечивает поиск по всему каталогу и
обратные ссылки.

На данный момент основными причинами почему стоит отказаться от использования ripgrep
являются одна или несколько из следующих:

* Вам нужен портативный и кроссплатформенный инструмент. Хоть ripgrep и работает 
  в Windows, macOS и Linux, он не является полностью кроссплатформенным и не
  соответствует ни одному стандарту, такому как POSIX. Более подходящим инструментом
  в таком случае является grep.
* Всё ещё существует какая-то другая функция (или баг), не указанная в этом README,
  на которую вы полагаетесь и которая имеется в другом инструменте, которого нет в
  ripgrep.
* Существует проблема с производительностью, когда ripgrep не работает там, где другой
  инструмент работает хорошо. (Пожалуйста отправьте отчёт об ошибке!)
* ripgrep невозможно установить на ваш компьютер или он не доступен для вашей платформы
  (Пожалуйста отправьте отчёт об ошибке!)


### Действительно ли он быстрее всех остальных инструментов?

В целом, да. Вы можете ознакомится с бенчмарками и детальным анализом в
[моём блоге](https://blog.burntsushi.net/ripgrep/).

В общих чертах, ripgrep быстрее потому что:

* Он построен на базе
  [регулярных выражений Rust](https://github.com/rust-lang/regex).
  Движок регулярных выражений Rust использует finite automata, SIMD и агрессивную
  оптимизацию чтобы сделать процесс поиска очень быстрым. (поддержка PCRE2 может быть включена
  с помощью флага `-P/--pcre2`.)
* Библиотека регулярных выражений Rust поддерживает производительность при полной поддержке
  Unicode, встраивая декодирование UTF-8 непосредственно в свой finite automaton engine.
* Он поддерживает поиск с использованием карт памяти или поэтапный поиск с использованием
  промежуточного буфера. Первый вариант лучше подходит для отдельных файлов, а второй - для
  больших каталогов. ripgrep автоматически выбирает наилучшую стратегию поиска.
* Автоматически применяет шаблоны `.gitignore` используя
  [`RegexSet`](https://docs.rs/regex/1/regex/struct.RegexSet.html).
  Это означает, что один путь к файлу может быть сопоставлен с несколькими шаблонами
  глобальных объектов одновременно.
* Он использует параллельный рекурсивный итератор без блокировок, предоставленный
  [`crossbeam`](https://docs.rs/crossbeam) и
  [`ignore`](https://docs.rs/ignore).


### Сравнение функционала

Энди Лестер, автор [ack](https://beyondgrep.com/), опубликовал отличную
таблицу, в которой сравниваются возможности  ack, ag, git-grep, GNU grep и
ripgrep: https://beyondgrep.com/feature-comparison/

Обратите внмание, что в последнее время в ripgrep появились несколько важных
новых функций, которые ещё не предоставлены в таблице. Это включает, помимо прочего,
файлы конфигурации, passthru, поддержку поиска в сжатых файлах, многострочный поиск
и поддержку модных регулярных выражений через PCRE2.


### Playground

Если вы хотите попробовать ripgrep перед установкой, есть неофициальный
[playground](https://codapi.org/ripgrep/) и [interactive
tutorial](https://codapi.org/try/ripgrep/).

Если у вас есть какие-либо вопросы по этому поводу, пожалуйста, откройте issue в [tutorial
repo](https://github.com/nalgeon/tryxinyminutes).


### Установка

Имя бинарного файла для ripgrep - `rg`.

**[Архивы скомпилированных бинарных файлов ripgrep доступны для Windows,
macOS и Linux.](https://github.com/BurntSushi/ripgrep/releases)** Бинарные файлы
Linux и Windows являются статическими исполняемыми файлами. Пользователям платформ,
явно не указанных ниже, рекомендуется загрузить один из этих архивов.

Если вы являетесь пользователем **macOS Homebrew** или **Linuxbrew**, вы можете
установить ripgrep из homebrew-core:

```
$ brew install ripgrep
```

Если вы пользователь **MacPorts**, вы можете установить ripgrep из
[official ports](https://www.macports.org/ports.php?by=name&substr=ripgrep):

```
$ sudo port install ripgrep
```

Если вы пользователь **Windows Chocolatey**, тогда вы можете установить ripgrep из
[official repo](https://chocolatey.org/packages/ripgrep):

```
$ choco install ripgrep
```

Если вы пользователь **Windows Scoop**, вы можете установить ripgrep из
[official bucket](https://github.com/ScoopInstaller/Main/blob/master/bucket/ripgrep.json):

```
$ scoop install ripgrep
```

Если вы пользователь **Windows Winget**, вы можете установить ripgrep из
[winget-pkgs](https://github.com/microsoft/winget-pkgs/tree/master/manifests/b/BurntSushi/ripgrep)
repository:

```
$ winget install BurntSushi.ripgrep.MSVC
```

Если вы пользователь **Arch Linux**, вы можете установить ripgrep из официальных репозиториев:

```
$ sudo pacman -S ripgrep
```

Если вы пользователь **Gentoo**, вы можете установить ripgrep из
[official repo](https://packages.gentoo.org/packages/sys-apps/ripgrep):

```
$ sudo emerge sys-apps/ripgrep
```

Если вы пользователь **Fedora**, вы можете установить ripgrep из офизиальных репозиториев.

```
$ sudo dnf install ripgrep
```

Если вы пользователь **openSUSE**, ripgrep уже включён в **openSUSE Tumbleweed**
и **openSUSE Leap** с версии 15.1.

```
$ sudo zypper install ripgrep
```

Если вы пользователь **RHEL/CentOS 7/8**, вы можете установить ripgrep из
[copr](https://copr.fedorainfracloud.org/coprs/carlwgeorge/ripgrep/):

```
$ sudo yum install -y yum-utils
$ sudo yum-config-manager --add-repo=https://copr.fedorainfracloud.org/coprs/carlwgeorge/ripgrep/repo/epel-7/carlwgeorge-ripgrep-epel-7.repo
$ sudo yum install ripgrep
```

Если вы пользователь **Nix**, вы можете установить ripgrep из
[nixpkgs](https://github.com/NixOS/nixpkgs/blob/master/pkgs/tools/text/ripgrep/default.nix):

```
$ nix-env --install ripgrep
```

Если вы пользователь **Flox**, вы можете установить ripgrep так:

```
$ flox install ripgrep
```

Если вы пользователь **Guix**, вы можете установить ripgrep из официальной коллекции
пакетов:

```
$ guix install ripgrep
```

Если вы пользователь **Debian** (или пользователь производной Debian, такой как **Ubuntu**),
тогда ripgrep может быть установлен используя бинарный `.deb` файл предоставляемый в каждом
[ripgrep release](https://github.com/BurntSushi/ripgrep/releases).

```
$ curl -LO https://github.com/BurntSushi/ripgrep/releases/download/14.1.0/ripgrep_14.1.1-1_amd64.deb
$ sudo dpkg -i ripgrep_14.1.1-1_amd64.deb
```

Если вы используете **Debian Stable**, ripgrep  [официально поддерживается
Debian](https://tracker.debian.org/pkg/rust-ripgrep), хотя его версия может быть
старше, чем пакет `deb` предоставляемый на предыдущем шаге.

```
$ sudo apt-get install ripgrep
```

Если вы пользователь **Ubuntu Cosmic (18.10)** (или новее), ripgrep
[доступе](https://launchpad.net/ubuntu/+source/rust-ripgrep) используя тот же
пакет что и Debian:

```
$ sudo apt-get install ripgrep
```

(ПРИМЕЧАНИЕ: Доступны различные snap для ripgrep в Ubuntu, но ни один из них,
не работает должным образом и генерирует ряд очень странных ошибок, которые
Я не знаю как исправить, и уменя нет времени их исправлять. Таким образом,
этот вариант установки не рекомендован.)

Если вы пользователь **ALT**, вы можете установить ripgrep из
[official repo](https://packages.altlinux.org/en/search?name=ripgrep):

```
$ sudo apt-get install ripgrep
```

Если вы пользователь **FreeBSD**, вы можете установить ripgrep из
[official ports](https://www.freshports.org/textproc/ripgrep/):

```
$ sudo pkg install ripgrep
```

Если вы пользователь **OpenBSD**, вы можете установить ripgrep из
[official ports](https://openports.se/textproc/ripgrep):

```
$ doas pkg_add ripgrep
```

Если вы пользователь **NetBSD**, вы можете установить ripgrep из
[pkgsrc](https://pkgsrc.se/textproc/ripgrep):

```
$ sudo pkgin install ripgrep
```

Если вы пользователь **Haiku x86_64**, вы можете установить ripgrep из
[official ports](https://github.com/haikuports/haikuports/tree/master/sys-apps/ripgrep):

```
$ sudo pkgman install ripgrep
```

Если вы пользователь **Haiku x86_gcc2**, вы можете установить ripgrep из
из того же port что и Haiku x86_64 используя вторичную сборку архитектуры x86:

```
$ sudo pkgman install ripgrep_x86
```

Если вы пользователь **Void Linux**, вы можете установить ripgrep из
[official repository](https://voidlinux.org/packages/?arch=x86_64&q=ripgrep):

```
$ sudo xbps-install -Syv ripgrep
```

Если вы **программист на Rust**, ripgrep можно установить с помощью `cargo`.

* Обратите внимание, что минимальная поддерживаемая версия Rust
  для ripgrep - **1.88.0**
* Обратите внимание, что бинарный файл может быть больше чем ожидалось, так как он
  символы отладки. Это сделано специально. Чтобы убрать символы отладки и, следовательно,
  уменьшить размер файла, используйте `strip` на бинарном файле.

```
$ cargo install ripgrep
```

В качестве альтернативы можно использовать [`cargo
binstall`](https://github.com/cargo-bins/cargo-binstall) для установки
бинарного файла ripgrep непосредственно с GitHub:

```
$ cargo binstall ripgrep
```


### Сборка

ripgrep написан на Rust, поэтому для его компиляции понадобится
[Rust installation](https://www.rust-lang.org/).
ripgrep компилируется на Rust 1.88.0 (stable) или новее. В целом, ripgrep отслеживает
последнюю версию компилятора Rust.

Чтобы собрать ripgrep:

```
$ git clone https://github.com/BurntSushi/ripgrep
$ cd ripgrep
$ cargo build --release
$ ./target/release/rg --version
0.1.3
```

**ПРИМЕЧАНИЕ:** В прошлом, ripgrep поддерживал функцию Cargo `simd-accel` при 
использовании компилятора Rust nightly. Это приносило пользу только при
перекодировании UTF-16. Посколько для этого требовались нестабильные функции,
этот способ сборки был подвержен поломкам. Из-за этого его поддержка была удалена.
Если вы хотите оптимизации SIMD для перекодировки в UTF-16, вам придётся обратиться
в проект [`encoding_rs`](https://github.com/hsivonen/encoding_rs) с просьбой использовать
стабильные API.

Наконец, дополнительная поддержка PCRE2 может быть собрана включив функцию
`pcre2`:

```
$ cargo build --release --features 'pcre2'
```

Включение функции PCRE2 работает с Rust stable компилятором и будет пытаться
автоматически найти библиотеку PCRE2 вашей системы и связать её через `pkg-config`.
Если библиотеки не существует, то ripgrep соберёт PCRE2 из исходного кода с помощью
C-компилятора вашей системы, а затем статически свяжет его с конечным исполняемым
файлом. Статическое связываение можно принудительно выполнить даже при наличии
доступной системной библиотеки PCRE2, либо собрав ripgprep с MUSL target, либо
установив `PCRE2_SYS_STATIC=1`

ripgrep можно создать с MUSL target в Linux, предварительно установив библиотеку
MUSL в вашу систему (используйте менеджер пакетов на ваш выбор). Затем просто нужно
добавить поддержку MUSL в toolchain Rust и пересобрать ripgrep, что даст полностью
статический исполняемый файл:

```
$ rustup target add x86_64-unknown-linux-musl
$ cargo build --release --target x86_64-unknown-linux-musl
```

Использование флага `--features`, указанного выше, работает так, как ожидалось.
Если вы хотите создать статический исполняемый файл с MUSL и PCRE2, то вам нужно
установить `musl-gcc`, который может находиться в отдельном пакете от фактической
библиотеки MUSL, в зависимости от вашего дистрибутива Linux.


### Выполнение тестов

ripgrep относительно хорошо протестирован, включая как модульные, так и
интеграционные тесты. Чтобы запустить полный набор тестов используйте:

```
$ cargo test --all
```

из корня репозитория.


### Сопутствующие инструменты

* [delta](https://github.com/dandavison/delta) - это пейджер с подсветкой синтаксиса,
который поддерживает формат вывода `rg --json`. Так что всё, что вам нужно сделать,
чтобы он работал, это `rg --json pattern | delta`. Более подробную информацию
смотрите в разделе [delta's manual section on grep](https://dandavison.github.io/delta/grep.html)

### Отчёт об уязвимостях

Чтобы сообщить об уязвимости, пожалуйста [свяжитесь с Эндрю Галлантом](https://blog.burntsushi.net/about/).
На странице контактов указан его адрес электронной почты и открытый ключ PGP,
если вы хотите отправить зашифрованное сообщение.

### Переводы

Ниже приведён список известных переводов документации ripgrep. Они поддерживаются
неофициально и могут быть устаревшими.

* [Китайский](https://github.com/chinanf-boy/ripgrep-zh#%E6%9B%B4%E6%96%B0-)
* [Испанский](https://github.com/UltiRequiem/traducciones/tree/master/ripgrep)
