+++
date = '2025-09-07T18:35:32-04:00'
draft = false
title = 'Cs644 Week 1'
tags = ['cs644']
+++

(★) What is the size_t type for in C?

`size_t` is an integer type, and the result of `sizeof` and other size functions. It's particularity is that it can hold the maximum size of the largest theoretical object in a program of any type (including arrays). This is important because normal `int` types may fail once they reach the system's architectural limits. 

(★) True or false: Semicolons are optional in C.

`false`. Semicolons are mandatory in c!

(★★) Start thinking about what language you want to use for the final project, and whether you want to follow the main track or work on your own project. (See syllabus for details.)

Given that I'm currently taking a class called `Understanding Networks`, I quite like the idea of following along with the syllabus, using the `C` language. I was tempted to lean towards CL, but I'm not proficient enough both learn the concepts and the language, while juggling a master's program as well.

(★★) Does this program have a bug? Why or why not?

```c
#include <stdbool.h>

enum LanguageCode {
  LANG_EN, // English
  LANG_FR, // French
  LANG_RU, // Russian
};

struct Language {
  enum LanguageCode code;
  bool has_verb_conjugations;
  bool has_grammatical_gender;
  bool has_noun_declensions;
};

struct Language make_language(enum LanguageCode code) {
  struct Language lang = {
    .code = code,
    .has_verb_conjugations = false,
    .has_grammatical_gender = false,
    .has_noun_declensions = false,
  };

  switch (code) {
    case LANG_RU:
      lang.has_noun_declensions = true;
    case LANG_FR:
      lang.has_verb_conjugations = true;
      lang.has_grammatical_gender = true;
      break;
    case LANG_EN:
      lang.has_verb_conjugations = true;
      break;
  }

  return lang;
}

int main() {}
```

This code compiles with no complaints. It has no output, given that `main` is unpopulated. That said, for correctness, we would want `main`, if set to an int, to return an actual `int`.

Otherwise, we could consider the absence of a `break` under `LANG_RU` to be a bug, unless the cascading effect is desired (which it very well might be).

(★★) Take a look at this C function from the Python source code. Try to understand what it does and how it works. (Bonus: Can you explain how the second function in the file works?)

```c
int
PyOS_mystrnicmp(const char *s1, const char *s2, Py_ssize_t size)
{
    const unsigned char *p1, *p2;
    if (size == 0)
        return 0;
    p1 = (const unsigned char *)s1;
    p2 = (const unsigned char *)s2;
    for (; (--size > 0) && *p1 && *p2 && (Py_TOLOWER(*p1) == Py_TOLOWER(*p2));
         p1++, p2++) {
        ;
    }
    return Py_TOLOWER(*p1) - Py_TOLOWER(*p2);
}
```

This function compares two strings using basic pointer arithmetic. 
1. It dereferences the provided pointers into internal pointers.
2. It then sets about comparing these, checking wether the character they represent is the same, as well as wether the pointer points to `\0`, the string's null-terminator.
3. What's unclear is how useful returning the difference between the two characters that are different in the string is.

(★★) Write a C program, redact.c, that takes a string argument and prints out the string with all digits replaced by the character 'X'. (Hint: Some standard library functions might come in handy.)

I'm using a similar function to the Python library one we saw above.

```c
#include <stdio.h>
#include <ctype.h>

int main() {
    char s[2048];
    char *p = s;

    fgets(s, sizeof s, stdin);

    while (*p) {
        if (isdigit(*p)) {
            *p = 'X';
        }
        *p++;
    }

    printf("%s\n", s);
}
```

(★★★) Research the concept of the stack and the heap for memory allocation. (Not to be confused with the data structures of the same name.) When do you use one versus the other? How do I know if a value in a C program is allocated on the stack or the heap?

A **stack** is a dynamically-allocated memory structure, part of a program, that is populated by local variables and functions. As a program runs trough functions, these are placed on the stack, along with a return address (like a GOTO for when the function ends), as well as the function's local variables. The stack is managed, but can potentially suffer from buffer overflows if the array addresses are incorrect.

In general, local variables and functions will be placed on the stack.

The **heap** is another portion of memory that is accessed through the `new` operator in C. This memory is composed of 'regions', and is *not* managed, unlike the stack. One has to look at and manage lifetimes, deallocation, etc.

In general, constructs created with `new` will be placed on the heap.

(★★★) What is the memory representation of strings (char*) in C? What is an alternate representation? What are the advantages and disadvantages of each?

Strings are represented as a sequence of bytes in memory. A pointer is provided, which represents the first memory location of the string's characters, and we assume that contiguous memory locations are provided for the other characters. We also assume strings are null terminated, and thus can test for their endings. 

This is also why in C the length of a string is always (or at least predominantly) passed to string functions - it is necessary given that we don't *really* know how long a string is (we can use `strlen`, which I assume checks for null).

Another representation is a `string literal`. A string literal is declared as such, `char *a = "hello my friend"`.

These strings live in read-only-memory, and cannot be modified. That said, their memory information *can* be copied over to dynamic memory, and then used.

`char s[] = "hello my friend"`
`char *p = (malloc, 6)`
`strcpy(p, "hello")`