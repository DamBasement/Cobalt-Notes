# Cobalt-Notes

**Malware Essentials
**The 'implant/agent' of most C2 frameworks are written as Windows DLLs.  However, DLLs are designed to run from disk, which is unsuitable or undesirable in many scenarios.  These DLLs are therefore paired with a 'loader', which is responsible for loading a DLL from memory (rather than of from disk), in a manner that's compatible with the operating system.  This loader + DLL combination is output from the framework as shellcode (.bin file).  Shellcode doesn't really 'run' by itself, so must be injected into memory and (typically) a new thread created to execute it.  Most frameworks are also capable of producing 'ready-to-run' payloads in formats such as .exe, .dll, and .ps1 files.  These are made by taking the aforementioned shellcode (loader + DLL) and embedding it a 'shellcode runner', implemented in these various formats.


**PE file structure**
The portable executable (PE) file format holds information about a program necessary for loading it into memory.  Both .exe and .dll files are PEs.


https://en.wikipedia.org/wiki/Portable_Executable#/media/File:Portable_Executable_32_bit_Structure_in_SVG_fixed.svg

**DOS header**
The DOS header is a fixed 64-byte structure, called IMAGE_DOS_HEADER, that exists at the start of every PE file.  Most of the members are not used anymore, but the two important ones are:

_e_magic_
This is the first member and is a 2-byte WORD.  This value is always 4D 5A, or MZ in ASCII.  It's simply a signature that marks the beginning of the PE, named after Mark Zbikowski - one of the developers of MS-DOS.

_e_lfanew_
This is the last member and is a 4-byte LONG.  It contains an offset to the PE signature at the start of the NT headers.  Because it's the last member, it is always located at an offset of 3C (60 in decimal) from the start of the PE file.


**DOS stub**
The DOS stub is only used when the PE file is executed under MS-DOS, which simply prints the message "This program cannot be run in DOS mode".  This stub doesn't have an official pre-defined size, so the modern Windows loader uses the offset in e_lfanew to skip over this stub and go directly to the NT headers (described below).


**NT headers**
The Windows SDK (software development kit) has two definitions for the NT header - IMAGE_NT_HEADERS for 32-bit PEs and IMAGE_NT_HEADERS64 for 64-bit PEs.

**PE signature**
Like the e_magic member of the DOS header, the PE signature is a 4-byte DWORD that always has a fixed value of 50 45 00 00 which is PE\0\0 in ASCII.  This is used to verify that the start of the NT header has been correctly located.

**File header**
The file header is a structure called IMAGE_FILE_HEADER, which has 7 members.  Some key ones are:

_Machine_ - a 2-byte WORD that indicates the CPU architecture the PE is compiled for.  The possible values are documented here.

_NumberOfSections -_ a WORD that holds the number of sections the PE has (described below).

_SizeOfOptionalHeader _- a 2-byte WORD that holds the size of the optional header.

_Characteristics _- a 2-byte flag that describes some attributes of the PE.


**Optional header
**The optional header structure can either be IMAGE_OPTIONAL_HEADER32 or IMAGE_OPTIONAL_HEADER64 depending on the PE architecture.  Some key members include:

_Magic_ - a value that determines whether the image is PE32 (i.e. 32-bit) or PE32+ (i.e. 64-bit).

_AddressOfEntryPoint_ - the address of the PE's entry point relative to the image base when loaded into memory.

_ImageBase_ - the preferred base address for the PE image to be loaded into.

_NumberOfRvaAndSizes _- the size of the DataDirectory array.
_DataDirectory -_ an array of IMAGE_DATA_DIRECTORY structures.

**Data directories**
The IMAGE_DATA_DIRECTORY structure contains two members.  A VirtualAddress, which points to the start of a particular data directory structure; and a Size, which is the size of that data directory.  These data directories contain information needed by the Windows loader, for example, the import directory contains details of the modules (DLLs) that that PE requires to function.

**Sections**
The PE sections contain the actual data and executable code of the program.  These typically include:
.text - the executable code of the program.
.data - initialised data. 
.bss - uninitialised data.
.rdata - read-only data.
.rsrc - resources used by the program such as icons.
Each section is appended to the a header that describes it.  A section header contains a:

**Name** - the name of the section.  This field is limited to 8-bytes.
VirtualSize - the total size of the section when loaded into memory.
VirtualAddress - the memory address of the section when loaded into memory.  The value is an offset relative to the image's base address.

**SizeOfRawData **- the size of the section as it is stored on disk.  This size may be different to its virtual size, e.g. if it's padded.
Characteristics - a set of flags that describe the characteristics of the section.  One such characteristic is the final memory permissions that this section memory should be (e.g. R, RW, RX, etc).
