# Volatile-in-C

## How do compilers work ?

## What is a Volatile Keyword in C ?
- Từ khóa Volatile nó thông báo cho compiler biết là giá trị của biến có thể thay đổi bất cứ lúc nào, ngăn chặn việc tối ưu hóa chương trình của compiler.

## Syntax
- To declare a variable volatile, include the keyword volatile before or after the data type in the variable definition.
~~~cpp
volatile int foo;
int volatile foo;

volatile int * foo;
int volatile * foo;

int * volatile foo;

int volatile * volatile foo;
~~~

## When To Use Volatile in C/C++ ?
- Memory-mapped peripheral registers
- Global variables modified by an interrupt service routine
- Global variables within a multi-threaded application
