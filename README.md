# C Memory Basics
A beginner-friendly C study guide (aimed at libft) covering memory concepts: stack vs. heap, const, signed/unsigned, static, scope vs. lifetime, dynamic allocation with malloc/free, the call stack, thread-local storage, and common memory bugs, ending with a quick-revision table.

## 1. Memory: where your data lives

**Stack**
Typically stores local variables and function-call information.

**Heap**
Memory you request dynamically using `malloc`, for example.

**Static storage**
Stores global variables and static variables for the program's lifetime.

## 2. const — cannot modify through this variable

**const** means the object cannot be modified through the particular declaration.
```c
const int age = 20;
// age = 25; // Error
```
**Important**: **const** does not automatically mean the data is stored in read-only memory.

## 3. **signed** and **unsigned** — number ranges

These describe whether an integer type can represent negative numbers.
```c
signed   int a = -10;
unsigned int b = 10;
```
**signed int**: can store negative and positive values.
**unsigned int**: stores only non-negative values.

For a typical 32-bit int:

| Type         | Typical range                   |
|--------------|---------------------------------|
| int          | −2,147,483,648 to 2,147,483,647 |
| unsigned int | 0 to 4,294,967,295              |

In **libft**, you often use `unsigned char` for memory functions because it represents byte values from 0 to 255.
```c
unsigned char c = 255;
```

## 4. Scope — where can you use a variable?

**Scope** = where a variable's name is visible in your code.

```c
void test(void)
{
    int x = 10;

    if (x > 0)
    {
        int y = 20;
        // x and y are accessible here
    }
    // x is accessible here
    // y is not accessible here
}
```
- `x` has block scope in the function body.
- `y` has block scope inside the `if` block.

When you leave a block, the variable's name is no longer accessible from outside it.

## 5. Lifetime — how long does a variable exist?

**Lifetime** = how long an object exists in memory during program execution.
```c
void test(void)
{
    int x = 10;
}
```
The local variable `x` has automatic storage duration: it ends when execution leaves its block.

Compare that with a `static` variable:
```c
void test(void)
{
    static int x = 10;
    x++;
}
```
Every time you call `test`, `x` keeps its previous value.

The key difference:
- `Scope`: Where can I access its name?
- `Lifetime`: How long does the object exist?

These are different concepts. A variable can have a limited scope but live for the entire program.


## 6. `static` — two common uses

1. Inside a function
```c
void counter(void)
{
    static int n = 0;
    n++;
}
```
`n` keeps its value between function calls and exists for the entire program execution.

2. Outside a function
```c
static int number = 10;
static void helper(void) {}
```

At file scope, `static` gives the variable or function **internal linkage**: its name is only accessible within that source file.

Remember: `static` does not always mean the same thing; its effect depends on where you use it.

## 7. Stack and heap — two important memory concepts

1. Stack

Typically used for function calls and local variables with automatic storage duration.
```c
void test(void)
{
    int x = 10;
}
```
The local `x` stops existing when the function's block ends.

2. Heap

Used for memory you request dynamically.
```c
char *s = malloc(sizeof(char) * (len + 1));
```
The allocated object stays alive until you release it with `free(s)`.

The stack is generally managed automatically. Heap memory allocated with `malloc` must be managed by your code.

Technically, C does not require implementations to use a stack and heap exactly like this, but this model is useful for learning.

## 8. Automatic and dynamic allocation

**Allocate** means reserve memory for an object.

1. Automatic allocation
```c
void test(void)
{
    int x = 10;
}
```
The object `x` has automatic storage duration. You do not call `malloc` or `free` for it.

2. Dynamic allocation
```c
#include <stdlib.h>

int main(void)
{
    int *p = malloc(sizeof(int));

    if (!p)
        return (1);
    *p = 212;
    free(p);
    return (0);
}
```

Here:
1. `malloc(sizeof(int))` requests enough memory for one `int`.
2. `p` stores the address returned by `malloc`.
3. `*p = 212` writes `212` into the allocated memory.
4. `free(p)` releases the allocated memory.

