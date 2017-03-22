# Compiling ffi for nw

In the following nwjs version *0.20.2* and *x64* architecture shall be replaced with your nwjs actual version and architecture

1. Install nw-gyp
```
npm install -g nw-gyp
```

2. Install module locally
```
npm install ffi --save
```
	
3. Go to module dir
```
cd node_modules/ffi
```
	
4. Set python27 path if not present in PATH
```
set PATH=%PATH%;C:\Python27
```
	
5. Rebuild ffi
```
nw-gyp rebuild --target=0.20.2 --arch=x64
```

7. *Then* go to ref path and rebuild it for nwjs
```
cd ../ref
nw-gyp rebuild --target=0.20.2 --arch=x64
```
	
8. Now you can use ffi with nwjs
	

# Loading and using DLL modules in nodejs and nwjs

## Basics

Suppose to have a Dynamic Link Library (DLL) exporting C functions.
- *The DLL shall match the architecture of the nodejs or nwjs in use*
- *On Windows the DLL shall export functions in a* `extern "C" { ... }` *blocks*, for example:

win32project1.h:
```
extern "C"
{
	WIN32PROJECT1_API int sum(int num, int* numbers);
	WIN32PROJECT1_API int sum2(int a, int b);
}
```

win32project1.cpp:
```
// This is an example of an exported function.
WIN32PROJECT1_API int sum(int num, int* numbers)
{
	int sum = 0;
	for (int i = 0; i < num; i++)
		sum += numbers[i];
	return sum;
}

// This is an example of an exported function.
WIN32PROJECT1_API int sum2(int a, int b)
{
	return a+b;
}
```

## Installation

*Beware, nwjs needs a rebuild of ffi and ref as in "Compiling ffi for nw"*

Install ffi, ref, ref-array:
```
npm install ffi --save 
npm install ref --save
npm install ref-array --save
```
- ffi is the library 
- ref is an helper library for working with native C types
- ref-array is an helper library for working with native C arrays

## Usage

1. In nodejs code, load the modules:
```
var ffi = require('ffi');
var ref = require('ref');
var refArray = require('ref-array');

var Int = ref.types.int;
var IntArray = refArray(Int);
```

2. Then load the DLL and its functions by their symbol (the function name) and signature ( `<return-value> [ <argument1>, <argument2>]` )
```
var myDll= ffi.Library( "Win32Project1", 
{
	"sum2": [ Int, [ Int, Int ] ],
	"sum": [ Int, [ Int, IntArray ] ]
});
```

3. Now you can use DLL functions that use basic C types with JS corresponding types easily
```
var res= myDll.sum2(13, 29);
```

## Working with buffers

When working with buffer, the best way in terms of performance is to copy the buffers/array.

Now, suppose you have a Typed Array in the form of a Int32Array, that you would like to pass to your DLL functions. For example:
```
var int32Array= new Int32Array(num);
for(let i= 0; i < num; i++) int32Array[i]= i+1;
```

As ref-array works with node Buffer, we need to create a Buffer that is not a copy int32Array, but int32Array and the Buffer shall share the same memory, with no copy. To do that:
```
// From int32Array to Buffer (no copy)
var buf= Buffer.from(int32Array.buffer);
```

Then we need to create a intArray that shares the memory with buf:	
```
// From Buffer to IntArray (no copy)
var intArray= new IntArray(buf);
```

Now the function using the array can be called:
```
var res= myDll.sum(intArray.length, intArray);
```

Creating Buffer and intArray is efficient, and calling myDll.sum is super efficient too.

In the whole the less efficient code is `for(let i= 0; i < num; i++) int32Array[i]= i+1;` for num >> 10000.

# Compiling a MarkDown file to html
```
npm install -g markdown-it
markdown file.md -o file.html
```

