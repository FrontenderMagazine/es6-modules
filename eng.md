# ECMAScript 6 modules: the future is now

This blog post first explains how modules work in ECMAScript 6, the next 
version of JavaScript. It then describes tools that allow you to already 
use them now.

## Module systems for current JavaScript

JavaScript does not have built-in support for modules, but the community has 
created impressive work-arounds. The two most important (and unfortunately 
incompatible) standards are:

- CommonJS Modules (CJS): The dominant incarnation of this standard is [
Node.js modules][1] (Node.js modules have a few features that go beyond CJS). 
Characteristics:

    - Compact syntax

    - Designed for synchronous loading

    - Main use: server

- Asynchronous Module Definition (AMD): The most popular implementation of this 
standard is [RequireJS][2]. Characteristics:

    - Slightly more complicated syntax, enabling AMD to work without eval() 
    (or a compilation step).

    - Designed for asynchronous loading

    - Main use: browsers

The above is but a simplified explanation of the current state of affairs. If 
you want to read more in-depth material, take a look at 
“[Writing Modular JavaScript With AMD, CommonJS & ES Harmony][3]” by Addy Osmani.

## ECMAScript 6 modules

The goal for [ECMAScript 6 (ES6) modules][4] was to create a format that both 
users of CJS and of AMD are happy with. To that end, their syntax is as compact 
as CJS. On the other hand, they are less dynamic than CJS (e.g., you can’t 
conditionally load a module with normal syntax). That has two main advantages:

- You get compile time errors if you try to import something that has not been 
exported.

- You can easily load ES6 modules asynchronously.

The ES6 module standard has two parts:

- Declarative syntax (for importing and exporting).

- Programmatic loader API: to configure how modules are loaded and to 
conditionally load modules.

## ECMAScript 6 module syntax

ECMAScript 6 modules look very similar to Node.js modules. A module is simply a 
file with JavaScript code in it. As an example, take the following project, 
whose files are stored in a directory calculator/.

        calculator/
            lib/
                calc.js
            main.js

### Exporting

If there is something you want others to use, you export it, by prefixing the 
keyword export to a variable declaration (via var, let, const), a function 
declaration or a class declaration<a href="#note-1" class="reference">1</a>. 
calculator/lib/calc.js contains the following text:

        // calculator/lib/calc.js
        let notExported = 'abc';
        export function square(x) {
            return x * x;
        }
        export const MY_CONSTANT = 123;

The above module exports the function square and the value MY_CONSTANT.

### Importing

main.js is another module and it imports square from calc.js:

        // calculator/main.js
        import { square } from 'lib/calc';
        console.log(square(3));

main.js refers to calc.js via the module ID 'lib/calc' (a string). The default 
interpretation of the ID is as a path relative to the importing module. Note 
that you can import more than one value if you want to:

        // calculator/main.js
        import { square, MY_CONSTANT } from 'lib/calc';

Alternatively, you can import the module as an object and access the exports 
via properties:

        // calculator/main.js
        import 'lib/calc' as c;
        console.log(c.square(3));

If you are unhappy with the name that an exporting module has chosen, you can 
rename locally:

        // calculator/main.js
        import { square as squ } from 'lib/calc';
        console.log(squ(3));

### Default exports

Sometimes a module only exports a single value (for example, a large class). 
Then you can make that value the default export:

        // myapp/models/Customer.js
        export default class { // anonymous class
            constructor(id, name) {
                this.id = id;
                this.name = name;
            }
        };

The syntax for importing a default export is similar to normal importing, but 
there are no braces (as a mnemonic, you are not importing something from inside 
the module, you are importing the module):

        // myapp/myapp.js
        import Customer from 'models/Customer';
        let c = new Customer(0, 'Jane');

### Inline modules

Sometimes you don’t want a module to occupy a complete file. One use case is 
packaging several modules in a single file. Such a multi-module file looks as 
follows and would be loaded as a script.

        module 'foo/m1' {
            …
        }
        module 'foo/m2' {
            …
        }
        module 'foo/m3' {
            …
        }

Another use case is to prevent variables from becoming global. For example, a 
current best practice is to use an IIFE<a href="#note-2" class="reference">2</a> 
for this purpose:

        <script>
            (function () {  // open IIFE
                var tmp = …;  // won’t become global
            }());  // close IIFE
        </script>

In ECMAScript 6, you can use an anonymous inner module:

        <script>
            module {  // anonymous inner module
                let tmp = …;  // won’t become global
            }
        </script>

<<<<<<< HEAD
Aside from being syntactically simpler, using a module in this manner has the advantage that the innards are automatically in strict mode<a href="#note-3" class="reference">3</a>.
=======
Aside from being syntactically simpler, using a module in this manner has the 
advantage that the innards are automatically in strict 
mode<a href="#note-3" class="reference">3</a> .
>>>>>>> 66b5637400a123cbac4f881135e27b1bfe1e4f96

