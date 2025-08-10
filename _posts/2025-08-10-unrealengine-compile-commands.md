---
title: "Unreal Engine C++ LSP integration with compile_commands.json"
date: 2025-08-09
---

## Quick Breakdown
_when writing c++ for unreal engine, you often want to use your own trusty neovim for the job, why? no particular reason_

C++ files generated for Unreal project usually live in a `Source` directory, 
and the goal is to generate the compile_commands.json so our LSP can function properly
_(function properly here means provide auto completion to functionality provided by UnrealEngine and auto generated header files)_

_(the LSP needs to know which header files to include which is usually specificed within the compile command, hence the name compile_commands.json)_

I found it usueful to wrap everything within some handy bash function that I included within my dotfiles.

and now time for some code dump

```bash
UnrealCompileCommands () {
    if [ "$1" ]; then
        UnrealCompileCommandsCoroutine $1
        return;
    fi;
    if [ "${PWD##*/}" != "Source" ]; then
        echo "Not in a 'Source' Directory, please provide a *.uproject path"
        echo "UnrealCompileCommands /path/to/file.uproject"
        return;
    fi;
    UnrealCompileCommandsCoroutine $PWD/$(ls ../*.uproject) 
}
UnrealCompileCommandsCoroutine () {

    if [ $UNREAL_PATH ]; then
        "${UNREAL_PATH%/*}/Engine/Build/BatchFiles/Linux/Build.sh" -mode=GenerateClangDatabase -project=$1 -game -engine $(cat $1 | jq '.Modules[0].Name' | tr -d '"') Development Linux
        mv "${UNREAL_PATH%/*}/compile_commands.json" .
        return;
    fi;
    echo "UNREAL_PATH is not set, please include it in your .bashrc"
}
```
1. The code above requires you to have `jq` installed, so it can lookup the Modules list within the `*.uproject` file
2. The code also requires you set up a special environment variable $UNREAL_PATH
    which you need to set before execution or within your .bashrc(depends on your terminal emulator if you're using something other than bash your probably know this already)
   or you can simply do ```bash 
   UNREAL_PATH=/path/to/unrealengine/folder UnrealCompileCommands```

## Usage: `UnrealCompileCommands` or `UnrealCompileCommands /path/to/uproject/file`
- By default the function will assume it's within a `Source` Directory and a `*.uproject` file is one level up(this might not be the case for you using this)
- It will also attempt to get the first Module name within the `*.uproject` file(this also might not be the desired behavior you can adjust this accordingly)
