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

We have multiple possible approaches to changing cell value:

- Code Caves
- DLL injection
- External cheat

I chose an external memory manipulation cheat because it does not require changes in the original binary, which Steam could recognize.
Now that we have chosen the approach, we need to define the necessary steps to perform to get a persistent cheat.

1. Find the dynamic address of a cell counter.
2. Get static values that point to our dynamic address.
3. Create a program in Rust that automates the process.

## Finding a dynamic address using Cheat Engine

"Cheat Engine is by far the de facto "industry standard" for GamePwn. It bundles together essentially every possible tool that could be required into one neat little package."[2] - PandaSt0rm
We can attach CheatEngine to our running DeadCells Game. To get relevant dynamic addresses, you need to play the game, get some cells, and then scan for new values in memory.
We don't know the type of value we are looking for, but it will likely be a number. After a few scans, you are left with multiple addresses that seem relevant.

Try modifying values to see if it changes in-game value. However, to update the game GUI, you need to play and get or spend some cells; changing its memory doesn't update the game GUI counter.
And voila, the cell counter changed. But this is not persistent. We need to perform a pointer scan, to get reliable pointers.

## Defeating dynamic memory allocation (DMA)

We will use another CheatEngine feature, `pointerscan` to get a static pointer.
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

Writing in Python with libraries like pyMeow[4] would be easier, but rust libraries' low-level and non-mature nature made me understand it in more detail.
Game hacking is a great intro to Windows API. As a Linux guy, I recommend trying something similar for everybody.

- [1]: Game hacking academy - https://gamehacking.academy/about
- [2]: Intro to game hacking - https://www.hackthebox.com/blog/intro-to-gamepwn-aka-game-hacking
- [3]: Rust game hacking library - https://github.com/pseuxide/toy-arms
- [4]: Python game hacking library - https://github.com/qb-0/pyMeow





RAW POST
---------------------------------
```
In this text, we'll explore the process of creating a cheat for the popular game Dead Cells. Intention behind this guide is purely educational, and I do not endorse cheating for malicious purposes. With that out of the way, let's dive in.

First, I wanted to create a guide on creating cheat for Wesnoth, but I found that somebody has already done it, and describe it in this excellent learning resource[1].

Source code: https://github.com/matejsmycka/cell-generator

Approach

DeadCells game uses cells as currency for unlocking new items. With each run, you start with 0 cells.
I will try to change the cell value to some arbitrary number.

We have multiple possible approaches to changing cell value:

Code Caves

DLL injection

External cheat

I chose an external memory manipulation cheat because it does not require changes in the original binary, which Steam could recognize.
Now that we have chosen the approach, we need to define the necessary steps to perform to get a persistent cheat.

Find the dynamic address of a cell counter.

Get static values that point to our dynamic address.

Create a program in Rust that automates the process.

Finding a dynamic address using Cheat Engine

"Cheat Engine is by far the de facto "industry standard" for GamePwn. It bundles together essentially every possible tool that could be required into one neat little package."[2] - PandaSt0rm

We can attach CheatEngine to our running DeadCells game. To get relevant dynamic addresses, you need to play the game, get some cells, and then scan for new values in memory. We don't know the type of value we are looking for, but it will likely be a number. After a few scans, you are left with multiple addresses that seem relevant.

Try modifying values to see if it changes in-game value. However, to update the game GUI, you need to play and get or spend some cells; changing its memory doesn't update the game GUI counter.


And voila, the cell counter changed. But this is not persistent. We need to perform a pointer scan, to get reliable pointers.

Defeating dynamic memory allocation (DMA)

Because modern operating systems use DMA, we have to find out location which doesn't disappear after restarting the process.

We will use another CheatEngine feature, pointerscanner to get a static pointer.

Good resource how to use pointerscanner is here unknowncheats.me.

In order to find out static clues to dynamic location, you need to:

locate dynamic address

perform pointerscan

close game

open game

load the last pointerscan and look for the value of the cell counter.

repeat steps 3-5 until you get a reliable multilevel pointer.

It consists of the base address of some game modules, for example, Kernel32.dll+0x00048184, and multiple static values called offsets. Now, we have multiple values which point to our dynamic memory location.

Unsafe rust

I chose rust language, because lot of resources are written in C++, which is similar, however I consider writing Rust more fun then writing it in C++. Python is also solid option.

After trying multiple libraries, I chose to go with a library named toy-arms[3].  

This lib is sort of a wrapper around low level memory and process functions, so I don't have to use Windows Foreign Function Interface (FFI) directly. I ended up with the following script. 

The code can be found here.

Here is pseudocode, which explains individual steps.

function main():
    proc = getProcessByName("deadcells.exe")
    client = proc.getModuleInfo("libhl.dll")
    moduleAddr = client.baseAddress + 0x00048184
    offsets = [0x6B8, 0x4, 0x18, 0x68, 0x480, 0xC, 0x358]

    cellCounterPtr = followPointerChain(moduleAddr, offsets)
    cellCounterValue = readMemory(cellCounterPtr)
    newValue = 99999
    writeMemory(cellCounterPtr, newValue)
  

And below is actual code of main.

fn main() {
    let proc;
    match Process::from_process_name("deadcells.exe") {
        Ok(p) => proc = p,
        Err(e) => {
            println!("{}", e);
            return;
        }
    }
    let client: Module;
    match proc.get_module_info("libhl.dll") {
        Ok(m) => client = m,
        Err(e) => {
            println!("{}", e);
            return;
        }
    }

    let module_addr = client.base_address + 0x00048184;
    let offsets = [0x6B8, 0x4, 0x18, 0x68, 0x480, 0xC, 0x358].to_vec();

    let cell_counter_ptr: usize = follow_ptr_chain_u32(&proc, module_addr, &offsets);
    let mut cell_counter_value: u32 = 0;
    let _ = read(
        &proc.handle,
        cell_counter_ptr as usize,
        size_of::<u32>(),
        &mut cell_counter_value as *mut u32,
    );

    println!("Current cell counter value: {:?}", cell_counter_value);
    println!("Enter a new value:");
    let mut input = String::new();
    io::stdin()
        .read_line(&mut input)
        .expect("Failed to read line");
    let mut new_value: u32 = input.trim().parse().unwrap();
    let new_value_ptr = &mut new_value;
    let _ = write(&proc.handle, cell_counter_ptr, new_value_ptr);
    println!("Cell counter UI will be updated after sub/add cell ingame event.")
}

NOTE:  Due to endianness, reverse your offsets found by CheatEngine.

After compiling, running rust binary, Powershell window will popup.

Possible improvements:

Some nice GUI could be implemented (for easier usage) or other in-game values could be changed, like freezing value of health, but these are rather simple implementations, considering that main logic was implemented.

Wrap-up

Writing in Python with libraries like pyMeow[4] would be easier, but rust libraries' low-level and non-mature nature made me understand it in more detail.
Game hacking is a great intro to Windows API. As a Linux guy, I recommend trying something similar for everybody.

[1]: Game hacking academy - https://gamehacking.academy/about

[2]: Intro to game hacking - https://www.hackthebox.com/blog/intro-to-gamepwn-aka-game-hacking

[3]: Rust game hacking library - https://github.com/pseuxide/toy-arms

[4]: Python game hacking library - https://github.com/qb-0/pyMeow

```
