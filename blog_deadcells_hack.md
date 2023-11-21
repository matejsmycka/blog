# Game Hacking - Creating cheat for Dead Cells using Rust

In this text, we'll explore the process of creating a cheat for the popular game [Dead Cells](https://store.steampowered.com/app/588650/Dead_Cells/). 
It's crucial to emphasize that the intention behind this guide is purely educational, and I do not endorse cheating for malicious purposes.
With that out of the way, let's dive in.

First, I wanted to create a guide on creating cheat for Wesnoth, but I found that somebody has already done it, 
and describe it in this excellent learning resource[1].

Source code: https://github.com/matejsmycka/cell-generator

## Approach

DeadCells game uses cells as currency for unlocking new items. With each run, you start with 0 cells.
I will try to change the cell value to some arbitrary number. 

We have multiple possible approaches how to changing cell value:

- Code Caves
- DLL injection
- External cheat

I chose an external memory manipulation cheat because it does not require changes in the original binary, which Steam could recognize.
Now that we have chosen the approach, we need to define the necessary steps to perform to get a persistent cheat.

1. Find the dynamic address of a cell counter.
2. Get static values that point to our dynamic address.
3. Create a program in Rust that automates the process.

## Finding a dynamic address using Cheat Engine

"Cheat Engine is by far the de facto "industry standard" for GamePwn. It bundles together essentially every possible tool that could be required, into one neat little package."[2] - PandaSt0rm
We can attach CheatEngine to our running DeadCells Game. To get relevant dynamic addresses, you need to play the game, get some cells, and then scan for new values in memory.
We don't know the type of value we are looking for, but it will likely be a number. After a few scans, you are left with multiple addresses that seem relevant.

Try modifying values to see if it changes in-game value. However, to update the game GUI, you need to play and get or spend some cells; changing its memory doesn't update the game GUI counter.
And voila, the cell counter changed. But this is not persistent. We need to perform a pointer scan, to get reliable pointers.

## Defeating dynamic memory allocation (DMA)

To get a static pointer, we will use another CheatEngine feature pointerscan.
You need to:

1. locate dynamic address
2. perform pointerscan
3. close game
4. open game
5. load the last pointerscan and look for the value of the cell counter.
6. repeat steps 3-5 until you get a reliable multilevel pointer.

It consists of the base address of some game modules, for example, `Kernel32.dll+0x00048184`, and multiple static values called offsets.
Now, we have multiple values which point to our dynamic memory location.

## Unsafe rust

I used a library named `toy-arms`[3]. This lib helped me, so I dont have to use Windows foreign function interface (FFI) directly.
I ended up with the following script. The code can be found [here](https://github.com/matejsmycka/cell-generator/blob/main/src/main.rs).
Below is pseudocode, which explains individual parts.

```
function main():
    proc = getProcessByName("deadcells.exe")
    client = proc.getModuleInfo("libhl.dll")
    moduleAddr = client.baseAddress + 0x00048184
    offsets = [0x6B8, 0x4, 0x18, 0x68, 0x480, 0xC, 0x358]

    cellCounterPtr = followPointerChain(moduleAddr, offsets)
    cellCounterValue = readMemory(cellCounterPtr)
    newValue = 99999
    writeMemory(cellCounterPtr, newValue)
  
```

Note that due to endianness, reverse your offsets found by CheatEngine.

## Wrap-up

Writing in Python with libraries like pyMeow[4] would be easier, but the low-level and non-mature nature of rust libraries made me understand it in more detail.
Game hacking is a great intro to Windows API. As a Linux guy, I recommend trying something similar for everybody.

[1]: Game hacking academy - https://gamehacking.academy/about
[2]: Intro to game hacking - https://www.hackthebox.com/blog/intro-to-gamepwn-aka-game-hacking
[3]: Rust game hacking lib - https://github.com/pseuxide/toy-arms
[4]: Python game hacking lib - https://github.com/qb-0/pyMeow
