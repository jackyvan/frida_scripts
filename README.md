# Tiny Frida Scripts


 
### Writing Objective-C Frida Scripts
##### Setup
 I used `WebStorm` from Jetbrains to write `Frida Scripts`.  Other people wrote scripts in `Python` and passed in the `Javascript`.  I liked `WebStorm` as:
 
  - Immediate Javascript `syntax feedback`
  - You got `auto-complete` for `Frida`
 
##### Enable auto-complete
![](.README_images/webstorm_setup_frida_autocomplete.png)

### Tips for writing Frida Scripts
##### Find function / methods Address and Module

 - To find a `Objective-C method` you can use: `ObjC.classes.NSString.$ownMethods`
 - To find a `C function` you can use: `DebugSymbol.fromAddress(Module.findExportByName(null, 'strstr'))`

##### Reference Counting
The Frida creator said: _"Never interact with Objective-C APIs without an `autorelease-pool`."_
 
##### Special Properties
 -  `foobar` has special properties `const foobar = new ObjC.Object(retval)`:
 
    -`foobar.$className`
 
    -`foobar.$moduleName`
 
    -`foobar.$kind`
    
    ......and more

##### Reading C Strings can throw errors
When dealing with `C` character arrays, `Memory.readUtf8String(args[1]);` can `throw` a Javascript error. For example:

`Error: can't decode byte 0xda in position 2 at /repl19.js:25`.

You can use: `Memory.readCString(args[1], 20)` to avoid this.  You can even limit the size of the read with an ( optional ) size value.

Or you can handle the error:
```
try {
    this._needle = Memory.readUtf8String(args[1]);
}
catch(err){
    nonDecoableChars++;             // this._needle == Javascript's undefined type
}
```
##### `lldb` convenience arguments differ to `frida`
For example:
 
 ```
-[NSString containsString:]

 -lldb  ---------------------------------

(lldb) po $arg1
haystack

(lldb) po (char *)$arg2
"containsString:"

(lldb) po $arg3
needle

-frida ---------------------------------
   Interceptor.attach(methodPointer, {
       onEnter: function (args) {
        this._needle = new ObjC.Object(args[2]);
 ```

##### Persist instance variables between onEnter() and onLeave()
```
onEnter: function (args) {
    this._needle = new ObjC.Object(args[2]);


onLeave: function (retval) {
    if(this._needle != '-') {
        // do something
    }
```
           

##### Cheat sheets
https://github.com/iddoeldor/frida-snippets