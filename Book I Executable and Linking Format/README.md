
## Table of Contents

- [**Introduction**](#Introduction)
- [**File Format**](#File-Format)
- [**ELF Header**](#ELF-Header)
- [**ELF Identification**](#ELF-Identification)
- [**Sections**](#Sections)
- [**Special Sections**](#Special-Sections)
- [**String Table**](#String-Table)
- [**Symbol Table**](#Symbol-Table)
- [**Symbol Values**](#Symbol-Values)
- [**Relocation**](#Relocation)

## Introduction
**There are three main types of object files**

- **`A relocatable file`** holds code and data suitable for linking with other object files to create an executable or a shared object file.

- **`An executable file`** holds a program suitable for execution

- **`A shared object file`** holds code and data suitable for linking in two contexts. First, the link editor may process it with other relocatable and shared object files to create another object file. Second, the dynamic linker combines it with an executable file and other shared objects to create a process image.

## **File Format**

**the object file format provides parallel views of a file's contents**
<img align="right" src="https://i.imgur.com/FZ1OdYe.png" width="500" >

> [!IMPORTANT]
> **`An ELF header:`** resides at the beginning and holds a "road map'' describing the file's organization.

> [!IMPORTANT]
> **`A program header table:`** if present, tells the system how to create a process image. Files used to build a process image (execute a program) must have a program header table; relocatable files do not need one

> [!IMPORTANT]
> **`A section header table:`** contains information describing the file's sections. Every section has an entry in the table; each entry gives information such as the section name, the section size, and so on. Files used during linking must have a section header table; other object files may or may not have one.

**Sections hold the bulk of object file information for the linking view: instructions, data, symbol table, relocation information, and so on.**

> [!NOTE]
> **Although the figure shows the program header table immediately after the ELF header, and the section header table following the sections, actual files may differ. Moreover, sections and segments have no specified order. Only the ELF header has a fixed position in the file.**

### **Data Representation**

|Name| Size| Alignment| Purpose|
|--|--|--|--|
|**Elf32_Addr**| 4| 4| Unsigned program address|
|**Elf32_Half**| 2| 2| Unsigned medium integer|
|**Elf32_Off**| 4| 4| Unsigned file offset|
|**Elf32_Sword**| 4| 4| Signed large integer|
|**Elf32_Word** | 4| 4| Unsigned large integer|
|**unsigned char**| 1| 1| Unsigned small integer|

**All data structures that the object file format defines follow the "natural'' size and alignment guidelines for the relevant class. If necessary, `data structures contain explicit padding to ensure 4-byte alignment for 4-byte objects`**, to force structure sizes to a multiple of 4, and so on. Data also have suitable alignment from the beginning of the file. Thus, for example, a structure containing an Elf32_Addr member will be aligned on a 4-byte boundary within the file. For portability reasons, ELF uses no bit fields.

## **ELF Header**
**Some object file control structures can grow, because the ELF header contains their actual sizes.** If the object file format changes, a program may encounter control structures that are larger or smaller than expected. Programs might therefore ignore "extra" information. The treatment of "missing" information depends on context and will be specified when and if extensions are defined.
```c
#define EI_NIDENT 16
typedef struct 
{
unsigned char e_ident[EI_NIDENT];
Elf32_Half e_type;
Elf32_Half e_machine;
Elf32_Word e_version;
Elf32_Addr e_entry;
Elf32_Off e_phoff;
Elf32_Off e_shoff;
Elf32_Word e_flags;
Elf32_Half e_ehsize;
Elf32_Half e_phentsize;
Elf32_Half e_phnum;
Elf32_Half e_shentsize;
Elf32_Half e_shnum;
Elf32_Half e_shstrndx;
} Elf32_Ehdr; 
```
**`e_ident`** The initial bytes mark the file as an object file and provide machine-independentdata with which to decode and interpret the file's contents. [**'More information on it'**](#e_ident-Identification-Indexes)

**`e_type`** This member identifies the object file type.


|Name|Value|Meaning|
|--|--|--|
|**ET_NONE**|0|No file type|
|**ET_REL**|1|Relocatable file|
|**ET_EXEC**|2|Executable file|
|**ET_DYN**|3|Shared object file|
|**ET_CORE**|4|Core file|
|**ET_LOPROC**|0xff00|Processor-specific|
|**ET_HIPROC**|0xffff|Processor-specific|

Although the core file contents are unspecified, type **`ET_CORE`** is reserved to mark
the file type. Values from **`ET_LOPROC`** through **`ET_HIPROC`** (inclusive) are
reserved for processor-specific semantics.

**`e_machine`** This member's value specifies **the required architecture** for an individual file.

|Name|Value|Meaning|
|--|--|--|
|**ET_NONE** |0 |No machine|
|**EM_M32** |1 |AT&T WE 32100|
|**EM_SPARC** |2 |SPARC|
|**EM_386** |3 |Intel Architecture|
|**EM_68K** |4 |Motorola 68000|
|**EM_88K** |5 |Motorola 88000|
|**EM_860** |7 |Intel 80860|
|**EM_MIPS** |8 |MIPS RS3000 Big-Endian|
|**EM_MIPS_RS4_BE** |10 |MIPS RS4000 Big-Endian|
|**RESERVED** |11-16| Reserved for future use|

> [!NOTE]
> **Processor-specific ELF names use the machine name to distinguish them**. For
example, the flags mentioned below use the prefix **`EF_`**; a flag named **`WIDGET`** for
the **`EM_XYZ`** machine would be called **`EF_XYZ_WIDGET`**.

**`e_version`** This member identifies **the object file version**.

|Name|Value|Meaning|
|--|--|--|
|**EV_NONE**| 0 |Invalid version|
|**EV_CURRENT** |1 |Current version|

The value 1 signifies the original file format; extensions will create new versions with higher numbers. The value of EV_CURRENT, **though given as 1 above, will change as necessary to reflect the current version number.**

**`e_entry`** This member gives **the virtual address** to which the system first transfers control, control, thus starting the process. If the file has no associated entry point, this member **holds zero**.

**`e_phoff`** This member holds **the program header table's file offset in bytes**. If the file has no program header table, this member **holds zero**.

**`e_shoff`** This member holds **the section header table's file offset in bytes**. If the file has no section header table, this member **holds zero**.

**`e_flags`** This member holds **processor-specific flags associated with the file**. Flag names take the form EF_machine_flag.

**`e_ehsize`** This member holds **the ELF header's size in bytes**.

**`e_phentsize`** This member holds **the size in bytes of one entry in the file's program header table**;

> [!NOTE]
> All entries are the same size.

**`e_phnum`** This member holds **the number of entries in the program header table**. Thus the product of `e_phentsize` and `e_phnum` gives the table's size in bytes. If a file has no program header table, e_phnum holds the **value zero**.

**`e_shentsize`** This member holds **a section header's size in bytes**. A section header is one entry
in the section header table; all entries are the same size.


> [!NOTE]
> All entries are the same size.

**`e_shnum`** This member holds t**he number of entries in the section header table**. Thus the
product of `e_shentsize` and `e_shnum` gives the section header table's size in
bytes. If a file has no section header table, `e_shnum` holds the value zero.

**`e_shstrndx`** This member holds **the section header table index of the entry associated with the section name string table**. If the file has no section name string table, this member holds the value `SHN_UNDEF`. See "Sections" and "String Table" below for more information.

## **ELF Identification**
As mentioned above, ELF provides an object file framework to support **multiple processors, multiple data encodings, and multiple classes of machines.** To support this object file family, the initial bytes of the file specify how to interpret the file, independent of the processor on which the inquiry is made and independent of the file's remaining contents.

**The initial bytes of an ELF header (and an object file) correspond to the `e_ident` member**

### **e_ident Identification Indexes**

|Name| Value| Purpose|
|--|--|--|
|**EI_MAG0**| 0| File identification|
|**EI_MAG1** |1| File identification|
|**EI_MAG2** |2| File identification|
|**EI_MAG3**| 3| File identification|
|**EI_CLASS** |4| File class|
|**EI_DATA** |5| Data encoding|
|**EI_VERSION** |6| File version|
|**EI_PAD** |7| Start of padding bytes|
|**EI_NIDENT** |16| Size of e_ident[ ]|




















