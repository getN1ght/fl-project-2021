# fl-project-2021 #

1. [Boost-spirit](#Boost-Spirit)
2. [Parsec](#Parsec)
3. [Pyparsing](#Pyparsing)

## Boost-Spirit

Используются зависимости:

* boost/spirit
* boost/fusion
* boost/variant

Компиляция проекта происходит при помощи ввода следующей команды:

``` bash
   g++ main.cpp -o src 
```

При запуске программа ожидает путь до файла, написанному на языке
из [description.txt](/description.txt). На выход выдается построенное
AST в случае, если программа принадлежит данному языку
или сообщение об ошибке в обратном случае.

Из особенностей

* Свой лексер и интеграция с ним (так и не воспользовался)
* Интеграция с библиотекой boost::fusion, позволяющая делать различные
  функциональные штуки
* Очень удобная перегрузка операторов для правил

## Тестирование ##

Первые 11 тестов должны показывать, что парсер распознает
языковые особенности, последние два - стрессы. На тесты большего
порядка просто не хватает стека, но и с такими время выполнения
уже достаточно большое. В частности,

```bash
Test #13
real    0m2,653s
user    0m2,442s
sys     0m0,052s
```

```bash
Test #14
real    0m0,618s
user    0m0,601s
sys     0m0,016s
```

Разница в размере входных данных примерно в два раза, но время
отличается так из-за количества откатов. Так как парсер
внутри Boost::Spirit использует алгоритм нисходящего рекурсивного
спуска, то в худшем случае может потребоваться экспоненциальное время.
Так же серьезным фактором, влияющем на время, является объем выходных
данных. Для теста `#13`  он составляет 42 мегабайта, что уже может
в значительной мере повлиять на время выполнения.

При ошибке парсер возвращает фрагмент входного файла, начиная
с первой не распознанной функции.

## Parsec ##

1. Библиотекa монадических парсер комбинаторов `parsec`, библиотека с хорошей документацией,
   большим количеством примеров.
2. Библиотека достаточно удобная, например есть встроенный парсер `buildExpressionParser`,
   который сильно облегчает парсинг языка, синтаксис очень интуитивный.
3. Обработка ошибок. Встроенная обработка ошибок в `parsec` уже имеет хорошую форму. Сообщение выглядит так:
   строка, символ, какой символ ожидался на самом деле. Пользователю эта информация может быть достаточно полезна, для того чтобы понять, как исправить код. С возможностью кастомизировать сообщения об ошибках я не разбирался.
4. Сложности и минусы `parsec`. Внутри `parsec` работает как LL(1) парсер, что конечно неплохо, но слабее, чем   парсеры из бизона или других парсер-генераторов. Из-за этого, например, в случае `statement = expression`
   `expression` не может начинаться с маленькой латинской буквы, так как там возникает неоднозначность - либо ожидается `assignment` либо `expression`. В таком случае нужно обернуть `expression` в скобки, чтобы все разбиралось однозначно. Помимо этого в остальных местах где были неоднозначности надо было аккуратно задать грамматику.
5. Время работы на тестах -

```bash
Test #18
real    0,968s
```

```bash
Test #19
real    7,139s
```

```bash
Test #20
real    0,171s
```

   Большие тесты состоят в основном только из выражений, так как вся сложность парсинга приходится именно на выажения, так как все `statement` имеют достаточно простую структуру. Для того чтобы запустить тесты достаточно запустить функцию `checkFile path/to/test`.

## Pyparsing ##

### Сборка ###

   1. `pip3 install pyparsing`
   2. `python3 parser.py`

### Информация о парсер-комбинаторе `pyparsing` ###

   1. Это парсер-комбинатор, который основан на `PEG`, а не `CFG`. Преимущество заключается в том, что `PEG` лучше умеет обрабатвать `choice operator` (то есть, по сути, некоторые неоднозначности грамматики)

   2. Плюсы библиотеки:
      - Достаточно сильно можно кастомизировать процесс парсинга, передавать свои обработчики токенов, заматченных отдельными правилами
      - Наличие неплохой документации и примеров, по которым можно разбираться с библиотекой
      
      Минусы библиотеки:
      - Довольно слабая декларативность при описании грамматики
      - Неудобная обработка ошибок, отсутствие качественной кастомизации в их обработке
      - По умолчанию ошибки, которые возникают при парсинге, не дают практически никакой информации о том, что именно не удалось распарсить 
      - Она на питоне...

### Как читать AST ###

Практически у всех узлов AST есть отдельные типы. Они будут выводиться. Также, если тип проще, чем pllaceholder, то будет выведено также содержимое соответствующего объекта (поля структуры)

Пример того, как выглядит описание узла AST типа `Expression` (`2 + 2 * 7 && 1 || 1 >= 0`) в конcоли
```
IntegerLiteral  2

IntegerLiteral  2

IntegerLiteral  7

EvalMultOp [<__main__.IntegerLiteral object at 0x7fc6152b2df0>, '*', <__main__.IntegerLiteral object at 0x7fc615398670>]

EvalAddOp [<__main__.IntegerLiteral object at 0x7fc615392fd0>, '+', <__main__.EvalMultOp object at 0x7fc6153982e0>]

IntegerLiteral  1

EvalAndOp [<__main__.EvalAddOp object at 0x7fc615398c70>, '&&', <__main__.IntegerLiteral object at 0x7fc6152bd1f0>]

IntegerLiteral  1

IntegerLiteral  0

EvalComparisonOp [<__main__.IntegerLiteral object at 0x7fc6152b2910>, '>=', <__main__.IntegerLiteral object at 0x7fc6152bd790>]

EvalOrOp [<__main__.EvalAndOp object at 0x7fc6152b2fd0>, '||', <__main__.EvalComparisonOp object at 0x7fc615398100>]
```

`IntegerLiteral` -- специальный узел AST, который хранит целочисленные литералы.

`EvalMultOp` -- специальный узел AST, который хранит операнды, над которыми производится умножение, а также хранит типы операций (подразумевается, что тут хранятся и деления, и умножения)

`EvalAddOp`, `EvalOrOp` -- аналогично

`EvalComparisonOp` -- на самом деле, работает очень похожим образом на EvalAddOp и EvalMulOp, хранит операнды и типы сравнений, которые были использованы

Общая логика такова, что слева указан тип ноды AST, а справа её содержимое

### Время работы ###

Я запускал на тех же тестах, что Серёжа и Артём. Все результаты были ожидаемые, однако не работают несколько тестов (`#18`, `#19`), `python` не хватает стека рекурсии. Самое долгое время на оставшихся тестах -- это 19 секунд, на тесте `#20`.

```bash
Test #20
real    0m18,110s
user    0m17,708s
sys     0m0,060s
```

#### PS ###

Изначально мной была предпринята попытка написать парсер на библиотеке `lexy` на `C++`.
Однако проблема заключалась в том, что там отсутствовала нормальная документация и примеры. Помимо этого по умолчанию он был очень плохо заточен под парсинг арифметических выражений. В итоге, спустя полтора дня страданий, было принято решение поменять выбор библиотеки и языка. Однако из плюсов хочется отметить обработку ошибок там. Она прекрасна.

## Сравнение библиотек ##

№ | Spirit | Parsec | PyParsing
--- | --- | --- | ---
13 | 2,653s | 0,006s | 0m0,991s
14 | 0,618s | 0,013s | 0m0,434s
18 | Stack Overflow | 0,968s | Stack Overflow
19 | Stack Overflow | 7,139s | Stack Overflow
20 | Stack Overflow | 0,171s | 18,110s

То, что `Parsec` работает быстрее обусловлено тем,
что внутри библиотеки используется алгоритм `LL(1)`,
а внутри `Spirit` и `PyParsing` - обычный рекурсивный спуск.

Обработка ошибок в `Parsec` реализована достаточно хорошо по умолчанию,
в `Boost` есть функционал для ее рализации, но он не самый удобный
(в реализации выше, например, реализован просто вывод не
распаршенной части файла). В `PyParsing` ситуация получше,
он иногда выводит подробное сообщение об ошибке по умолчанию,
но не всегд можно по нему понять, что произошло.

В плане удобства использования библиотечных функций
самым удобным оказался `Parsec`, так как он по умолчанию
умеет парсить выражения и имеет интуитивный синтаксис. Да и как
для парсинга чего-либо функциональные языки подходят лучше.

Во время использования `Spirit`'а  было только такое ощущение:

![avatar](https://www.meme-arsenal.com/memes/95c076adbf58cf87ca912ac51d1726e9.jpg)

Из хорошего - интеграция с библиотеками от `Boost`,
например - `Boost/Fusion`. Эта библиотека, например, имитирует
поведение функциональных языков, однако она точно не сравнится
с Haskell и подобными. Также есть описание грамматики
очень схоже к описанием регулярных выражений.

Про `PyParsing`.
Из-за динамической типизации в `Python` не нужно
описывать структуры-контейнеры, что сильно сокращает количество кода.

Еще одно важное отличие - `Haskell` хватает размера стека,
чтобы распарсить программы больших размеров, в то время как
оставшимся языкам - нет.

По итогу Питон победил Плюсы
![avatar](https://avatars.mds.yandex.net/get-zen_doc/1594475/pub_5e690687327b3f744e0036c4_5e6907b2ca551a68df05024f/scale_1200)
