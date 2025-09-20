## Руководство Пользователя

Это руководство предусмотренно для базового ознакомления с 
возможностями ripgrep и общего обзора его функций. Предпологается
что ripgrep уже [установлен](README.md#installation) и что
читатели в некоторой степени знакомы с использованием инструментов
командной строки. Также предполагается использование Unix-подобной
системы, хотя большинство команд можно легко перевести в любую среду
командной строки.


### Содержание

* [Основы](#основы)
* [Рекурсивный поиск](#рекурсивный-поиск)
* [Автоматическая фильтрация](#автоматическая-фильтрация)
* [Ручная фильтрация: globs](#ручная-фильтрация-globs)
* [Ручная фильтрация: типы файлов](#ручная-фильтрация-типы-файлов)
* [Замены](#замены)
* [Конфигурационный файл](#конфигурационный-файл)
* [Кодировка файлов](#Кодировка-файлов)
* [Бинарные данные](#бинарные-данные)
* [Препроцессор](#препроцессор)
* [Общие параметры](#общие-параметры)


### Основы

ripgrep - это инструмент командной строки, который выполняет поиск в
файлах по заданному шаблону. ripgrep читает каждый файл построчно. Если
строка соответствует шаблону, то эта строка будет выведена. Если строка не
соответствует шаблону, то она не выводится.

Лучший способ разобраться с тем как это работает - на примере. Чтобы
показать пример, нам нужен какой-то файл чтобы выполнить по нему поиск.
Давайте попробуем выполнить поиск в исходном коде ripgprep. Сначала
возьмём архив с исходным кодом ripgprep из https://github.com/BurntSushi/ripgrep/archive/0.7.1.zip и распакуем его:

```
$ curl -LO https://github.com/BurntSushi/ripgrep/archive/0.7.1.zip
$ unzip 0.7.1.zip
$ cd ripgrep-0.7.1
$ ls
benchsuite  grep       tests         Cargo.toml       LICENSE-MIT
ci          ignore     wincolor      CHANGELOG.md     README.md
complete    pkg        appveyor.yml  compile          snapcraft.yaml
doc         src        build.rs      COPYING          UNLICENSE
globset     termcolor  Cargo.lock    HomebrewFormula
```

Давайте попробуем выполнить наш первый поиск, найдя все строки где
встречается слово `fast` в файле `README.md`:

```
$ rg fast README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast while
119:### Is it really faster than everything else?
124:Summarizing, `ripgrep` is fast because:
129:  optimizations to make searching very fast.
```

(**ПРИМЕЧАНИЕ:** Если вы встретились с сообщением об ошибке от ripgprep, 
в котором говорится что поиск по файлам не был выполнен, запустите
ripgrep повторно с флагом `--debug`. Одной из вероятных причин этого
является правило `*` в файле `$HOME/.gitignore`.)

Давайте разберём что произошло. ripgrep прочитал содержимое `README.md`,
и для каждой строки, содержащей `fast`, ripgrep вывел её в ваш терминал.
По умолчанию ripgrep также включил номер для каждой строки. Если ваш
терминал поддерживает цвета, то ваш вывод может выглядеть примерно так:

[![A screenshot of a sample search ripgrep](https://burntsushi.net/stuff/ripgrep-guide-sample.png)](https://burntsushi.net/stuff/ripgrep-guide-sample.png)

В этом примере мы искали нечто, называемое "буквальной" строкой. Это
означает, что наш шаблон был просто обычным текстом, который мы попросили
найти. Но ripgrep поддерживает возможность создания шаблонов с помощью
[регулярных выражений](https://ru.wikipedia.org/wiki/Regular_expression). Например, что, если бы мы захотели найти все
появления слова `fast`, за которым следует некоторое количество букв?

```
$ rg 'fast\w+' README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
119:### Is it really faster than everything else?
```

В этом примере мы использовали шаблон`fast\w+`. Этот шаблон указывает 
ripgprep найти все строки содержащие сочетание букв `fast`, за которыми
следуют *один или несколько символов*, похожих на слова. А именно, `\w`
соответствует символам, из которых состоят слова (например как `a` и `L`,
но не `.` и ` `.). Знак `+` после `\w` означает "повторить предыдущий
шаблон один или несколько раз". Это означает что слово `fast` не будет
совпадать с шаблоном, поскольку после последней `t` нет подходящих 
символов. Но такое слово, как `faster`, будет соответствовать. `faste` бы
тоже подошло!

Вот другой вариант выполнения того же примера:

```
$ rg 'fast\w*' README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast while
119:### Is it really faster than everything else?
124:Summarizing, `ripgrep` is fast because:
129:  optimizations to make searching very fast.
```

В этом случае мы использовали `fast\w*` для нашего шаблона вместо
`fast\w+`. Знак `*` означает что шаблон должен совпасть *ноль* или больше
раз. В таком случае, ripgrep выведет те же строки что и шаблон `fast`, но
если ваш терминал поддерживает цвета, вы заметите что `faster` будет
выделено цветом вместо префикса `fast`.

Полное объяснение пользования регулярными выражениями выходит за рамки
данного руководства, но докумнтацию специфического синтаксиса ripgrep
можно найти здесь:
https://docs.rs/regex/*/regex/#syntax


### Рекурсивный поиск

В предыдущем разделе, мы показали как использовать ripgrep для поиска по одному файлу.
В этом разделе, мы покажем как использовать ripgrep для поиска по целой директории файлов.
На самом деле, *рекурсивный поиск* в текущей рабочей директории является режимом работы
ripgrep по умолчанию, а это значит что выполнить это очень просто.

Используя наш распакованный архив исходного кода ripgrep, вот пример как найти все
объявленные функции с именем `write`:

```
$ rg 'fn write\('
src/printer.rs
469:    fn write(&mut self, buf: &[u8]) {

termcolor/src/lib.rs
227:    fn write(&mut self, b: &[u8]) -> io::Result<usize> {
250:    fn write(&mut self, b: &[u8]) -> io::Result<usize> {
428:    fn write(&mut self, b: &[u8]) -> io::Result<usize> { self.wtr.write(b) }
441:    fn write(&mut self, b: &[u8]) -> io::Result<usize> { self.wtr.write(b) }
454:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
511:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
848:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
915:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
949:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
1114:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
1348:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
1353:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
```

(**ПРИМЕЧАНИЕ:** Мы избегаем `(` здесь, потому что `(` имеет особое значение внутри
регулярных выражений. Также можно использовать `rg -F 'fn write('` для достижения
того же результата, где флаг `-F` интерпретирует ваш шаблон как буквальную строку
вместо регулярного выражения.)

В этом примере мы совсем не указывали файл. Вместо этого ripgrep по умолчанию
выполнил поиск в вашей текущей директории по отсутствию пути. В общем случае
`rg foo` эквивалентно `rg foo ./`.

Этот поиск показал результаты в обоих директориях `src` и `termcolor`. Директория
`src` - это основной код ripgrep, а `termcolor` является зависимостью от ripgrep
(и используется другими инструментами.) Что если бы мы хотели выполнить поиск только
по основному коду ripgrep? Это делается очень просто, достаточно указать нужную директорию:

```
$ rg 'fn write\(' src
src/printer.rs
469:    fn write(&mut self, buf: &[u8]) {
```

В данном случае ripgrep ограничил поиск директорией `src`. Другим способом выполнения
этого поиска было бы перейти в директорию `src` при помощи `cd` и просто снова использовать
`rg 'fn write\('`.

### Автоматическая фильтрация

После рекурсивного поиска наиболее важной особенностью ripgrep является то,
по чему он не будет выполнять поиск. По умолчанию при поиске в директории ripgprep
игнорирует всё перечисленное ниже:

1. Файлы и директории, соответствующие шаблонам glob в этих трёх категориях:
      1. `.gitignore` globs (включая глобальные и спцифичные для репозитория globs). Это
         учитиывает `.gitignore` файлы в родительских директориях которые являются
         частью того же `git` репозитория. (Только в случае если не будет задан флаг
         `--no-require-git`.)
      2. `.ignore` globs, которые при возникновении конфликта имеют приоритет над всеми
         всеми gitignore globs. Это включает файлы `.ignore` в родительских директориях.
      3. `.rgignore` globs, которые при возникновении конфликта имеют приоритет над всеми
         `.ignore` globs. Это включает файлы `.rgignore` в родительских директориях.
2. Спрятанные файлы и директории.
3. Бинарные файлы. (ripgrep рассматривает любтые файлы с байтом `NUL` как бинарные.)
4. Символические ссылки не отслеживаются.

Все эти функции можно переключать с помощью различных флагов, предоставляемых
ripgrep:

1. Вы можете отключить все фильтры связанные с ignore при помощи флага`--no-ignore`.
2. Поиск по спрятанным файлам и директориям можно включить с помощью `--hidden` (сокращённо `-.`).
3. Поиск по бинарным файлам можно включить с помощью флага `--text` (сокращённо `-a`).
   Будьте осторожны с этим флагом! Бинарные файлы могут выдавать управляющие символы в ваш
   терминал, что может привести к неожиданным результатам.
4. ripgrep может отслеживать символические ссылки с помощью флага `--follow` (сокращённо `-L`).

Для удобства ripgrep также представляет флаг `--unrestricted` (сокращённо `-u`).
Повторное использование этого флага приведёт к тому, что ripgrep будет отключать всё больше
и больше функций фильтрации. То есть`-u` отключит обработку `.gitignore`, `-uu` выполнит поиск
по скрытым файлам и директориям, а `-uuu` выполнит поиск бинарных файлов. Это полезно, когда вы
используете ripgrep и не уверены, скрывает ли его фильтрация от вас результаты. Установка пары
флагов `-u` - это быстрый способ выяснить это. (Используйте флаг `--debug`, если вы всё ещё 
не уверены, и если это не поможет, [оформите issue](https://github.com/BurntSushi/ripgprep/issues/new).)

Обработка `.gitignore` в ripgrep на самом деле выходит за рамки простых файлов `.gitignore`.
ripgrep также будет соблюдать правила, специфичные для репозитория, найденные в
`$GIT_DIR/info/exclude`, а также любые глобальные правила ignore в вашем `core.excludesFile`
(который обычно является `$XDG_CONFIG_HOME/git/ignore` в Unix-подобных системах).

Иногда требуется выполнить поиск в файлах, которые находятся в вашем `.gitignore`, поэтому
можно указать дополнительные правила ignore или переопределения в файле `.ignore`
(не зависящем от приложения) или `.rgignore` (специфичном для ripgrep).

Например, предположим, что у вас есть файл `.gitignore`, который выглядит следующим образом:

```
log/
```

Как правило, это означает, что ни один каталог `log` не будет отслеживается `git`.
Однако, возможно он содержит полезные данные, которые вы хотели бы включить в свой поиск, но
по-прежнему не хотите отслеживать их в `git`. Вы можете добиться этого, создав файл `.ignore`
в той же директории, что и файл `.gitignore`, со следующим содержимым:

```
!log/
```

ripgrep обрабатывает файлы `.ignore` с более высоким приоритетом, чем файлы `.gitignore`
(и обрабатывает файлы `.rgignore` с более высоким приоритетом, чем файлы `.ignore`).
Это означает, что ripgrep сначала увидит правило белого списка `!log/` и выполнит
полис в этой директории.

Как и `.gitignore`, файл `.ignore` может быть помещён в любую директорию. Его правила будут
обрабатываться в соответствии с директорией, в которой он находится, точно так же, как
`.gitignore`.

Чтобы обрабатывать файлы `.gitignore` и `.ignore` без учёта регистра, используйте флаг
`--ignore-file-case-insensitive`. Это особенно полезно в файловых системах, не чувствительных
к регистру, таких как Windows и macOS. Однако, обратите внимание, что это может привести
к значительному снижению производительности и поэтому по умолчанию отключено.

Более подробное описание того, как интерпретируются шаблоны glob в файле `.gitignore`,
смотрите в `man gitignore`.


### Ручная фильтрация: globs

В предыдущем разделе мы говорили о фильтрации в ripgrep, которую он выполняет по
умолчанию. Она "автоматическая", поскольку реагирует на ваше окружение. То есть она
использует уже существующие файлы `.gitignore` для получения более релевантных
результатов поиска.

В дополнение к автоматической фильтрации, ripgrep также предоставляет дополнительные
возможности ручной или специальной фильтрации. Она бывает двух видов: дополнительные 
шаблоны glob, указанные в командах ripgrep, и фильтрация по типу файла. В этом
разделе рассматриваются шаблоны glob, а в следующем разделе - фильтрация по типу файла.

В нашем исходном коде ripgrep (инструкции о том, как получить архив с исходным кодом 
для поиска, смотрите в [Основы](#основы)), допустим, мы хотели посмотреть, какие функции
зависят от `clap`, нашего парсера аргументов.

Мы могли бы сделать следующее:

```
$ rg clap
[lots of results]
```

Но это показывает нам слишком много ненужной информации, а нас интересует только то,
где мы написали `clap` как зависимость. Вместо этого мы могли бы ограничиться файлами TOML,
с помощью которых зависимости передаются в инструмент сборки Rust, Cargo:

```
$ rg clap -g '*.toml'
Cargo.toml
35:clap = "2.26"
51:clap = "2.26"
```

Синтаксис `-g '*.toml'` означает: "убедитесь, что каждый искомый файл соответствует этому
glob шаблону". Обратите внимание, что мы заключили `'*.toml'` в одинарные кавычки, чтобы наша
оболочка не расширяла `*`.

Если бы мы хотели, мы могли бы указать ripgrep включить в поиск все файлы кроме файлов с
расширением `.toml`:

```
$ rg clap -g '!*.toml'
[lots of results]
```

Это снова даст вам множество результатов, как было в примере выше, но они не будут включать
файлы, заканчивающиеся на `.toml`. Обратите внимание, что использование `!` здесь для 
обозначения "отрицания" немного нестандартно, но оно было выбрано в соответствии с тем,
как записываются glob в файлах ".gitignore". (Хотя, значение является противоположностью.
В файлах `.gitignore` префикс `!` означает белый список, а в командной строке `!` означает
черный список.)

Glob интерпретируются точно так же, как шаблоны `.gitignore`. То есть более поздние 
glob будут переопределять более ранние glob. Например, следующая команда будет выполнять 
поиск только в файлах `*.toml`.:

```
$ rg clap -g '!*.toml' -g '*.toml'
```

Интересно, что изменение порядка следования glob в этом случае ничего не даст, поскольку 
наличие хотя бы одного glob, не внесенного в черный список , будет требовать, чтобы каждый 
искомый файл соответствовал хотя бы одному glob. В этом случае глобальный объект черного 
списка имеет приоритет над предыдущим глобальным объектом и вообще предотвращает поиск в 
каком-либо файле!

### Ручная фильтрация: типы файлов

Со временем вы можете заметить, что снова и снова используете одни и те же шаблоны glob.
Например, вы можете обнаружить, что выполняете множество поисковых
запросов, в которых хотите видеть результаты только для файлов Rust:

```
$ rg 'fn run' -g '*.rs'
```

Вместо того, чтобы каждый раз записывать glob, вы можете использовать поддержку ripgrep
типов файлов:

```
$ rg 'fn run' --type rust
```

или, более кратко:

```
$ rg 'fn run' -trust
```

Принцип действия флага `--type` прост. Он действует как имя, присваиваемое одному или
нескольким glob, соответствующим подходящим файлам. Это позволяет вам создать единый тип,
который может охватывать широкий диапазон расширений файлов. Например, если вы хотите 
выполнить поиск в файлах C, вам нужно будет проверить как исходные файлы C, так и заголовочные
файлы C:

```
$ rg 'int main' -g '*.{c,h}'
```

Или вы могли бы просто использовать тип файла C:

```
$ rg 'int main' -tc
```

Точно так же, как вы можете создавать glob с черными списками, вы также можете вносить
в черный список типы файлов:

```
$ rg clap --type-not rust
```

или, более кратко:

```
$ rg clap -Trust
```

То есть `-t` означает "включать файлы этого типа", где `-T` означает "исключать
файлы этого типа".

Чтобы просмотреть glob, составляющие типы, запустите `rg --type-list`:

```
$ rg --type-list | rg '^make:'
make: *.mak, *.mk, GNUmakefile, Gnumakefile, Makefile, gnumakefile, makefile
```

По умолчанию ripgrep поставляется с набором предопределенных типов. Как правило, эти
типы соответствуют хорошо известным общедоступным форматам. Но вы также можете определить
свои собственные типы. Например, возможно, вы часто выполняете поиск в "веб-файлах", которые
состоят из JavaScript, HTML и CSS:

```
$ rg --type-add 'web:*.html' --type-add 'web:*.css' --type-add 'web:*.js' -tweb title
```

или, более кратко:

```
$ rg --type-add 'web:*.{html,css,js}' -tweb title
```

Приведенная выше команда определяет новый тип `web`, соответствующий glob объекту
`*.{html,css,js}`. Затем она применяет новый фильтр с помощью `-tweb` и выполняет поиск по
шаблону `title`. Если вы запустили:

```
$ rg --type-add 'web:*.{html,css,js}' --type-list
```

Тогда вы увидите, что ваш тип `web` отображается в списке, даже если он не является частью
встроенных типов ripgrep.

Здесь важно подчеркнуть, что флаг `--type-add` применяется только к текущей команде. Он не
добавляет новый тип файла и не сохраняет его где-либо в постоянной форме. Если вы хотите,
чтобы тип был доступен в каждой команде ripgrep, вам следует либо создать псевдоним оболочки:

```
alias rg="rg --type-add 'web:*.{html,css,js}'"
```

либо добавить `--type-add=web:*.{html,css,js}` в свой конфигурационный файл ripgrep.
([Файлы конфигурации](#файлы-конфигурации) будут рассмотрены более подробно позже.)

#### Специальный тип файла `all`

Специальным параметром, поддерживаемым флагом `--type`, является `all`. `--type all`
выполняет поиск в любом из поддерживаемых типов файлов, перечисленных в `--type-list`,
включая те, которые были добавлены в командной строке с помощью `--type-add`. Это эквивалентно
команде `rg -type agda -type asciidoc -type asm ...`, где `...`
означает список флагов `--type` для остальных типов в `--type-list`.

В качестве примера давайте предположим, что у вас есть bash скрипт в вашем текущем каталоге
`my-shell-script`, который включает в себя библиотеку `my-shell-library.bash`.
И `rg -type sh`, и `rg -type all` будут искать совпадения только в
`my-shell-library.bash`, а не в `my-shell-script`, потому
что glob, соответствующие типу файла `sh`, не включают файлы без расширения. С
другой стороны, `rg --type-not all` будет искать `my-shell-script`, но не
`my-shell-library.bash`.

### Замены

ripgrep предоставляет ограниченную возможность изменять свои выходные данные, заменяя
соответствующий текст каким-либо другим текстом. Это проще всего объяснить на примере.
Помните, когда мы искали слово `fast` в README ripgrep?

```
$ rg fast README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast while
119:### Is it really faster than everything else?
124:Summarizing, `ripgrep` is fast because:
129:  optimizations to make searching very fast.
```

Что, если бы мы захотели *заменить* все слова `fast` на `FAST`? Это
легко сделать с помощью флага `--replace` в ripgrep:

```
$ rg fast README.md --replace FAST
75:  FASTer than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays FAST while
119:### Is it really FASTer than everything else?
124:Summarizing, `ripgrep` is FAST because:
129:  optimizations to make searching very FAST.
```

Или, более кратко:

```
$ rg fast README.md -r FAST
[snip]
```

В общем, флаг `--replace` применяется *только* к соответствующей части текста
в выходных данных. Если вместо этого вы хотите заменить целую строку текста, то
вам нужно включить всю строку в ваш шаблон. Например:

```
$ rg '^.*fast.*$' README.md -r FAST
75:FAST
88:FAST
119:FAST
124:FAST
129:FAST
```

Альтернативно, вы можете комбинировать `--only-matching` (или сокращенно `-o`) с
флагом `--replace` для достижения того же результата:

```
$ rg fast README.md --only-matching --replace FAST
75:FAST
88:FAST
119:FAST
124:FAST
129:FAST
```

Или, более кратко:

```
$ rg fast README.md -or FAST
[snip]
```

Наконец, замены могут включать в себя группы. Например, предположим, что
мы хотим найти все слова `fast`, за которыми следует другое слово, и
соединить их при помощи тире. Шаблон, который мы могли бы использовать для этого, -
`fast\s+(\w+)`, который соответствует слову `fast`, за которым следует любое количество пробелов,
а за ними любое количество символов "слова". Мы помещаем `\w+` в "группу захвата" (обозначенную круглыми скобками), чтобы мы могли ссылаться на нее позже в нашей заменяющей строке. Например:

```
$ rg 'fast\s+(\w+)' README.md -r 'fast-$1'
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast-while
124:Summarizing, `ripgrep` is fast-because:
```

Наша строка замены, `fast-$1`, состоит из `fast-`, за которой следует
содержимое группы захвата с индексом `1`. (Группы захвата на самом деле начинаются
с индекса 0, но `0`-я группа захвата всегда соответствует всему
совпадению. Группа захвата с индексом `1` всегда соответствует первой
явной группе захвата, найденной в шаблоне регулярных выражений.)

Группы захвата также могут быть названы, что иногда удобнее, чем
использовать индексы. Например, следующая команда эквивалентна приведенной
выше:

```
$ rg 'fast\s+(?P<word>\w+)' README.md -r 'fast-$word'
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast-while
124:Summarizing, `ripgrep` is fast-because:
```

Важно отметить, что ripgrep **никогда не будет изменять ваши файлы**. То есть
флаг `--replace` управляет только выводом ripgrep. (И нет флага, позволяющего выполнять
замену в самом файле.)


### Конфигурационный файл

Возможно, что параметры ripgrep по умолчанию не будут подходить во всех случаях.
По этой причине, а также потому, что псевдонимы командной строки не всегда удобны, ripgrep
поддерживает файлы конфигурации.

Настроить конфигурационный файл очень просто. Программа ripgrep не будет автоматически искать
конфигурационный файл в каком-либо заданном каталоге. Вместо этого вам нужно
задать переменную среды `RIPGREP_CONFIG_PATH` как путь к вашему конфигурационному файлу.
Как только переменная среды будет задана, откройте файл и просто введите
флаги, которые вы хотите установить автоматически. Существует только два правила для
описания файла конфигурации:

1. Каждая строка является аргументом оболочки после удаления пробелов.
2. Строки, начинающиеся с `#` (необязательно с любым количеством пробелов)
игнорируются.

В частности, нет возможности экранирования. Каждая строка передается в ripgrep в виде отдельного
аргумента командной строки дословно.

Вот пример конфигурационного файла, который демонстрирует некоторые особенности
форматирования:

```
$ cat $HOME/.ripgreprc
# Don't let ripgrep vomit really long lines to my terminal, and show a preview.
--max-columns=150
--max-columns-preview

# Add my 'web' type.
--type-add
web:*.{html,css,js}*

# Search hidden files / directories (e.g. dotfiles) by default
--hidden

# Using glob patterns to include/exclude files or folders
--glob=!.git/*

# or
--glob
!.git/*

# Set the colors.
--colors=line:none
--colors=line:style:bold

# Because who cares about case!?
--smart-case
```

Когда мы используем флаг, у которого есть значение, мы либо помещаем флаг и значение в
одну строку, но разделяем их знаком `=` (например, `--max-columns=150`), либо
помещаем флаг и значение в две разные строки. Это связано с тем, что анализатор
аргументов ripgrep знает, что единственный аргумент `--max-columns=150` должен обрабатываться
как флаг со значением, но если бы мы написали `--max-columns 150` в нашем
конфигурационном файле, то анализатор аргументов ripgrep не знал бы, что с ним делать.

Размещение флага и значения в разных строках в точности эквивалентно и
зависит от стиля.

Рекомендуется использовать комментарии, чтобы вы помнили, что делает конфигурация. Пустые
строки тоже допустимы.

Итак, предположим, вы используете приведенный выше конфигурационный файл, но, находясь в
терминале, вы хотите видеть строки длиной более 150 столбцов. Что делать? К счастью,
достаточно ввести `--max-columns 0` (или сокращенно `-M0`) в командной строке,
что переопределит настройки вашего конфигурационного файла. Это работает, потому что файл
конфигурации ripgrep имеет *предварительное значение* перед явными аргументами, которые вы
указываете в командной строке. Поскольку флаги, заданные позже, переопределяют флаги, заданные
ранее. Это работает для большинства других флагов, и в документации к каждому флагу указано,
какие другие флаги переопределяют его.

Если вы не уверены, из какого конфигурационного файла ripgrep считывает аргументы, то запуск 
ripgrep с флагом `--debug` должен помочь вам разобраться. В данных отладки должно
быть указано, какой конфигурационный файл загружается, и какие аргументы были считаны из
конфигурации.

Наконец, если вы хотите быть абсолютно уверены, что ripgrep  *не читает* файл конфигурации, то
вы можете передать флаг `--no-config`, который исключит использование конфигурационного файла,
независимо от того, какие другие методы настройки будут добавлены в ripgrep в будущем.

### Кодировка файлов

[Кодировка текста](https://ru.wikipedia.org/wiki/Character_encoding) это сложная тема, но мы
можем попытаться обобщить её значимость для ripgrep:

* Файлы, как правило, представляют собой просто набор байтов. Надежного способа узнать
  их кодировку не существует.
* Либо кодировка шаблона должна совпадать с кодировкой файлов, в которых выполняется
  поиск, либо должна быть выполнена какая-либо форма перекодирования, которая преобразует либо
  шаблон, либо файл в ту же кодировку, что и другой файл.
* ripgrep, как правило, лучше всего работает с обычными текстовыми файлами, а среди обычных
  текстовых файлов наиболее популярными кодировками, являются ASCII, latin1 или UTF-8.
  В качестве особого исключения в среде Windows преобладает UTF-16.

Учитывая эту информацию, вот как ведет себя ripgrep, когда задан флаг `--encoding auto`,
которое используется по умолчанию:

* Предполагается, что все вводимые данные совместимы с ASCII (что означает, что каждый байт,
  соответствующий ASCII-коду, на самом деле является ASCII-кодом). Это включает в себя сам ASCII,
  latin1 и UTF-8.
* ripgrep лучше всего работает с кодировкой UTF-8. Например, движок регулярных выражений
  ripgrep поддерживает функции Unicode. А именно, классы символов, такие как `\w`, будут
  соответствует всем символам слов в соответствии с определением Unicode, и `.` будет
  соответствовать любой кодовой точке Unicode вместо любого байта. В этих конструкциях
  используется UTF-8, поэтому они просто не будут совпадать, если в файле встретятся байты,
  которые не соответствуют UTF-8.
* Чтобы обработать случай с UTF-16, ripgrep выполнит так называемый "анализ спецификации".
  по умолчанию. То есть будут считаны первые три байта файла, и если они соответствуют
  спецификации UTF-16, то ripgrep перекодирует содержимое файла из UTF-16 в UTF-8, а затем
  выполнит поиск по перекодированной версии файла. (Это приводит к снижению
  производительности, поскольку в дополнение к поиску в регулярных выражениях требуется
  перекодирование.) Если файл содержит недопустимый UTF-16, то вместо недопустимых кодовых
  единиц заменяется кодовая точка Unicode.
* Для обработки других случаев ripgrep предоставляет флаг `-E/--encoding`, который позволяет
  вам указать кодировку из [Стандарта кодирования](https://encoding.spec.whatwg.org/#concept-encoding-get). ripgrep будет считать, что все файлы *поиска* соответствуют указанной 
  кодировке (если только файл имеет спецификацию) и выполнит этап перекодирования точно так же,
  как в случае с UTF-16, описанном выше.

По умолчанию ripgrep не требует, чтобы его входные данные были в формате UTF-8. То есть ripgrep
может выполнять и будет выполнять поиск в произвольных байтах. Ключевым моментом здесь является
то, что если вы будете искать содержимое, которое не соответствует UTF-8, то полезность вашего
шаблона снизится. Если вы будете искать байты, которые не совместимы с ASCII, то, скорее
всего, шаблон ничего не найдет. Учитывая все вышесказанное, этот режим работы важен, поскольку
он позволяет вам находить ASCII или UTF-8 *внутри* файлов, которые в остальном представляют 
собой произвольные байты.

В качестве особого случая флаг `-E/--encoding` поддерживает значение `none`, которое полностью
отключает всю логику, связанную с кодированием, включая анализ спецификации. Если для параметра
`-E/--encoding` задано значение `none`, ripgrep выполнит поиск в необработанных байтах базового
файла без этапа перекодирования. Например, вот как вы могли бы выполнить поиск в исходной
кодировке UTF-16 строки `Шерлок`:

```
$ rg '(?-u)\(\x045\x04@\x04;\x04>\x04:\x04' -E none -a some-utf16-file
```

Of course, that's just an example meant to show how one can drop down into
raw bytes. Namely, the simpler command works as you might expect automatically:
Конечно, это всего лишь пример, призванный показать, как можно использовать
необработанные байты. А именно, поиск можно выполнить более простым образом, автоматически:

```
$ rg 'Шерлок' some-utf16-file
```

Наконец, можно отключить поддержку Unicode в ripgrep с помощью регулярного выражения.
Например, предположим, что вы хотите, чтобы `.` соответствовал любому байту , а не какой-либо
кодовой точке Unicode. (Это может потребоваться при поиске в бинарном файле, поскольку `.` 
по умолчанию не соответствует недопустимому UTF-8.) Вы могли бы сделать это, отключив Unicode
с помощью флага регулярного выражения:

```
$ rg '(?-u:.)'
```

Это работает для любой части шаблона. Например, в приведенном ниже примере вы найдете
любой символ слова в Unicode, за которым следует любой символ слова в ASCII, за которым следует
другой символ слова в Unicode:

```
$ rg '\w(?-u:\w)\w'
```


### Binary data

In addition to skipping hidden files and files in your `.gitignore` by default,
ripgrep also attempts to skip binary files. ripgrep does this by default
because binary files (like PDFs or images) are typically not things you want to
search when searching for regex matches. Moreover, if content in a binary file
did match, then it's possible for undesirable binary data to be printed to your
terminal and wreak havoc.

Unfortunately, unlike skipping hidden files and respecting your `.gitignore`
rules, a file cannot as easily be classified as binary. In order to figure out
whether a file is binary, the most effective heuristic that balances
correctness with performance is to simply look for `NUL` bytes. At that point,
the determination is simple: a file is considered "binary" if and only if it
contains a `NUL` byte somewhere in its contents.

The issue is that while most binary files will have a `NUL` byte toward the
beginning of its contents, this is not necessarily true. The `NUL` byte might
be the very last byte in a large file, but that file is still considered
binary. While this leads to a fair amount of complexity inside ripgrep's
implementation, it also results in some unintuitive user experiences.

At a high level, ripgrep operates in three different modes with respect to
binary files:

1. The default mode is to attempt to remove binary files from a search
   completely. This is meant to mirror how ripgrep removes hidden files and
   files in your `.gitignore` automatically. That is, as soon as a file is
   detected as binary, searching stops. If a match was already printed (because
   it was detected long before a `NUL` byte), then ripgrep will print a warning
   message indicating that the search stopped prematurely. This default mode
   **only applies to files searched by ripgrep as a result of recursive
   directory traversal**, which is consistent with ripgrep's other automatic
   filtering. For example, `rg foo .file` will search `.file` even though it
   is hidden. Similarly, `rg foo binary-file` will search `binary-file` in
   "binary" mode automatically.
2. Binary mode is similar to the default mode, except it will not always
   stop searching after it sees a `NUL` byte. Namely, in this mode, ripgrep
   will continue searching a file that is known to be binary until the first
   of two conditions is met: 1) the end of the file has been reached or 2) a
   match is or has been seen. This means that in binary mode, if ripgrep
   reports no matches, then there are no matches in the file. When a match does
   occur, ripgrep prints a message similar to one it prints when in its default
   mode indicating that the search has stopped prematurely. This mode can be
   forcefully enabled for all files with the `--binary` flag. The purpose of
   binary mode is to provide a way to discover matches in all files, but to
   avoid having binary data dumped into your terminal.
3. Text mode completely disables all binary detection and searches all files
   as if they were text. This is useful when searching a file that is
   predominantly text but contains a `NUL` byte, or if you are specifically
   trying to search binary data. This mode can be enabled with the `-a/--text`
   flag. Note that when using this mode on very large binary files, it is
   possible for ripgrep to use a lot of memory.

Unfortunately, there is one additional complexity in ripgrep that can make it
difficult to reason about binary files. That is, the way binary detection works
depends on the way that ripgrep searches your files. Specifically:

* When ripgrep uses memory maps, then binary detection is only performed on the
  first few kilobytes of the file in addition to every matching line.
* When ripgrep doesn't use memory maps, then binary detection is performed on
  all bytes searched.

This means that whether a file is detected as binary or not can change based
on the internal search strategy used by ripgrep. If you prefer to keep
ripgrep's binary file detection consistent, then you can disable memory maps
via the `--no-mmap` flag. (The cost will be a small performance regression when
searching very large files on some platforms.)


### Preprocessor

In ripgrep, a preprocessor is any type of command that can be run to transform
the input of every file before ripgrep searches it. This makes it possible to
search virtually any kind of content that can be automatically converted to
text without having to teach ripgrep how to read said content.

One common example is searching PDFs. PDFs are first and foremost meant to be
displayed to users. But PDFs often have text streams in them that can be useful
to search. In our case, we want to search Bruce Watson's excellent
dissertation,
[Taxonomies and Toolkits of Regular Language Algorithms](https://burntsushi.net/stuff/1995-watson.pdf).
After downloading it, let's try searching it:

```
$ rg 'The Commentz-Walter algorithm' 1995-watson.pdf
$
```

Surely, a dissertation on regular language algorithms would mention
Commentz-Walter. Indeed it does, but our search isn't picking it up because
PDFs are a binary format, and the text shown in the PDF may not be encoded as
simple contiguous UTF-8. Namely, even passing the `-a/--text` flag to ripgrep
will not make our search work.

One way to fix this is to convert the PDF to plain text first. This won't work
well for all PDFs, but does great in a lot of cases. (Note that the tool we
use, `pdftotext`, is part of the [poppler](https://poppler.freedesktop.org)
PDF rendering library.)

```
$ pdftotext 1995-watson.pdf > 1995-watson.txt
$ rg 'The Commentz-Walter algorithm' 1995-watson.txt
316:The Commentz-Walter algorithms : : : : : : : : : : : : : : :
7165:4.4 The Commentz-Walter algorithms
10062:in input string S , we obtain the Boyer-Moore algorithm. The Commentz-Walter algorithm
17218:The Commentz-Walter algorithm (and its variants) displayed more interesting behaviour,
17249:Aho-Corasick algorithms are used extensively. The Commentz-Walter algorithms are used
17297: The Commentz-Walter algorithms (CW). In all versions of the CW algorithms, a common program skeleton is used with di erent shift functions. The CW algorithms are
```

But having to explicitly convert every file can be a pain, especially when you
have a directory full of PDF files. Instead, we can use ripgrep's preprocessor
feature to search the PDF. ripgrep's `--pre` flag works by taking a single
command name and then executing that command for every file that it searches.
ripgrep passes the file path as the first and only argument to the command and
also sends the contents of the file to stdin. So let's write a simple shell
script that wraps `pdftotext` in a way that conforms to this interface:

```
$ cat preprocess
#!/bin/sh

exec pdftotext - -
```

With `preprocess` in the same directory as `1995-watson.pdf`, we can now use it
to search the PDF:

```
$ rg --pre ./preprocess 'The Commentz-Walter algorithm' 1995-watson.pdf
316:The Commentz-Walter algorithms : : : : : : : : : : : : : : :
7165:4.4 The Commentz-Walter algorithms
10062:in input string S , we obtain the Boyer-Moore algorithm. The Commentz-Walter algorithm
17218:The Commentz-Walter algorithm (and its variants) displayed more interesting behaviour,
17249:Aho-Corasick algorithms are used extensively. The Commentz-Walter algorithms are used
17297: The Commentz-Walter algorithms (CW). In all versions of the CW algorithms, a common program skeleton is used with di erent shift functions. The CW algorithms are
```

Note that `preprocess` must be resolvable to a command that ripgrep can read.
The simplest way to do this is to put your preprocessor command in a directory
that is in your `PATH` (or equivalent), or otherwise use an absolute path.

As a bonus, this turns out to be quite a bit faster than other specialized PDF
grepping tools:

```
$ time rg --pre ./preprocess 'The Commentz-Walter algorithm' 1995-watson.pdf -c
6

real    0.697
user    0.684
sys     0.007
maxmem  16 MB
faults  0

$ time pdfgrep 'The Commentz-Walter algorithm' 1995-watson.pdf -c
6

real    1.336
user    1.310
sys     0.023
maxmem  16 MB
faults  0
```

If you wind up needing to search a lot of PDFs, then ripgrep's parallelism can
make the speed difference even greater.

#### A more robust preprocessor

One of the problems with the aforementioned preprocessor is that it will fail
if you try to search a file that isn't a PDF:

```
$ echo foo > not-a-pdf
$ rg --pre ./preprocess 'The Commentz-Walter algorithm' not-a-pdf
not-a-pdf: preprocessor command failed: '"./preprocess" "not-a-pdf"':
-------------------------------------------------------------------------------
Syntax Warning: May not be a PDF file (continuing anyway)
Syntax Error: Couldn't find trailer dictionary
Syntax Error: Couldn't find trailer dictionary
Syntax Error: Couldn't read xref table
```

To fix this, we can make our preprocessor script a bit more robust by only
running `pdftotext` when we think the input is a non-empty PDF:

```
$ cat preprocessor
#!/bin/sh

case "$1" in
*.pdf)
  # The -s flag ensures that the file is non-empty.
  if [ -s "$1" ]; then
    exec pdftotext - -
  else
    exec cat
  fi
  ;;
*)
  exec cat
  ;;
esac
```

We can even extend our preprocessor to search other kinds of files. Sometimes
we don't always know the file type from the file name, so we can use the `file`
utility to "sniff" the type of the file based on its contents:

```
$ cat processor
#!/bin/sh

case "$1" in
*.pdf)
  # The -s flag ensures that the file is non-empty.
  if [ -s "$1" ]; then
    exec pdftotext - -
  else
    exec cat
  fi
  ;;
*)
  case $(file "$1") in
  *Zstandard*)
    exec pzstd -cdq
    ;;
  *)
    exec cat
    ;;
  esac
  ;;
esac
```

#### Reducing preprocessor overhead

There is one more problem with the above approach: it requires running a
preprocessor for every single file that ripgrep searches. If every file needs
a preprocessor, then this is OK. But if most don't, then this can substantially
slow down searches because of the overhead of launching new processors. You
can avoid this by telling ripgrep to only invoke the preprocessor when the file
path matches a glob. For example, consider the performance difference even when
searching a repository as small as ripgrep's:

```
$ time rg --pre pre-rg 'fn is_empty' -c
crates/globset/src/lib.rs:1
crates/matcher/src/lib.rs:2
crates/ignore/src/overrides.rs:1
crates/ignore/src/gitignore.rs:1
crates/ignore/src/types.rs:1

real    0.138
user    0.485
sys     0.209
maxmem  7 MB
faults  0

$ time rg --pre pre-rg --pre-glob '*.pdf' 'fn is_empty' -c
crates/globset/src/lib.rs:1
crates/ignore/src/types.rs:1
crates/ignore/src/gitignore.rs:1
crates/ignore/src/overrides.rs:1
crates/matcher/src/lib.rs:2

real    0.008
user    0.010
sys     0.002
maxmem  7 MB
faults  0
```


### Common options

ripgrep has a lot of flags. Too many to keep in your head at once. This section
is intended to give you a sampling of some of the most important and frequently
used options that will likely impact how you use ripgrep on a regular basis.

* `-h`: Show ripgrep's condensed help output.
* `--help`: Show ripgrep's longer form help output. (Nearly what you'd find in
  ripgrep's man page, so pipe it into a pager!)
* `-i/--ignore-case`: When searching for a pattern, ignore case differences.
  That is `rg -i fast` matches `fast`, `fASt`, `FAST`, etc.
* `-S/--smart-case`: This is similar to `--ignore-case`, but disables itself
  if the pattern contains any uppercase letters. Usually this flag is put into
  alias or a config file.
* `-F/--fixed-strings`: Disable regular expression matching and treat the pattern
   as a literal string.
* `-w/--word-regexp`: Require that all matches of the pattern be surrounded
  by word boundaries. That is, given `pattern`, the `--word-regexp` flag will
  cause ripgrep to behave as if `pattern` were actually `\b(?:pattern)\b`.
* `-c/--count`: Report a count of total matched lines.
* `--files`: Print the files that ripgrep *would* search, but don't actually
  search them.
* `-a/--text`: Search binary files as if they were plain text.
* `-U/--multiline`: Permit matches to span multiple lines.
* `-z/--search-zip`: Search compressed files (gzip, bzip2, lzma, xz, lz4,
  brotli, zstd). This is disabled by default.
* `-C/--context`: Show the lines surrounding a match.
* `--sort path`: Force ripgrep to sort its output by file name. (This disables
  parallelism, so it might be slower.)
* `-L/--follow`: Follow symbolic links while recursively searching.
* `-M/--max-columns`: Limit the length of lines printed by ripgrep.
* `--debug`: Shows ripgrep's debug output. This is useful for understanding
  why a particular file might be ignored from search, or what kinds of
  configuration ripgrep is loading from the environment.
