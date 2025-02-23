%{
  version: "1.2.0",
  title: "Строки",
  excerpt: """
  Строки, списки символов, графемы и коды символов.
  """
}
---

## Строки

Строки в Elixir &mdash; это не что иное, как последовательность байтов.
Взглянем на пример:

```elixir
iex> string = <<104,101,108,108,111>>
"hello"
iex> string <> <<0>>
<<104, 101, 108, 108, 111, 0>>
```

После объединения строки с байтом `0`, IEx отображает строку в двоичном виде, так как она перестает быть валидной.
Этот трюк может помочь в просмотре байтов любой строки.

>ПРИМЕЧАНИЕ: Используя << >>, мы сообщаем компилятору, что элементы внутри этих символов &mdash; это байты.

## Списки символов

В языке Elixir есть списки символов, однако строки представлены в виде последовательности байтов. Строки в Elixir заключаются в двойные кавычки, а списки символов &mdash; в одинарные.

Но в чём разница? Каждое значение в списке символов &mdash; это Unicode код символа, тогда как в двоичном виде они кодируются как UTF-8.
Рассмотрим поглубже:

```elixir
iex> 'hełło'
[104, 101, 322, 322, 111]
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

`322` является Unicode кодом символа ł, но он кодируется в UTF-8 как два байта `197`, `130`.

Вы можете получить код символа используя `?`

```elixir
iex> ?Z  
90
```

Это позволяет использовать обозначение `?Z` вместо символа 'Z'.

Программируя на Elixir, мы обычно используем строки, а не списки символов. Поддержка списков символов включена, в основном, для совместимости с некоторыми модулями Erlang.

Для получения дополнительной информации, читайте официальную документацию [`Getting Started Guide`](http://elixir-lang.org/getting-started/binaries-strings-and-char-lists.html).

## Графемы и коды символов

Unicode код &mdash; это символ Unicode, который представлен одним или более байтами в зависимости от кодировки UTF-8. Символы, не входящие в набор US ASCII всегда кодируются более, чем одним байтом. Например, латинские символы с тильдой или акцентами: `á, ñ, è` обычно закодированы двумя байтами. Символы из азиатских языков, как правило, закодированы тремя-четырьмя байтами. Графема состоит из нескольких Unicode кодов и выглядит как один символ.

Модуль строки предоставляет две функции для их получения: `graphemes/1` и `codepoints/1`. Взглянем на пример:

```elixir
iex> string = "\u0061\u0301"
"á"

iex> String.codepoints string
["a", "́"]

iex> String.graphemes string
["á"]
```

## Строковые функции

Рассмотрим некоторые самые важные и полезные функции модуля строки. Этот урок покрывает лишь часть доступных функций. Чтобы ознакомиться с полным списком, можно посетить [страницу](https://hexdocs.pm/elixir/String.html) официальной документации.

### length/1

Возвращает количество графем в строке.

```elixir
iex> String.length "Hello"
5
```

### replace/3

Возвращает новую строку, заменив в исходной строке выбранное выражение на некоторую строку замены.

```elixir
iex> String.replace("Hello", "e", "a")
"Hallo"
```

### duplicate/2

Возвращает строку, повторённую n раз.

```elixir
iex> String.duplicate("Oh my ", 3)
"Oh my Oh my Oh my "
```

### split/2

Возвращает список строк, разделённых по выражению.

```elixir
iex> String.split("Hello World", " ")
["Hello", "World"]
```

## Упражнения

Давайте пройдёмся по простым упражнениям, чтобы убедиться, что мы готовы работать со строками.

### Анаграммы

A и B считаются анаграммами, если существует способ перестановки A или B таким образом, чтобы в итоге строки стали равны. Например:

+ A = super
+ B = perus

Если мы изменим порядок символов в A, мы можем получить B, и наоборот.

И как мы можем проверить в Elixir, являются ли две строки анаграммами? Простейшее решение &mdash; отсортировать графемы в каждой строке по алфавиту и проверить, равны ли списки. Давайте попробуем:

```elixir
defmodule Anagram do
  def anagrams?(a, b) when is_binary(a) and is_binary(b) do
    sort_string(a) == sort_string(b)
  end

  def sort_string(string) do
    string
    |> String.downcase()
    |> String.graphemes()
    |> Enum.sort()
  end
end
```

Давайте взглянем на `anagrams?/2`. Мы проверяем, является ли каждый из параметров бинарными данными. Именно таким образом в Elixir можно проверить, что параметр является строкой.

Дальше мы вызываем функцию, сортирующую строку в алфавитном порядке, предварительно переведя строки в нижний регистр и использовав `String.graphemes/1`, чтобы получить список графем в строке. Наконец, он передает этот список в `Enum.sort/1`. Довольно очевидно, не так ли?

Проверим вывод в `iex`:

```elixir
iex> Anagram.anagrams?("Hello", "ohell")
true

iex> Anagram.anagrams?("María", "íMara")
true

iex> Anagram.anagrams?(3, 5)
** (FunctionClauseError) no function clause matching in Anagram.anagrams?/2

    The following arguments were given to Anagram.anagrams?/2:

        # 1
        3

        # 2
        5

    iex:11: Anagram.anagrams?/2
```

Как вы могли заметить, последний вызов `anagrams?` завершился с FunctionClauseError. Эта ошибка говорит нам о том, что в нашем модуле нет функции, которая могла бы принимать два небинарных аргумента. И это именно то, что мы хотим &mdash; просто принимать две строки и ничего больше.