Note that you do not need to be inside a module in order to import things. 
The import declaration can be used in normal script context. 

### An alternative to inlined exports

If you don’t want to insert exports in your code, you have the option of 
exporting everything later, e.g. at the end:

        let notExported = 'abc';
        function square(x) {
            return x * x;
        }
        const MY_CONSTANT = 123;

        export { square, MY_CONSTANT };

You can also rename while exporting:

        export { square as squ, MY_CONSTANT as SOME_CONSTANT };

### Re-exporting one’s imports

You can re-export things from another module:

        export { encrypt as en } from 'lib/crypto';

You can also re-export everything:

        export * from 'lib/crypto';

## ECMAScript 6 module loader API

In addition to the declarative syntax for working with modules, there is also 
a [programmatic API][5]. It allows you to do two things: programmatically 
working with modules and scripts and configuring module loading.

### Importing modules and loading scripts

You can programmatically import modules, with a syntax reminiscent of AMD 
modules:

        System.import(
            ['module1', 'module2'],
            function (module1, module2) {  // success
                …
            },
            function (err) {  // failure
                …
            }
        );

Among other things, this enables you to conditionally load modules.

System.load() works similarly to System.import(), but loads script files 
instead of importing modules.

### Configuring module loading

The module loader API has various hooks for configuration. A few examples of 
what they allow you to do:

- Customize how module IDs are mapped to modules.

- Lint modules on import (e.g. via JSLint or JSHint).

- Automatically translate modules on import (they could contain CoffeeScript or 
TypeScript code).

- Use legacy modules (AMD, Node.js).

You’d have to implement these things yourself, but the hooks for them are there.

## Using ECMAScript 6 modules today

The two most recent projects enabling you to use ECMAScript modules today are:

- [ES6 Module Transpiler][6]: write your modules using a subset of ECMAScript 6 
(roughly: ECMAScript 5 + export + import), compile them to AMD or CommonJS 
modules. A [blog post][7] by Ryan Florence explains this approach in detail.

- [ES6 Module Loader][8]: polyfills the ECMAScript 6 module loader API on 
current browsers. To enter the world of modules, you use the API:

            System.baseURL = '/lib';
            System.import('js/test1', function (test1) {
                test1.tester();
            });

    In actual modules, you use ECMAScript 5 + export + import. For example:

            export function tester() {
                console.log('hello!');
            }

Other possibilities:

- [require-hm][9]: a plugin for RequireJS allowing it to load ECMAScript 6 
modules (only ECMAScript 5 plus importing and exporting is supported). 
A [blog post][10] by Caolan McMahon explains how it works. Warning: uses an 
older module syntax.

- [Traceur][11] (an ECMAScript 6 to ECMAScript 5 compiler): has partial support 
for modules and may eventually support them fully.

- [TypeScript][12] (roughly: ECMAScript 6 plus optional static typing): 
compiles modules in external files (which can use most of ECMAScript 6) to 
AMD or CommonJS.

## Further reading

- The specification of ECMAScript 6 modules: Modules are not yet in the 
[draft ECMAScript 6 specification][13]. Until they are, consult the 
[Harmony wiki][14] for details.

- “[ES6 Modules][15]” by Yehuda Katz: a discussion of common use cases and 
interoperability with existing module systems.

## References

<a href="#note-1" id="note-1" class="reference">1</a> [ECMAScript.next: classes][16]  
<a href="#note-1" id="note-2" class="reference">2</a> [JavaScript quirk 6: the scope of variables][17]  
<a href="#note-1" id="note-3" class="reference">3</a> [JavaScript’s strict mode: a summary][18]  


[1]: http://nodejs.org/api/modules.html
[2]: http://requirejs.org/
[3]: http://addyosmani.com/writing-modular-js/
[4]: http://wiki.ecmascript.org/doku.php?id=harmony:modules
[5]: http://wiki.ecmascript.org/doku.php?id=harmony:module_loaders
[6]: https://github.com/umdjs/es6-module-transpiler
[7]: http://ryanflorence.com/2013/es6-modules-and-browser-app-delivery/
[8]: https://github.com/ModuleLoader/es6-module-loader
[9]: https://github.com/jrburke/require-hm
[10]: http://caolanmcmahon.com/posts/try_harmony_modules_today/
[11]: https://github.com/google/traceur-compiler
[12]: http://www.typescriptlang.org/
[13]: http://wiki.ecmascript.org/doku.php?id=harmony:specification_drafts
[14]: http://wiki.ecmascript.org/doku.php?id=harmony:modules
[15]: https://gist.github.com/wycats/51c96e3adcdb3a68cbc3
[16]: http://www.2ality.com/2012/07/esnext-classes.html
[17]: http://www.2ality.com/2013/05/quirk-variable-scope.html
[18]: http://www.2ality.com/2011/01/javascripts-strict-mode-summary.html
