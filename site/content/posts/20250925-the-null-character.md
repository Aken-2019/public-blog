+++ 
title = "Understanding the NULL Character: Behavior Across Languages, Databases, and Editors"
date = "2025-09-25 8:00:00 +0000"
tags = ["python", "go", "javascript", "c", "null character", "programming"]
draft = false
showtoc = true
summary = "In this post, we explore the NULL character in different programming languages and databases, and how it's presented in different contexts."
+++
The NULL character, often represented as `\0`, is a special character in programming that signifies the end of a string. It is widely used in various programming languages, including C, C++, Python, Go, and JavaScript. In this post, we will explore how the NULL character is represented and used in different programming languages, databases and editors.

## Programming Languages
### C and C++
```c
#include <stdio.h>
#include <string.h> 
int main() {
    char str[] = "Hello, World!\0This part is ignored.";
    printf("String: %s\n", str); // Output: Hello, World!
    printf("Length: %zu\n", strlen(str)); // Output: 13
    return 0;
}
```

Because it's origins are in C, we start with C and C++.

In C and C++, the NULL character is represented as `'\0'`. It is used to terminate strings, which are essentially arrays of characters. The presence of the NULL character indicates the end of the string, allowing functions like `strlen` and `printf` to determine where the string ends. 
You can run the following example by yourself with this link: [C_Null_Char](https://www.programiz.com/online-compiler/2vpu3MipPaqy0)

In this example, the string `str` contains a NULL character after "Hello, World!". When printed, only the part before the NULL character is displayed. When calculating the length of the string using `strlen`, it returns 13, which is the length of "Hello, World!" excluding the NULL character and anything that follows it.


### Python


```python
# Example of NULL character in Python
s = "Hello, World!\0This part is ignored."
print("String:", s)  # Output: Hello, World!This part is ignored.
print("Length:", len(s))  # Output: 35

assert chr(0) == "\0"
```

In Python, the NULL character is represented as `'\0'`. You can also obtain this character using the `chr` function. Python strings are not null-terminated, but you can include the NULL character in a string. It is often used in binary data or when interfacing with C libraries. You can try the following code with this link: [Python_Null_Char](https://www.programiz.com/online-compiler/58MSOzhTCDkCH)

### JavaScript

```javascript
// Example of NULL character in JavaScript
let str = "Hello, World!\0This part is ignored.";
console.log("String:", str); // Output: Hello, World!This part is ignored.
console.log("Length:", str.length); // Output: 35
```
In JavaScript, the NULL character is represented as `'\0'`. JavaScript strings are not null-terminated, but you can include the NULL character in a string. You can try the following code with this link: [JavaScript_Null_Char](https://www.programiz.com/online-compiler/5gysYhcaOuqGu)

### Go

```go
package main

import (
    "fmt"
)

func main() {
    str := "Hello, World!\x00This part is ignored."
    fmt.Println("String:", str) // Output: Hello, World!This part is ignored.
    fmt.Println("Length:", len(str)) // Output: 35
}
```

In Go, the NULL character is represented as `'\0'`, but it is not commonly used. Go strings are not null-terminated; they are represented as a sequence of bytes with a length. However, you can still use the NULL character within strings.

## SQL Databases
The null character can also be used in SQL queries, although its behavior may vary depending on the database system. 

### PostgreSQL

```sql
SELECT chr(0) AS result;
-- Error: null character not permitted
```
Running the above query in PostgreSQL 16.6 will throw an error because PostgreSQL does not permit null characters in strings.

### MySQL

```sql
SELECT 
    'Hello, World!' || CHAR(0) || 'This part is ignored.' AS null_in_string,
    CONCAT('Hello, World!', CHAR(0), 'This part is ignored.') AS null_concacted,
    CHAR(0) AS null_only;

-- Output:
-- +----------------+--------------------------------------------------------------------------+----------------------+
-- | null_in_string | null_concacted                                                           | null_only            |
-- +----------------+--------------------------------------------------------------------------+----------------------+
-- |              0 | 0x48656C6C6F2C20576F726C6421005468697320706172742069732069676E6F7265642E | 0x00                 |
-- +----------------+--------------------------------------------------------------------------+----------------------+
```

Due to my limited SQL knowledge, the behavior of NULL characters in MySQL seems weird. Therefore, I asked ChatGPT to help me with this:
1. The `||` operator in MySQL is not used for string concatenation; instead, it is a logical OR operator. To concatenate strings in MySQL, you should use the `CONCAT` function. When you use `'Hello, World!' || CHAR(0) || 'This part is ignored.'`, MySQL evaluates the logical OR. `CHAR(0)` is treated as `0`, which is considered false in a boolean context. Therefore, the expression evaluates to `'Hello, World!'`, and the rest is ignored.
2. Using the `CONCAT` function with `CHAR(0)` results in a binary string because `CHAR(0)` introduces a binary context. This is what I see in MySQL 9.4.0 console. In some online MySQL playground, you may see the output as string with `null` replaced by a ` ` (space) character. This is the hexadecimal representation of the concatenated string:
- 48656C6C6F2C20576F726C6421 = "Hello, World!"
- 00 = NULL character
- 5468697320706172742069732069676E6F7265642E = "This part is ignored."
3. Selecting `CHAR(0)` alone returns a binary string representation of the NULL character, which is displayed as `0x00` in hexadecimal format.


### BigQuery

```sql
SELECT 
    'Hello, World!' || CHR(0) || 'This part is ignored.' AS null_in_string,
    CONCAT('Hello, World!', CHR(0), 'This part is ignored.') AS null_concacted,
    CHR(0) AS null_only;
-- Output:
-- +-----------------------------------+-------------------------------------+-----------+
-- | null_in_string                    | null_concacted                      | null_only |
-- +-----------------------------------+------------------------------------+-----------+
-- |Hello, World!This part is ignored. | Hello, World!This part is ignored. |           |
-- +-----------------------------------+-------------------------------------+-----------+
```

In BigQuery, the NULL character is allowed in strings, and it behaves similarly to other programings such as Go and Python. You can use the `CHR` function to represent the NULL character but it won't show anything in the terminal console or web UI.


## Editors
Most modern text editors and IDEs can handle NULL characters, but their behavior may vary. Some editors may display the NULL character as a special symbol (like `␀`), while others may ignore it or replace it with a space. It's important to be cautious when editing files that contain NULL characters, as they can affect how the file is interpreted by different programs.

### NotePad++

In my oppinion, Notepad++ is the best text editor to handle NULL characters. It displays the NULL character as `NUL` and allows you to see and edit it directly.

![Notepad++ Null Character Screenshot](/20250925-the-null-character/notepadpp_null.png)

### VSCode - Text File
In VSCode, the NULL character is not displayed. A text file containing NULL characters will be shown as an empty file.

![VSCode Null Character Screenshot](/20250925-the-null-character/vscode_null.png)


### VSCode - Notebook
Print the NULL character in a VSCode notebook will show an empty space, which is very misleading.

![VSCode Notebook Null Character Screenshot](/20250925-the-null-character/vscode_notebook_null.png)

### Jupyter Notebook
Compared to VSCode notebook, Jupyter Notebook handles NULL characters better. It does not show anything.

![Jupyter Notebook Null Character Screenshot](/20250925-the-null-character/jupyter_notebook_null.png)

### Vim
In Vim, the NULL character is displayed as `^@`. You can see and edit it directly in the editor.
![Vim Null Character Screenshot](/20250925-the-null-character/vim_null.png)

## Summary

Below are the tables summarizing the representation and behavior of the NULL character in different programming languages and SQL databases, as well as text editors.

| Language | Representation | Behavior/Notes |
|-------------------|----------------|----------------|
| C/C++             | `"\0"`         | ℹ️ Used to terminate strings. Functions like `strlen` and `printf` rely on it to determine string length. |
| Python          | `"\0"` or `chr(0)` | Allowed in strings but not shown. |
| JavaScript       | `"\0"`         | Allowed in strings but not shown. |
| Go                | `"\x00"`         |  Allowed in strings but not shown. |

| Database          | Representation | Behavior/Notes |
|-------------------|----------------|----------------|
| PostgreSQL        | `CHR(0)`      | 🚨 Not permitted in strings; will throw an error if used. |
| MySQL             | `CHAR(0)`     | Treated as an binary string. |
| BigQuery          | `CHR(0)`      | Allowed in strings but not shown. |


| Editor            | Representation | Behavior/Notes |
|-------------------|----------------|----------------|
| VSCode (Notebook) | ` `            | ⚠️ Appears as empty space.  |
| Notepad++         | `NUL`          | Displayed as `NUL`. |
| Vim               | `^@`           | Displayed as `^@`. |
| VSCode (Text File)| Not displayed  |  |
| Jupyter Notebook  | Not displayed  |  |
