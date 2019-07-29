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
            results.instance.exports.importMethodTest(40);      // import -> 
    
            // export -> C-WASM method call by JS
            results.instance.exports.setNumber(30);
            let value = results.instance.exports.getNumber();
            console.log('message', value);
        });
</script>
```
