# Setup Environment

```
1). First Visit [WasmFiddle](https://wasdk.github.io/WasmFiddle)
2). Need to familiar with C or C++ syntax, we already know webassembly can be compile from different languages. But here we are practicing only C and C++.
3). Now write simple C code like below, in left `C Code Panel`
```

```c
int number = 0;

int main() { 
  return 200;
}

int getNumber() {
  return number;
}

void setNumber(n) {
  number = n;
}
``` 

4). On the Right Side 'JS Code Panel'

```js
let wasmModule = new WebAssembly.Module(wasmCode);
let wasmInstance = new WebAssembly.Instance(wasmModule, wasmImports);

let exports = wasmInstance.exports;

log(exports.main());                   // calling main method
exports.setNumber(20);                 // set value
log(exports.getNumber());              // get value
```

5). Now hit the `run` button and see output at the `right bottom Panel`.
6). For custom implementation closer look at `right bottom panel` download `WASM` button to download it [eg. program.wasm]
7. Now play with examples.

References:
[https://developer.mozilla.org/en-US/docs/WebAssembly](https://developer.mozilla.org/en-US/docs/WebAssembly)

# Lesson 01 - Basic 

First Need to load `wasm` file

```html

<h1>Hello! From Webassembly</h1>
<script>
    WebAssembly.instantiateStreaming(fetch('program.wasm'))         // here load your WebAssembly Binary
        .then(results => {

            // just make it global and understand wasm binary
            window.wasm = results;
            console.log(results);
            console.log(WebAssembly.Module.exports(results.module));
        });

</script>

```

# Lesson 02 - Export (JS Call Our WASM methods)

```html

<h1>Hello! From Webassembly</h1>
<script>
    WebAssembly.instantiateStreaming(fetch('program.wasm'))         // here load your WebAssembly Binary
        .then(results => {

            // exports wasm methods
            let exports = results.instance.exports;
            exports.setNumber(120);                         // calling method to set value
            let number = exports.getNumber();               // get the value by calling binary .wasm file method
            console.log(number);

        });

</script>
```

# Lesson 03 - Import (Opposite, WASM Call Our JS methods)

`Step 1:` Create new C program, download `program.wasm` file and updated previous one.

```c
int number = 0;
void consoleLogMessage(int n);
int main() { 
  return 200;
}
int getNumber() {
  return number;
}
void setNumber(n) {
  number = n;
}
int importMethodTest(int n){
  consoleLogMessage(n);
}
```

`Step 2:` update your `.index.html` file

```html
<h1>Hello! From Webassembly</h1>
<script>
    // this import reference to `program.wasm`
    const imports = {
        env: {
            consoleLogMessage: console.log
        }
    };
    WebAssembly.instantiateStreaming(fetch('program.wasm'), imports)
        .then(results => {
            console.log('WASM Loaded................');
           
            // import -> JS method call by C-WASM
            results.instance.exports.importMethodTest(40);
    
            // export -> C-WASM method call by JS
            results.instance.exports.setNumber(30);
            let value = results.instance.exports.getNumber();
            console.log('message', value);
        });
</script>
```

# Lesson: 04 - Export & Import Overview

```html
<script>
    const imports = {
        env: {
            consoleLogMessage: console.log
        }
    };
    WebAssembly.instantiateStreaming(fetch('program.wasm'), imports)
        .then(results => {
            console.log('WASM Loaded................');


            // just make it global and understand wasm binary
            window.wasm = results;


            // import
            results.instance.exports.importMethodTest(40);


            // Export
            results.instance.exports.setNumber(30);
            let value = results.instance.exports.getNumber();
            console.log('message', value);


            console.log('All the Imports Methods: ', WebAssembly.Module.imports(results.module));         // fetch all the `Import` method
            console.log('All the Exports Methods: ', WebAssembly.Module.exports(results.module));         // fetch all the `Export` method
        });

</script>
```

# Lesson 05: Memory & String

Create new `program.wasm` file, and updated 

```
#include <string.h>

void numberLog(int n);
void stringLog(char *offset, int length);

int main() { 
  return 200;
}

void getNumber(n) {
  numberLog(n);
}

int messageLog(int n){
  char * msg = "Hello! From C Code"; 
  stringLog(msg, strlen(msg));
}
```
> Updated Script File

```html
<script>
    // Read data from webassembly memory ArrayBuffer
    const readMemoryStringFromBuffer = (offset, length) => {
        let memoryBuffer = wasm.instance.exports.memory.buffer;

        const strBuffer = new Uint8Array(memoryBuffer, offset, length);
        const string = new TextDecoder().decode(strBuffer);
        console.log('message', string);
    };

    const imports = {
        env: {
            numberLog: console.log,
            stringLog: readMemoryStringFromBuffer
        }
    };

    WebAssembly.instantiateStreaming(fetch('program.wasm'), imports).then(results => {

        // make it global
        window.wasm = results;

        console.log('WASM Loaded................');


        console.log('All Import Methods:- ', WebAssembly.Module.imports(results.module));
        results.instance.exports.getNumber(33);
        results.instance.exports.messageLog();
    });
</script>
```

# Lesson 06 - Webassembly Memory and Event
> Using event directly update user interface, Cause Webassembly doesn't access DOM.

```html
<script>

    // Read data from webassembly memory ArrayBuffer
    const readMemoryStringFromBuffer = (offset, length) => {
        let memoryBuffer = wasm.instance.exports.memory.buffer;

        const strBuffer = new Uint8Array(memoryBuffer, offset, length);
        const string = new TextDecoder().decode(strBuffer);

        // Make notify and make use of that string throw event
        window.dispatchEvent(new CustomEvent('wasmValue', {detail: string}));
    };
    window.addEventListener('wasmValue', result => {
       console.log('Receive String From C:- ', result.detail);
    });

    const imports = {
        env: {
            numberLog: console.log,
            stringLog: readMemoryStringFromBuffer
        }
    };

    WebAssembly.instantiateStreaming(fetch('program.wasm'), imports).then(results => {

        // make it global
        window.wasm = results;

        console.log('WASM Loaded................');


        console.log('All Import Methods:- ', WebAssembly.Module.imports(results.module));
        results.instance.exports.getNumber(33);
        results.instance.exports.messageLog();
    });

</script>
```

# Lesson 07 - Webassembly Dynmaic Custom Memory Import

> Go to [WasmFiddle](https://wasdk.github.io/WasmFiddle/)

```c

#include <string.h>

void numberLog(int n);
void stringLog(char *offset, int length);

int main() { 
  return 200;
}

void getNumber(n) {
  numberLog(n);
}

int messageLog(int n){
  char * msg = "Hello! From C Code"; 
  stringLog(msg, strlen(msg));
}

```

Note: Build Raw/Plain `Wat file` and `copy it`
Now go to [Webassembly Studio](https://webassembly.studio/)

> open `main.wat` file and `paste` plain wat

```wat
(module
 (type $FUNCSIG$vi (func (param i32)))
 (type $FUNCSIG$vii (func (param i32 i32)))
 (import "env" "numberLog" (func $numberLog (param i32)))
 (import "env" "stringLog" (func $stringLog (param i32 i32)))
 (import "env" "memory" (memory $0 1))
 (table 0 anyfunc)
 (data (i32.const 16) "Hello! From C Code\00")
 (export "memory" (memory $0))
 (export "main" (func $main))
 (export "getNumber" (func $getNumber))
 (export "messageLog" (func $messageLog))
 (func $main (; 2 ;) (result i32)
  (i32.const 200)
 )
 (func $getNumber (; 3 ;) (param $0 i32)
  (call $numberLog
   (get_local $0)
  )
 )
 (func $messageLog (; 4 ;) (param $0 i32) (result i32)
  (local $1 i32)
  (call $stringLog
   (i32.const 16)
   (i32.const 18)
  )
  (get_local $1)
 )
)
```

Change import memory declaration, then `Save` that file.
Now, From Out Download `main.wasm` file,

> Finally updated `.html` file

```html
<script>

    // Initialise wasm with custom memory (array buffer)
    // 2 pages: 2 * 64kb (128kb)
    const wasmMemory = new WebAssembly.Memory({ initial: 2 });


    // Read data from webassembly memory ArrayBuffer
    const readMemoryStringFromBuffer = (offset, length) => {
        let memoryBuffer = wasmMemory.buffer;

        const strBuffer = new Uint8Array(memoryBuffer, offset, length);
        const string = new TextDecoder().decode(strBuffer);

        // Make notify and make use of that string throw event
        window.dispatchEvent(new CustomEvent('wasmValue', {detail: string}));
    };
    window.addEventListener('wasmValue', result => {
        console.log('Receive String From C:- ', result.detail);
    });

    const imports = {
        env: {
            numberLog: console.log,
            stringLog: readMemoryStringFromBuffer,
            memory: wasmMemory,                     // dynamically initialize memory
        }
    };

    WebAssembly.instantiateStreaming(fetch('/main.wasm'), imports).then(results => {

        // make it global
        window.wasm = results;
        console.log('WASM Loaded................');


        console.log('All Import Methods:- ', WebAssembly.Module.imports(results.module));
        results.instance.exports.getNumber(33);
        results.instance.exports.messageLog();
    });

</script>
```
 
