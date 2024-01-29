# Kontrollstruktur

Nushell bietet diverse Befehle, die helfen, zu kontrollieren, welche Gruppen von Code ausgeführt werden.
In Programmiersprachen wird dies oft als Kontrollstruktur, oder _Kontrollfluss_ bezeichnet.

::: tip
Eine Sache, die zu beachten ist. Alle Befehle auf dieser Seite verwenden [Blöcke](/de/book/types_of_data.md#blöcke-blocks).
Dies bedeutet, dass [Umgebungsvariablen](/de/book/environment.md) und andere [veränderbare Variablen](/de/book/variables_and_subexpressions.md) darin verwendet werden können
:::

## Bereits behandelt

Weiter unten werden einige Befehle zur Behandlung des Kontrollflusses behandelt.
Es soll jedoch nicht unerwähnt bleiben, dass wir einige Befehle bereits in vorangehenden Kapiteln kennengelernt haben.
Dies sind u.A.:

- Pipelines im Kapitel [pipelines](/de/book/pipeline.md)
- Closures im Kapitel [types of data](/book/types_of_data.md)
- Iterations Befehle im Kapitel [Mit Listen arbeiten](/de/book/working_with_lists.md) page. Such as:
  - [`each`](/commands/docs/each.md)
  - [`where`](/commands/docs/where.md)
  - [`reduce`](/commands/docs/reduce.md)

## Wahl (Verzweigung)

Der folgende Befehl führt Code aus aufgrund von gegebenen Abhängigkeiten.

::: tip
Die Wahl/Verzweigungs Befehle sind Ausdrücke und geben als solche Werte zurück.
Anders als andere Befehle auf dieser Seite. Das heisst folgendes funktioniert.

```nu
> 'foo' | if $in == 'foo' { 1 } else { 0 } | $in + 2
3
```
:::

### `if`

[`if`](/commands/docs/if.md) wertet Verzweigungen aus [Blöcken](/de/book/types_of_data.md#blocks) aus, welche auf den Resultaten von einer oder mehreren Bedingungen basieren. 
Der "if" Befehl funktioniert ähnlich wie in anderen Programmiersprachen, zum Beispiel:

```nu
> if $x > 0 { 'positive' }
```

Gibt `'positive'` zurück, wenn die Bedingung `true` ist (`$x` grösser als `0`) und `null` wenn die Bedingung `false` ist (`$x` kleiner oder gleich `0`).
Wird der Verweigung ein `else` Zweig hinzugefügt, dann wird der `else` Block ausgeführt, sobald die `if` Bedingung `false` ist. Zum Beispiel:

```nu
> if $x > 0 { 'positive' } else { 'non-positive' }
```



Dieses Mal wird `'positive'` zurückgegeben wenn die Bedingung `true` (`$x` ist grösser als Null) ist,
und `'non-positive'` wenn die Bedingung `false` (`$x` ist kleiner oder gleich Null) ist.

Mehrere `if`'s können auch aneinander gereit werden:

```nu
> if $x > 0 { 'positive' } else if $x == 0 { 'zero' } else { "negative" }
```

Wenn die erste Bedingung `true` ist (`$x` ist grösser als Null) wird `'positive'` zurückgegeben.
Wenn die erste Bedingung `false` ist und die nächste Bedingung `true` (`$x` ist Null) wird `'zero'` zurückgegeben,
ansonsten wird `'negative'` ausgegeben (`$x` ist kleiner als Null).

### `match`

[`match`](/commands/docs/match.md) führt einer von mehreren Bedinungs-Zweigen aus, basierend auf den gegebenen Werten.
Ein [pattern matching](/cookbook/pattern_matching.md) (Musterabgleich) kann ebenfalls verwendet werden, um Werte aus zusammengesetzten Typen wie Listen und Records zu entpacken.

Einfach gesehen führt [`match`](/commands/docs/match.md) abhängig von Bedingungen unterschiedlichen Code aus, analog zum "switch" Mechanismus bekannt aus anderen Programmiersprachen.
[`match`](/commands/docs/match.md) prüft ob der Wert nach dem Wort [`match`](/commands/docs/match.md) dem Wert jedes Zweiges vor dem `=>` entspricht, und wenn dies erfüllt ist,
wird der Code nach dem `=>` ausgeführt.

```nu
> match 3 {
    1 => 'one',
    2 => {
        let w = 'w'
        't' + $w + 'o'
    },
    3 => 'three',
    4 => 'four'
}
three
```

Die Zweige können entweder einen einzelnen Wert zurückgeben, oder wie im zweiten Zweig das Ergebnis eines ganzen [block](/book/types_of_data.md#blocks).


#### Erreiche alle Zweige

Ein Zweig dessen Match mit `_` markiert ist, wird dann erreicht, wenn alle anderen Zweige keine Übereinstimmung zeigen.

```nu
> let foo = match 7 {
    1 => 'one',
    2 => 'two',
    3 => 'three',
    _ => 'other number'
}
> $foo
other number
```

(Zur Erinnerung, [`match`](/commands/docs/match.md) ist ein Ausdruck, weshalb das Ergebnis direkt `$foo` zugeordnet werden kann.)




#### Mustervergleich (Pattern Matching)

Werte von Typen wie Listen und Records können mit [pattern matching](/cookbook/pattern_matching.md) "entpackt" werden.
Diese Werte 


You can "unpack" values from types like lists and records with [pattern matching](/cookbook/pattern_matching.md). You can then assign variables to the parts you want to unpack and use them in the matched expressions.

```nu
> let foo = { name: 'bar', count: 7 }
> match $foo {
    { name: 'bar', count: $it } => ($it + 3),
    { name: _, count: $it } => ($it + 7),
    _ => 1
}
10
```

The `_` in the second branch means it matches any record with field `name` and `count`, not just ones where `name` is `'bar'`.

#### Guards

You can also add an additional condition to each branch called a "guard" to determine if the branch should be matched. To do so, after the matched pattern put `if` and then the condition before the `=>`.

```nu
> let foo = { name: 'bar', count: 7 }
> match $foo {
    { name: 'bar', count: $it } if $it < 5 => ($it + 3),
    { name: 'bar', count: $it } if $it >= 5 => ($it + 7),
    _ => 1
}
14
```

---

You can find more details about [`match`](/commands/docs/match.md) in the [pattern matching cookbook page](https://www.nushell.sh/cookbook/pattern_matching.md).

## Loops

The loop commands allow you to repeat a block of code multiple times.

### Loops and other iterating commands

The functionality of the loop commands is similar to commands that apply a closure over elements in a list or table like [`each`](/commands/docs/each.md) or [`where`](/commands/docs/where.md) and many times you can accomplish the same thing with either. For example:

```nu
> mut result = []
> for $it in [1 2 3] { $result = ($result | append ($it + 1)) }
> $result
╭───┬───╮
│ 0 │ 2 │
│ 1 │ 3 │
│ 2 │ 4 │
╰───┴───╯


> [1 2 3] | each { $in + 1 }
╭───┬───╮
│ 0 │ 2 │
│ 1 │ 3 │
│ 2 │ 4 │
╰───┴───╯
```

While it may be tempting to use loops if you're familiar with them in other languages, it is considered more in the [Nushell-style](book/thinking_in_nu.md) (idiomatic) to use commands that apply closures when you can solve a problem either way. The reason for this is because of a pretty big downside with using loops.

#### Loop disadvantages

The biggest downside of loops is that they are statements, unlike [`each`](/commands/docs/each.md) which is an expression. Expressions, like [`each`](/commands/docs/each.md) always result in some output value, however statements do not.

This means that they don't work well with immutable variables and using immutable variables is considered a more [Nushell-style](/book/thinking_in_nu.md#variables-are-immutable). Without a mutable variable declared beforehand in the example in the previous section, it would be impossible to use [`for`](/commands/docs/each.md) to get the list of numbers with incremented numbers, or any value at all.

Statements also don't work in Nushell pipelines which require some output. In fact Nushell will give an error if you try:

```nu
> [1 2 3] | for x in $in { $x + 1 } | $in ++ [5 6 7]
Error: nu::parser::unexpected_keyword

  × Statement used in pipeline.
   ╭─[entry #5:1:1]
 1 │ [1 2 3] | for x in $in { $x + 1 } | $in ++ [5 6 7]
   ·           ─┬─
   ·            ╰── not allowed in pipeline
   ╰────
  help: 'for' keyword is not allowed in pipeline. Use 'for' by itself, outside of a pipeline.
```

Because Nushell is very pipeline oriented, this means using expression commands like [`each`](/commands/docs/each.md) is typically more natural than loop statements.

#### Loop advantages

If loops have such a big disadvantage, why do they exist? Well, one reason is that closures, like [`each`](/commands/docs/each.md) uses, can't modify mutable variables in the surrounding environment. If you try to modify a mutable variable in a closure you will get an error:

```nu
> mut foo = []
> [1 2 3] | each { $foo = ($foo | append ($in + 1)) }
Error: nu::parser::expected_keyword

  × Capture of mutable variable.
   ╭─[entry #8:1:1]
 1 │ [1 2 3] | each { $foo = ($foo | append ($in + 1)) }
   ·                  ──┬─
   ·                    ╰── capture of mutable variable
   ╰────
```

If you modify an environmental variable in a closure, you can, but it will only modify it within the scope of the closure, leaving it unchanged everywhere else. Loops, however, use [blocks](/book/types_of_data.md#blocks) which means they can modify a regular mutable variable or an environmental variable within the larger scope.

```nu
> mut result = []
> for $it in [1 2 3] { $result = ($result | append ($it + 1)) }
> $result
╭───┬───╮
│ 0 │ 2 │
│ 1 │ 3 │
│ 2 │ 4 │
╰───┴───╯
```

### `for`

[`for`](/commands/docs/for.md) loops over a range or collection like a list or a table.

```nu
> for x in [1 2 3] { $x * $x | print }
1
4
9
```

#### Expression command alternatives

- [`each`](/commands/docs/each.md)
- [`par-each`](/commands/docs/par-each.md)
- [`where`](/commands/docs/where.md)/[`filter`](/commands/docs/filter.md)
- [`reduce`](/commands/docs/reduce.md)

### `while`

[`while`](/commands/docs/while.md) loops the same block of code until the given condition is `false`.

```nu
> mut x = 0; while $x < 10 { $x = $x + 1 }; $x
10
```

#### Expression command alternatives

The "until" and other "while" commands

- [`take until`](/commands/docs/take_until.md)
- [`take while`](/commands/docs/take_while.md)
- [`skip until`](/commands/docs/skip_until.md)
- [`skip while`](/commands/docs/skip_while.md)

### `loop`

[`loop`](/commands/docs/loop.md) loops a block infinitely. You can use [`break`](/commands/docs/break.md) (as described in the next section) to limit how many times it loops. It can also be handy for continuously running scripts, like an interactive prompt.

```nu
> mut x = 0; loop { if $x > 10 { break }; $x = $x + 1 }; $x
11
```

### `break`

[`break`](/commands/docs/break.md) will stop executing the code in a loop and resume execution after the loop. Effectively "break"ing out of the loop.

```nu
> for x in 1..10 { if $x > 3 { break }; print $x }
1
2
3
```

### `continue`

[`continue`](/commands/docs/continue.md) will stop execution of the current loop, skipping the rest of the code in the loop, and will go to the next loop. If the loop would normally end, like if [`for`](/commands/docs/for.md) has iterated through all the given elements, or if [`while`](/commands/docs/while.md)'s condition is now false, it won't loop again and execution will continue after the loop block.

```nu
> mut x = -1; while $x <= 6 { $x = $x + 1; if $x mod 3 == 0 { continue }; print $x }
1
2
4
5
7
```

## Errors

### `error make`

[`error make`](/commands/docs/error_make.md) creates an error that stops execution of the code and any code that called it, until either it is handled by a [`try`](/commands/docs/try.md) block, or it ends the script and outputs the error message. This functionality is the same as "exceptions" in other languages.

```nu
> print 'printed'; error make { msg: 'Some error info' }; print 'unprinted'
printed
Error:   × Some error info
   ╭─[entry #9:1:1]
 1 │ print 'printed'; error make { msg: 'Some error info' }; print 'unprinted'
   ·                  ─────┬────
   ·                       ╰── originates from here
   ╰────
```

The record passed to it provides some information to the code that catches it or the resulting error message.

You can find more information about [`error make`](/commands/docs/error_make.md) and error concepts on the [Creating your own errors page](/book/creating_errors.md).

### `try`

[`try`](/commands/docs/try.md) will catch errors created anywhere in the [`try`](/commands/docs/try.md)'s code block and resume execution of the code after the block.

```nu
> try { error make { msg: 'Some error info' }}; print 'Resuming'
Resuming
```

This includes catching built in errors.

```nu
> try { 1 / 0 }; print 'Resuming'
Resuming
```

The resulting value will be `nothing` if an error occurs and the returned value of the block if an error did not occur.

If you include a `catch` block after the [`try`](/commands/docs/try.md) block, it will execute the code in the `catch` block if an error occurred in the [`try`](/commands/docs/try.md) block.

```nu
> try { 1 / 0 } catch { 'An error happened!' } | $in ++ ' And now I am resuming.'
An error happened! And now I am resuming.
```

It will not execute the `catch` block if an error did not occur.

## Other

### `return`

[`return`](/commands/docs/return.md) Ends a closure or command early where it is called, without running the rest of the command/closure, and returns the given value. Not often necessary since the last value in a closure or command is also returned, but it can sometimes be convenient.

```nu
def 'positive-check' [it] {
    if $it > 0 {
        return 'positive'
    };

    'non-positive'
}
```

```nu
> positive-check 3
positive

> positive-check (-3)
non-positive

> let positive_check = {|it| if $it > 0 { return 'positive' }; 'non-positive' }

> do $positive_check 3
positive

> do $positive_check (-3)
non-positive
```
