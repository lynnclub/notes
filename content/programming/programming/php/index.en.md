---
title: "PHP"
type: "docs"
weight: 7
---

Official Website
[https://www.php.net/](https://www.php.net/)

Basic Syntax
[https://www.runoob.com/php/php-tutorial.html](https://www.runoob.com/php/php-tutorial.html)

## Lifecycle

![Lifecycle](php.png)

Key four calls in PHP lifecycle: MINT (Module Initialization) -> RINT (Request Initialization) -> RSHUTDOWN (Request Shutdown) -> MSHOTDOWN (Module Shutdown)

SAPI (Server Application Programming Interface) is the interaction entry point for application layer (such as Apache, Nginx, CLI, etc.). CGI, FastCGI, CLI, etc. can all be called SAPI.

## PHP Coroutines

PHP is single-threaded language, doesn't directly provide true asynchronous programming capabilities. What's implemented through yield is pseudo-coroutines, but can use Swoole extension to implement asynchronous programming.

### yield and Generators

yield is typically used with functions to create generator functions (Generator). Generators are special iterators that allow generating values one by one when needed, without having to generate all values at once and store them in memory. This is very useful for handling large amounts of data or delayed loading scenarios.

Generator functions contain one or more yield statements. Each yield statement produces a value and maintains function state in subsequent calls, allowing continuation from where it last stopped.

```php
function asyncTask() {
    yield "Step 1";
    yield "Step 2";
    yield "Step 3";
}

$task = asyncTask();

foreach ($task as $step) {
    echo $step . PHP_EOL;
}
```

In the above example, asyncTask is a generator function (pseudo-coroutine) that produces three steps through yield. Through foreach loop, these steps are executed sequentially, but after each yield, coroutine execution is suspended until the next iteration.

### Coroutine

The most commonly used PHP coroutine library is Swoole, which provides complete coroutine support including coroutine creation, management, and scheduling.

```php
Coroutine::create(function () {
    // Coroutine task code
});
```

## FPM Tuning

## Why PHP7 is Faster Than 5

Essentially optimization of zend engine, reducing memory allocation times, more use of stack memory, caching array hash values, changing string parsing to parameters into macro expansion, using large continuous memory blocks instead of small fragmented memory, etc.

1. Structure for storing variables becomes smaller, trying to make struct members share memory space, reducing references, lowering memory usage, improving variable operation speed.
2. String structure changes - string information and data itself were originally stored in two independent memory blocks, PHP7 tries to store them in same memory block, improving CPU cache hit rate.
3. Array structure changes - array elements and hash mapping tables in PHP5 would be stored in multiple memory blocks, PHP7 tries to allocate them in same memory block, reducing memory usage and improving CPU cache hit rate.
4. Improved function call mechanism, through optimization of parameter passing process, reducing some instruction operations, improving execution efficiency.
5. OpCode bytecode caching.

## Performance Analysis

Lightweight performance analysis tool xhprof
[PECL link](http://pecl.php.net/package/xhprof)
