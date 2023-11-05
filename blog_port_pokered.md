# Blog: Porting a GameBoy game to the Web.

Picture this: Taking a game as classic as Pokemon Red and making it run in your web browser. 

This text describes the process of making a legacy game to run in the browser via compiler [Emscripten](https://emscripten.org/).
The code is executed on the client side, the main advantage being that no server is required, so it can be hosted for free on Gitlab or GitHub Pages.
This approach is truly cross-platform, making it possible to play the same game on any device and architecture that supports browsers. 

Here you can try it yourself [https://525073.gitlab-pages.ics.muni.cz/wasm-pokemon/](https://525073.gitlab-pages.ics.muni.cz/wasm-pokemon/)

## How it works

Old games need an emulator to be playable, which simulates the original hardware but as software on modern PCs.
Emulators can be written in any language. However, in order to make it run on the client side, it has to be in a language that the browser runtime can execute.
Historically, browsers could execute only javascript, but in the last years, all major browsers implemented support for [WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly/Concepts) (WASM).
Emulators, which are written in some compiled language like C, C++, Rust, or Go (Checkout [list](https://webassembly.org/getting-started/developers-guide/) here.), can be compiled to WASM. You can then hardcode the game into the emulator itself.
So when you visit the website, JavaScript will load WASM binary with an embedded emulator and game ROM.

## Step 1. - Choosing a compiler

There are quite a lot of compilers, however, I found Emscripten to be the best because it handles many things like automatic HTML and CSS generation for your WASM binary.

## Step 2. - Choosing emulator

There are a lot of limitations that will occur in porting something to WASM. WASM has its own interface [WASI](https://wasi.dev/), and unlike POSIX, you can't just read and write to files as you wish.
All files that WASM runtime can access have to be bundled in the binary itself. It also cannot call any external libraries. Everything has to be included in compile time. Fun fact, you can't even open a network socket in the WASM.
So picking as lightweight an emulator as possible is crucial.

Another thing is that you have to choose a graphic library that the compiler supports. Emscripten supports SDL2 and Raylib, 
After trying multiple emulators, I chose [GameBoy Emulator by David Jolly](https://sr.ht/~dajolly/dmgl/) because it supported both libraries and was lightweight.

## Step 3. Porting

In this part, you have to load the legally obtained ROM of the game directly to the code. I used a tool `xxd`, which translates the binary directly into a huge C array, it is ugly but reliable.
Remove any ARGS that have to be provided. It is not feasible to launch the WASM binary with arguments, but it can be done in the javascript loader. Remove any features that are not mandatory for the emulator,
like Controller support. 

## Step 4. Compiling

This is the part where I spend most of the time, modifying all compiler options is tedious work, and I couldn't get the RayLib to work.
After a lot of time of googling and trying different options, I came up with the following command:
`emcc -o build/index.html $(find src -name "*.c" && find tool -name "*.c") -Wall -std=c99 -D_DEFAULT_SOURCE -Wno-missing-braces -Wunused-result -Os -I. -Itool  -I src/ -I src/system -s USE_GLFW=3 -s ASYNCIFY -s TOTAL_MEMORY=67108864 -s FORCE_FILESYSTEM=1 --shell-file /usr/lib/emscripten/src/shell_minimal.html -DPLATFORM_WEB -s "EXPORTED_FUNCTIONS=["_free","_malloc","_main"]" -s EXPORTED_RUNTIME_METHODS=ccall -DCLIENT_SDL2 -sUSE_SDL=2 -s ALLOW_MEMORY_GROWTH=1 -s TOTAL_STACK=32MB`
It could be optimized like the header files could be imported automatically, the total memory is overkill, and some options might be redundant.
After finishing the compile stage, the compiler will yield HTML, CSS, JS, and WASM files.
Run `python -m http.server` to play the game locally.

## Step 5. Tinkering

In this stage, you can alter the files above, like modifying the CSS to resize the play screen or modifying the HTML to alert the player how to play.
Finally, deploy the project with some free services, below is an example of how it easy to the game on GitLab Pages:
```
image: ruby:latest

pages:
  stage: deploy
  script:
    - mkdir public
    - cp build/index.wasm ./public/
    - cp index.html ./public/
    - cp index.js ./public/
  artifacts:
    paths:
    - public
```

Thank you for reading, feedback is appreciated.

matejsmycka@gmail.com
