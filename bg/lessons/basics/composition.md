---
layout: page
title: Композиция
category: basics
order: 8
lang: bg
---

От опит знаем колко неудобно е да държим всички наши функции в един и същи файл и една област на видимост.  В този урок ще разгледаме как да групираме функции и да дефинираме специален асоциативен масив известен като `struct`, за да организираме кода си по-оптимално.

{% include toc.html %}

## Модули

Модулите са най-добрият начин да организираме функциите си в области на видимост.  Освен да групираме функции, те ни позволяват да дефинираме именовани и частни функции които покрихме в предишен урок.

Нека разгледаме един прост пример:

``` elixir
defmodule Example do
  def greeting(name) do
    ~s(Hello #{name}.)
  end
end

iex> Example.greeting "Sean"
"Hello Sean."
```

Възможно е да вграждаме модули в Elixir, което ни позволява да използваме допълнителни наименовани пространства:

```elixir
defmodule Example.Greetings do
  def morning(name) do
    "Good morning #{name}."
  end

  def evening(name) do
    "Good night #{name}."
  end
end

iex> Example.Greetings.morning "Sean"
"Good morning Sean."
```

### Атрибути на модули

Атрибутите на модули най-често се използват като константи в.  Нека разгледаме един прост пример:

```elixir
defmodule Example do
  @greeting "Hello"

  def greeting(name) do
    ~s(#{@greeting} #{name}.)
  end
end
```

Важно е да се отбележи, че има запазени атрибути в Elixir.  Трите най-често срещани са:

+ `moduledoc` — Документира текущия модул.
+ `doc` — Документация за функции и макроси.
+ `behaviour` — Използва OTP определено от потребителя поведение.

## Структури

Структурите са специални асоциативни масиви с дефиниран набор от ключове и стойности по подразбиране.  Трябва да бъде дефинира в модул, от който структурата взима името си.  Обичайно е за структура да е единственото нещо дефинирано в модул.

За да дефинираме структура използваме `defstruct` заедно със списък от полета с ключове и стойности по подразбиране:

```elixir
defmodule Example.User do
  defstruct name: "Sean", roles: []
end
```

Нека създадем няколко структури:

```elixir
iex> %Example.User{}
%Example.User{name: "Sean", roles: []}

iex> %Example.User{name: "Steve"}
%Example.User{name: "Steve", roles: []}

iex> %Example.User{name: "Steve", roles: [:admin, :owner]}
%Example.User{name: "Steve", roles: [:admin, :owner]}
```

Можем да обновяваме структурите си както бихме обновили и асоциативен масив:

```elixir
iex> steve = %Example.User{name: "Steve", roles: [:admin, :owner]}
%Example.User{name: "Steve", roles: [:admin, :owner]}
iex> sean = %{steve | name: "Sean"}
%Example.User{name: "Sean", roles: [:admin, :owner]}
```

Най-важното е, че можем да съпоставяме структури спрямо асоциативни масиви:

```elixir
iex> %{name: "Sean"} = sean
%Example.User{name: "Sean", roles: [:admin, :owner]}
```

## Композиция

Сега след като знаем как да създаваме модули и структури, нека се научим как да внедрим съществуващата функционалност в тях посредством композиция.  Elixir ни предлага множество различни начини за взаимодействие с други модули, нека разгледаме какво имаме налично.

### `alias`

Позволява ни да използваме съкращение за имена на модули, използва се много често в Elixir код:

```elixir
defmodule Sayings.Greetings do
  def basic(name), do: "Hi, #{name}"
end

defmodule Example do
  alias Sayings.Greetings

  def greeting(name), do: Greetings.basic(name)
end

# Без alias

defmodule Example do
  def greeting(name), do: Saying.Greetings.basic(name)
end
```

Ако има конфликт с две съкращения или просто желаете да съкратите като съвсем различно име, моем да използваме опцията `:as` опция:

```elixir
defmodule Example do
  alias Sayings.Greetings, as: Hi

  def print_message(name), do: Hi.basic(name)
end
```

Възможно е да съкратим няколко модула на веднъж:

```elixir
defmodule Example do
  alias Sayings.{Greetings, Farewells}
end
```

### `import`

Ако искаме да внесем функции и макроси вместо да съкращаваме може да използваме `import/`:

```elixir
iex> last([1, 2, 3])
** (CompileError) iex:9: undefined function last/1
iex> import List
nil
iex> last([1, 2, 3])
3
```

#### Филтриране

По подразбиране всички функции и макроси са внесени в областта на видимост, но може да ги филтрираме чрез опиите `:only` и `:except`.

За да внесем специфични функции и макроси, трябва да предоставим двойките име/брой на `:only` и `:except`.  Нека започнем като внесем само функцията `last/1`:

```elixir
iex> import List, only: [last: 1]
iex> first([1, 2, 3])
** (CompileError) iex:13: undefined function first/1
iex> last([1, 2, 3])
3
```

Ако внесем всичко освен `last/1` и пробваме същите функции като преди:

```elixir
iex> import List, except: [last: 1]
nil
iex> first([1, 2, 3])
1
iex> last([1, 2, 3])
** (CompileError) iex:3: undefined function last/1
```

В допълнение към двойките име/брой има два специални атома, `:functions` и `:macros`, които внасят само функции и макроси респективно:

```elixir
import List, only: :functions
import List, only: :macros
```

### `require`

Въпреки по рядко използвана, `require/2` е важна въпреки това.  Извиквайки модул чрез нея подсигурява, че той е компилиран и зареден.  Това е най-полезно, когато се нуждаем от достъп до макросите на модула:

```elixir
defmodule Example do
  require SuperMacros

  SuperMacros.do_stuff
end
```

Ако се опитаме да извикаме макро, което още не е заредено, Elixir ще ни върне грешка.

### `use`

Макрото `use` извиква специално макро, наречено __using__/1, от посочения модул. Ето един пример:

```elixir
# lib/use_import_require/use_me.ex
defmodule UseImportRequire.UseMe do
  defmacro __using__(_) do
    quote do
      def use_test do
        IO.puts "use_test"
      end
    end
  end
end
```

и добавяме този ред към UseImportRequire:

```elixir
use UseImportRequire.UseMe
```

Използвайки UseImportRequire.UseMe чрез `use` дефинира функция use_test/0 чрез извикване на макрото __using__/1.

Това е всичко, което `use` прави. Обаче е честа практика за макрото __using__ да извика на свой ред alias, require, или import. Това от своя страна ще създаде съкращения или внасяния в използвания модул. Това позволява на модула, който използваме, да дефинира правила как неговите функции и макроси да бъдат реферирани. Това може да е много гъвкаво, понеже __using__/1 може да изгражда референции към други модули, особено под-модули.

Платформата Phoenix използва `use` и __using__/1, за да намали нуждада от повторяеми съкращения и внасяния в дефинирани от потребителя модули.

Ето един хубав и кратък пример от модула Ecto.Migration:

```elixir
defmacro __using__(_) do
  quote location: :keep do
    import Ecto.Migration
    @disable_ddl_transaction false
    @before_compile Ecto.Migration
  end
end
```

Макрото Ecto.Migration.__using__/1 включва извикване към `import`, така че ако изпозлваме `use` Ecto.Migration също така внасяме `import` Ecto.migration. Също така изгражда свойство на модула което предположимо контролира поведението на  Ecto.

Да обобщим: макрото `use` само извиква макрото __using__/1 на съответния модул. За да разберете наистина какво прави това, трябва да прочетете за макрото __using__/1.
