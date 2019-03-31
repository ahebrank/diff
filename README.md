# sebastian/diff

Diff implementation for PHP, factored out of PHPUnit into a stand-alone component.

## Installation

You can add this library as a local, per-project dependency to your project using [Composer](https://getcomposer.org/):

    composer require sebastian/diff

If you only need this library during development, for instance to run your project's test suite, then you should add it as a development-time dependency:

    composer require --dev sebastian/diff

### Usage

#### Generating diff

The `Differ` class can be used to generate a textual representation of the difference between two strings:

```php
<?php
use Tmp\Diff\Differ;

$differ = new Differ;
print $differ->diff('foo', 'bar');
```

The code above yields the output below:
```diff
--- Original
+++ New
@@ @@
-foo
+bar
```

There are three output builders available in this package:

#### UnifiedDiffOutputBuilder 

This is default builder, which generates the output close to udiff and is used by PHPUnit.

```php
<?php

use Tmp\Diff\Differ;
use Tmp\Diff\Output\UnifiedDiffOutputBuilder;

$builder = new UnifiedDiffOutputBuilder(
    "--- Original\n+++ New\n", // custom header
    false                      // do not add line numbers to the diff 
);

$differ = new Differ($builder);
print $differ->diff('foo', 'bar');
```

#### StrictUnifiedDiffOutputBuilder

Generates (strict) Unified diff's (unidiffs) with hunks,
similar to `diff -u` and compatible with `patch` and `git apply`.

```php
<?php

use Tmp\Diff\Differ;
use Tmp\Diff\Output\StrictUnifiedDiffOutputBuilder;

$builder = new StrictUnifiedDiffOutputBuilder([
    'collapseRanges'      => true, // ranges of length one are rendered with the trailing `,1`
    'commonLineThreshold' => 6,    // number of same lines before ending a new hunk and creating a new one (if needed)
    'contextLines'        => 3,    // like `diff:  -u, -U NUM, --unified[=NUM]`, for patch/git apply compatibility best to keep at least @ 3
    'fromFile'            => null,
    'fromFileDate'        => null,
    'toFile'              => null,
    'toFileDate'          => null,
]);

$differ = new Differ($builder);
print $differ->diff('foo', 'bar');
```

#### DiffOnlyOutputBuilder

Output only the lines that differ.

```php
<?php

use Tmp\Diff\Differ;
use Tmp\Diff\Output\DiffOnlyOutputBuilder;

$builder = new DiffOnlyOutputBuilder(
    "--- Original\n+++ New\n"
);

$differ = new Differ($builder);
print $differ->diff('foo', 'bar');
```

#### DiffOutputBuilderInterface

You can pass any output builder to the `Differ` class as longs as it implements the `DiffOutputBuilderInterface`. 

#### Parsing diff

The `Parser` class can be used to parse a unified diff into an object graph:

```php
use Tmp\Diff\Parser;
use SebastianBergmann\Git;

$git = new Git('/usr/local/src/money');

$diff = $git->getDiff(
  '948a1a07768d8edd10dcefa8315c1cbeffb31833',
  'c07a373d2399f3e686234c4f7f088d635eb9641b'
);

$parser = new Parser;

print_r($parser->parse($diff));
```

The code above yields the output below:

    Array
    (
        [0] => Tmp\Diff\Diff Object
            (
                [from:Tmp\Diff\Diff:private] => a/tests/MoneyTest.php
                [to:Tmp\Diff\Diff:private] => b/tests/MoneyTest.php
                [chunks:Tmp\Diff\Diff:private] => Array
                    (
                        [0] => Tmp\Diff\Chunk Object
                            (
                                [start:Tmp\Diff\Chunk:private] => 87
                                [startRange:Tmp\Diff\Chunk:private] => 7
                                [end:Tmp\Diff\Chunk:private] => 87
                                [endRange:Tmp\Diff\Chunk:private] => 7
                                [lines:Tmp\Diff\Chunk:private] => Array
                                    (
                                        [0] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 3
                                                [content:Tmp\Diff\Line:private] =>      * @covers SebastianBergmann\Money\Money::add
                                            )

                                        [1] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 3
                                                [content:Tmp\Diff\Line:private] =>      * @covers SebastianBergmann\Money\Money::newMoney
                                            )

                                        [2] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 3
                                                [content:Tmp\Diff\Line:private] =>      */
                                            )

                                        [3] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 2
                                                [content:Tmp\Diff\Line:private] =>     public function testAnotherMoneyWithSameCurrencyObjectCanBeAdded()
                                            )

                                        [4] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 1
                                                [content:Tmp\Diff\Line:private] =>     public function testAnotherMoneyObjectWithSameCurrencyCanBeAdded()
                                            )

                                        [5] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 3
                                                [content:Tmp\Diff\Line:private] =>     {
                                            )

                                        [6] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 3
                                                [content:Tmp\Diff\Line:private] =>         $a = new Money(1, new Currency('EUR'));
                                            )

                                        [7] => Tmp\Diff\Line Object
                                            (
                                                [type:Tmp\Diff\Line:private] => 3
                                                [content:Tmp\Diff\Line:private] =>         $b = new Money(2, new Currency('EUR'));
                                            )
                                    )
                            )
                    )
            )
    )
