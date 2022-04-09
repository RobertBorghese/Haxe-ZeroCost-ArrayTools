# Magic Array Tools (Haxe)
Extension functions for `Array`s/`Iterable`s that are compile-time converted to a single, optimal for-loop.

```haxe
// Place at top of file or in import.hx
using MagicArrayTools;

// ---

var arr = [1, 2, 3, 4];

arr.filter(n -> n % 2 == 0)                                for(it in arr) {
   .map(n -> "This is even number: " + n)    /* ---> */        if(it % 2 != 0) continue;
   .forEach(trace);                                            final it2 = "This is even number: " + it;
                                                               trace(it2);
                                                           }
```

---

# [Installation]
| # | What to do | What to write |
| - | ------ | ------ |
| 1 | Install via haxelib. | <pre>haxelib install magic-array-tools</pre> |
| 2 | Add the lib to your `.hxml` file or compile command. | <pre lang="hxml">-lib magic-array-tools</pre> |
| 3 | Add this top of your source file or `import.hx`. | <pre lang="haxe">using MagicArrayTools;</pre> |

---

# [Feature Index]

| Feature | Description |
| --- | --- |
| [Inline Mode](https://github.com/RobertBorghese/Haxe-MagicArrayTools/README.md#inline-mode) | A shorter, faster syntax for callbacks |
| [Disable Auto For-Loop](https://github.com/RobertBorghese/Haxe-MagicArrayTools/README.md#disable-auto-for-loop) | Temporarily or permanently disable the automatic for-loop creation |
| [`map` and `filter`](https://github.com/RobertBorghese/Haxe-MagicArrayTools/README.md#map-and-filter) | Remapping and filtering functions |
| [`forEach` and `forEachThen`](https://github.com/RobertBorghese/Haxe-MagicArrayTools/README.md#foreach-and-foreachthen) | Iterate and run an expression or callback
---

# [Core Quirks/Features]

### Inline Mode

While local functions can be passed as an argument, for short/one-line operations it is recommended "inline mode" is used. This resolves any issues that comes from Haxe type inferences, and it helps apply the exact expression where desired.

Any function that takes a callback as an argument can accept an expression (`Expr`) that's just the callback body. Use a single underscore identifier (`_`) to represent the argument that would normally be passed to the callback (usually the processed array item). This expression will be placed and typed directly in the resuling for-loop.
```haxe
[1, 2, 3].map(i -> "" + i); // Error:
                            // Int should be String
                            // ... For function argument 'i'

[1, 2, 3].map("" + _);            // Fix using inline mode!
[1, 2, 3].map((i:Int) -> "" + i); // (Explicit-typing also works)
```

&nbsp;

### Disable Auto For-Loop

In certain circumstances, one may want to disable the automatic for-loop building. Using the `@disableAutoForLoop` metadata will disable this for all subexpressions. However, for-loops can still be manually constructed by appending `.buildForLoop()`.

If manually building for-loops using `buildForLoop()` is preferred, defining the compilation flag (`-D disableAutoForLoop`) will disable the automatic building for the entire project.
```haxe
class ConflictTester {
    public function new() {}
    public function map(c: (Int) -> Int) return 1234;
    public function iterator(): Iterator<Int> { return 0...5; }
}

// ---

@disableAutoForLoop {
    final obj = new ConflictTester();
    obj.map(i -> i);                // 1234
    obj.map(i -> i).buildForLoop(); // [0, 1, 2, 3, 4]
}
```

&nbsp;

### `map` and `filter`

These functions work exactly like the `Array`'s `map` and `filter` functions.
```haxe
var arr = [1, 2, 3, 4, 5];

var len = arr.filter(_ < 2).length;
assert(len == 1);


var spaces = arr.map(StringTools.lpad("", " ", _));
assert(spaces[2] == "   ");
```

&nbsp;

### `forEach` and `forEachThen`

Calls the provided function/expression on each element in the `Array`/`Iterable`. `forEachThen` will return the array without modifying the elements. On the other hand, `forEach` returns `Void` and should be used in cases where the iterable object is not needed afterwards.
```haxe
// do something 10 times
(0...10).forEach(test);    /* ---> */    for(it in 0...10) {
                                             test(it);
                                         }
```

```haxe
// add arbitrary behavior within for-loop
["i", "a", "bug", "hello"]                    /* ---> */    {
    .filter(_.length == 1)                                      final result = [];
    .forEachThen(trace("Letter is: " + _))                      for(it in ["i", "a", "bug", "hello"]) {
    .map(_.charCodeAt(0));                                          if(it.length != 1) continue;
                                                                    trace("Letter is: " + it);
                                                                    final it2 = it.charCodeAt(0);
                                                                    
                                                                    result.push(it2);
                                                                }
                                                                result;
                                                            }
```

&nbsp;