**Important**: `p` is a local pointer variable, but the memory it points to is dynamically allocated. The pointer and the allocated object are two different things.

## 9. Dynamic memory allocation

Dynamic allocation lets you request memory while your program is running, when you know how much you need.

The main functions are:

| Function              | Purpose                                                   |
|-----------------------|-----------------------------------------------------------|
| `malloc(size)`        | Allocates memory; contents are uninitialized.             |
| `calloc(count, size)` | Allocates memory and initializes all bytes to zero.       |
| `realloc(ptr, size)`  | Resizes an existing allocation, possibly moving it.       |
| `free(ptr)`           | Releases a dynamic allocation.                            |

Example:
```c
#include <stdlib.h>
#include <string.h>

// Create a string and allocate memory for it
char *s = malloc(sizeof(char) * 6);
// Check if the allocation failed, then return NULL
if (!s)
    return (NULL);

// Set value: five bytes for the characters, and one byte for the
// null terminator '\0' that closes the string
strcpy(s, "Hello");

// Free memory
free(s);
```
This requests six bytes for six characters. If you're storing "Hello" as a C string, you need six bytes: five characters plus the terminating `'\0'`.

Note: `s = "Hello";` would be wrong here. It would replace the pointer returned by `malloc` with the address of a string literal, leaking the allocated memory and making `free(s)` invalid. Copy the characters into the allocated memory instead (`strcpy`, or `ft_strlcpy` in libft).

## 10. Call stack — what happens when functions call each other?

The call stack tracks active function calls.
```c
void c(void)
{
    int x = 3;
}

void b(void)
{
    c();
}

int main(void)
{
    b();
}
```
The order is:

`main()` starts
`b()` is called
`c()` is called
`c()` finishes → `b()` finishes → `main()` continues

Each function call typically has its own stack frame containing information needed to execute and return from that call.

Stack vs. call stack: the call stack is the structure used to track function calls; the stack is the memory area commonly used to implement it.

## 11. Thread-local storage (TLS)

A thread is an independent sequence of execution inside a program.
Thread-local storage means each thread gets its own separate instance of a variable.
In C11, you can use `_Thread_local`:
```c
_Thread_local int counter = 0;
```

If two threads access `counter`, each thread has its own value. Changing it in one thread does not change the other thread's instance.
This is useful in multithreaded programs, but you don't need it for basic libft functions.

## 12. Memory management — avoiding memory problems

Memory management means controlling how memory is allocated, used, and released.
Three important problems to understand:

1. Memory leak

You allocate memory but forget to release it when it's no longer needed.
```c
int *p = malloc(sizeof(int));
// Forgot to free(p)
```

2. Use-after-free

You access memory after releasing it.
```c
free(p);
// *p = 10;  // Invalid
```

3. Buffer overflow

You write beyond the memory you have available.
```c
char s[5];
// s[5] = 'A';  // Out of bounds
```

Valid indexes are 0 through 4.

## Quick revision

| Concept               | Simple meaning                                                                                  |
|-----------------------|-------------------------------------------------------------------------------------------------|
| const                 | Prevents modification through a particular declaration.                                         |
| signed                | Allows negative integer values.                                                                 |
| static                | Gives static storage duration to a local variable; at file scope, also gives internal linkage.  |
| Scope                 | Where a name can be used.                                                                       |
| Lifetime              | How long an object exists.                                                                      |
| Stack                 | Commonly used for function calls and automatic local variables.                                 |
| Heap                  | Commonly used for dynamically allocated objects.                                                |
| Automatic allocation  | Storage duration managed automatically by the language implementation.                          |
| Dynamic allocation    | Requesting memory at runtime using functions like `malloc`.                                     |
| Memory management     | Allocating, using, and releasing memory correctly.                                              |
| Thread-local storage  | Each thread has its own instance of a variable.                                                 |