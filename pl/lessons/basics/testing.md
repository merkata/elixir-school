---
layout: page
title: Testowanie kodu
category: basics
order: 12
lang: pl
---

Testowanie kodu jest bardzo ważną częścią procesu produkcji oprogramowania. W tej lekcji przyjrzymy się zagadnieniu 
testowania kodu w Elixirze z wykorzystaniem ExUnit. Poznamy też kilka dobrych praktyk z tym związanych.  

## Spis treści

- [ExUnit](#exunit)
  - [assert](#assert)
  - [refute](#refute)
  - [assert_raise](#assertraise)
- [Konfiguracja testów](#Konfiguracja-testów)
- [Mockowanie](#Mockowanie)

## ExUnit

Elixir posiada wbudowaną bibliotekę ExUnit, która zawiera wszystko, co potrzebne do pisania testów. Zanim zagłębimy się
w ten temat, musimy wspomnieć, że testy są w Elixirze tworzone w postaci skryptów w plikach '.exs'. Zanim uruchomimy 
nasze testy, musimy wystartować ExUnita za pomocą 'ExUnit.start()', jest to zazwyczaj robione w skrypcie 'test/test_helper.exs'.

Generując projekt w poprzedniej lekcji, mix był na tyle miły, że utworzył plik `test/example_test.exs` zawierający 
prosty test:

```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  test "the truth" do
    assert 1 + 1 == 2
  end
end
```

Możemy uruchomić nasze test za pomocą `mix test`.  Powinniśmy otrzymać komunikat podobny do poniższego:

```shell
Finished in 0.03 seconds (0.02s on load, 0.01s on tests)
1 tests, 0 failures
```

### assert

Jeżeli kiedyś pisałeś już testy to zapewne znasz pojęcie `assert`; niektóre biblioteki używają `should` lub `expect` 
zamiennie z `assert`.

Makro `assert` sprawdza, czy wyrażenie jest prawdziwe. Jeżeli nie jest, to zwróci błąd, a nasz test nie powiedzie się.
By to sprawdzić, zmieńmy nasz przykładowy test i uruchommy polecenie 'mix test':

```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  test "the truth" do
    assert 1 + 1 == 3
  end
end
```

W efekcie otrzymamy zupełnie inny komunikat:

```shell
  1) test the truth (ExampleTest)
     test/example_test.exs:5
     Assertion with == failed
     code: 1 + 1 == 3
     lhs:  2
     rhs:  3
     stacktrace:
       test/example_test.exs:6

......

Finished in 0.03 seconds (0.02s on load, 0.01s on tests)
1 tests, 1 failures
```

ExUnit dokładnie wskazuje miejsca, w których testy się nie powiodły, jakie były wartości oczekiwane, a jakie zostały 
faktycznie zwrócone.

### refute

`refute` jest tym dla `assert` czym `unless` dla `if`.  Użyj `refute` jeżeli chcesz sprawdzić wyrażenie, które zawsze
 jest nieprawdziwe.  

### assert_raise

Czasami ważne jest sprawdzenie, czy został zwrócony wyjątek. Możemy to zrobić za pomocą `assert_raise`.  W kolejnej 
lekcji poświęconej Plugowi zobaczymy przykłady zastosowania `assert_raise`.

## Konfiguracja testów

W pewnych sytuacjach musimy przygotować środowisko przed uruchomieniem testów. W tym celu możemy użyć makr `setup` i 
`setup_all`. Makro `setup` będzie uruchomione przed każdym testem, a `setup_all` zostanie uruchomione jednorazowo 
przed wszystkimi testami. Powinny one zwrócić `{:ok, state}`, gdzie `state` będzie dostępny dla naszych testów.

Przykładowo zmieńmy nasz test tak, by korzystał z `setup_all`:

```elixir
defmodule ExampleTest do
  use ExUnit.Case
  doctest Example

  setup_all do
    {:ok, number: 2}
  end

  test "the truth", state do
    assert 1 + 1 == state[:number]
  end
end
```

## Mockowanie

W Elixirze mockom mówimy stanowcze nie. Możesz mieć chęć skorzystania z mocków,, ale są one niechętnie widziane w 
społeczności Elixira i to nie bez powodu. Jeżeli będziesz podążać za wskazówkami, wzorcami i dobrymi praktykami to 
testowanie funkcji w izolacji nie będzie trudne.

Na serio, nie używaj mocków.

