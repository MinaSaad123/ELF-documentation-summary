
## Table of Contents

- **Introduction**
- **File Format**
- **ELF Header**
- **ELF Identification**
- **Sections**
- **Special Sections**
- **String Table**
- **Symbol Table**
- **Symbol Values**
- **Relocation**

## Introduction
**There are three main types of object files**

- **`A relocatable file`** holds code and data suitable for linking with other object files to create an executable or a shared object file.

- **`An executable file`** holds a program suitable for execution

- **`A shared object file`** holds code and data suitable for linking in two contexts. First, the link editor may process it with other relocatable and shared object files to create another object file. Second, the dynamic linker combines it with an executable file and other shared objects to create a process image.


