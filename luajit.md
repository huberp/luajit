---
project: luajit
tagline: LuaJIT binary
---

## What

LuaJIT binaries (frontend, static library, dynamic library).

Comes bundled the `luajit` command, which is a simple shell script that
finds and loads the appropriate luajit executable for your platform, so that
typing `./luajit` (that's `luajit` on Windows) always works.

LuaJIT was compiled using its own makefile.

## Making portable apps

To make portable apps that run from any directory, every subsystem that
ultimately opens a file must look for that file in a location relative
to the app's directory. This means at least three things:

 * Lua's require() must look in exe-relative dirs first,
 * the OS's shared library loader must look in exe-relative dirs first,
 * the app itself must look for assets, config files, etc. in exe-relative
 dirs first.

The solutions for the first two problems are platform-specific and
are described below. As for the third problem, you can extract the exe's
path from arg[-1]. To get the location of the _running script_,
as opposed to that of the executable, use [glue.bin]. To add more paths
to package.path and package.cpath at runtime, use [glue.luapath]
and [glue.cpath].

### Finding Lua modules

#### Windows

`!\..\..\?.lua;!\..\..\?\init.lua` was added to the default package.path
in luaconf.h. This allows luapower modules to be found regardless of what
the current directory is, making the distribution portable.

The default `package.cpath` was also modified from `!\?.dll` to `!\clib\?.dll`.
This is to distinguish between Lua/C modules and other binary dependencies.

#### Linux and OSX

In Linux and OSX, luajit is a shell wrapper script that sets LUA_PATH
and LUA_CPATH to acheive the same effect and assure isolation from
system libraries.

#### The current directory

Lua modules (including Lua/C modules) searched for in the current directory
___first___ (on any platform), so the isolation from the host system
is not absolute.

This is the Lua's default setting and although it's arguably a security risk,
it's convenient for when you want to have a single luapower tree, possibly
added to the system PATH, to be shared between many apps. In this case,
starting luajit in the directory of the app makes the app's modules
accessible automatically.

### Finding shared libraries

#### Windows

Windows looks for dlls in the directory of the executable first by default,
and that's where the luapower dlls are, so isolation from system libraries
is acheived automatically in this case.

#### Linux

Linux binaries are built with `rpath=$ORIGIN` which makes ldd look for
shared objects in the directory of the exe first.

#### OSX

OSX binaries are built with `rpath=@loader_path` which makes the
dynamic loader look for dylibs in the directory of the exe first.

#### The current directory

The current directory is _not used_ for finding shared libraries
on Linux and OSX. It's only used on Windows, but has lower priority
than the exe's directory (except on WinXP before SP2 where it has
higher priority).


[glue.bin]:     glue#bin
[glue.luapath]: glue#luapath
[glue.cpath]:   glue#cpath

