# Improve LLDB on FreeBSD

From [FreeBSD Google Summer of Code Ideas](https://wiki.freebsd.org/SummerOfCodeIdeas#Improve_LLDB_on_FreeBSD):
```
The LLVM debugger, lldb, is part of the FreeBSD base system. This project aims to extend lldb on FreeBSD in a number of ways.

1. Better kernel crash dump information

When loading a kernel core lldb just reports e.g. "Core file '/var/crash/vmcore.7' (x86_64) was loaded." We ought to select the appropriate thread when possible (e.g., the thread that panicked) and print the kernel message buffer from the core file. See notes in https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=283190 for an example of the output we get, and a reference to the developer handbook chapter showing what kgdb (GNU debugger) emits.

2. Improve lldb Lua bindings

lldb supports scripting in C++, Python, and Lua. The Lua bindings are a more recent addition, and some features that would make them much more usable on FreeBSD are missing. The first goal of this project is to improve Lua binding support, add documentation, and bring the Lua bindings to parity with the Python bindings.

Example tasks:

Enable Lua bindings to create new LLDB commands
Generate Lua documentation similar to the Python documentation
Write a tutorial for scripting LLDB with Lua
Add convenience methods to support common FreeBSD kernel debugging tasks from Lua, such as those described in https://freebsdfoundation.org/wp-content/uploads/2019/01/Debugging-the-FreeBSD-Kernel.pdf

3. Integration lldb into crashinfo

FreeBSD provides a script, /usr/sbin/crashinfo, which runs after a system (kernel) crash and extracts information from the core dump to store in a text file. See the crashinfo man page for more information. crashinfo currently makes use of the GNU debugger, gdb, which must be installed from the package collection or ports tree. The second goal of this project will be to add lldb support to crashinfo, providing the same information that crashinfo's gdb integration provides. (gdb support should be retained in the script and used when gdb is available).
```

I'll take task 1 and 3, since task 2 needs more investigation on LLDB source code.

## Better kernel crash dump information
### Create `kdb(1)` script (`kgdb(1)` equivalent) (Under review)

This is simply a wrapper script that can trigger lldb for kernel debuggin like we trigger kgdb. There are two options that won't be included in `kdb(1)`.

- `-q`: lldb doesn't print copyright banner. Printing crash info is prevented through batch mode (`-b` or `--batch`)
- `-a`: mostly used for emacs, llvm doesn't have this.

The script is ready as of Jan 24 but I need a few days to test this.

### Print kenrel message buffer (PR Submitted on LLVM)

Read and print `struct msgbuf *msgbufp`.

### Automatically select crasehd thread (PR Submitted on LLVM)

We should tell lldb which thread was stopped one. This will be treated as an exception `kernel panic`.

### Order thread list incrementally by pid (PR Submitted on LLVM)

Since `allproc` is ordered reversely, we need to reverse the order then print thread list.

### Add RegisterContext for Process/FreeBSD

* Add ppc64 support.
* Add ppc64le suppport.
* Add riscv64 support.

### Print loaded debugging symbols from kenrel modules

Currently doesn't happen in lldb.

### Show stack frame where execution stopped

Currently doesn't happen in lldb.

### Add RegisterContext for Process/FreeBSDKernel

* Add armv7 support.
* Add ppc support.
* Add ppc64 support.
* Add ppc64le suppport.
* Add riscv64 support.
* Add mips64 support.

### Enable writing to `/dev/mem`

Equivalent to `-w` in kgdb(1) but it hasn't been implemented yet. I'll implement this on lldb side (`DoWriteMemory`)

### Update test suite

Moritz System's libfbsdcore and test-vmcores repositories are inactive for years and their website is inaccessible.
Currently I submitted PR for disable testing on upstream lldb.
Those two repositories will be migrated to the FreeBSD GitHub organization and have more improvements there.
Once everything is done, I'll create complete test suite and submit it to upstream lldb.

## Integration lldb into crashinfo
### Add python script for lldb

Currently we don't have `acttrace` for lldb, so `crashinfo` cannot be ported for lldb. On kgdb, this script is automatically loaded but lldb doesn't have this facility. We can either:

1. Add python script autoload to llvm (takes time)
2. Add python script autoload just for FreeBSDKernel

### Port gdb scripts to lldb

Port `sys/tools/kernel-gdb.py` and `sys/tools/gdb/*` to lldb.

### Add lldb support for crashinfo

This will require a lot of if/fi because gdb and lldb handles stuff differently (even with options).
