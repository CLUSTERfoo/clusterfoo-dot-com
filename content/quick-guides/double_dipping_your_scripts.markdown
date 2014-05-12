---
title: Reusing Javascripts Across Browser and Server
kind: article
created_at: 2014-05-12
category: quick-guide
---

<!-- _. -->

The following method will allow you to reuse the same javascript file across
multiple environments:

1. As an npm/bower package.
2. As a drop-in browser script.
3. As a node.js module, using `require`.
4. As a Browserify module, using `require`.

We'll start with the follwing script. It's made up of two utility functions,
`foo` and `bar`:

### my_module.js

    function foo() { console.log("Foo!"); }
    function bar() { console.log("Bar!"); }
{: class="language-js"}

## Step 1: Preparing our script for the browser.

First, we must prepare our script for the browser. I like to wrap and namespace
all my browser scripts using my unique github handle.
For an in-depth explanation of what's going on,
<a href="/quick-guides/namespacing_your_crap"> check out my guide on namespacing
javascripts</a>.

### my_module.js

    ;(function() {
        // Create an object to contain all that we wish to export.
        var C = {};

        // Alias "C" to our namespace obejct.
        appendToNamespace("Clusterfoo", "myModule", C)
        function appendToNamespace(namespace, elementName, elementValue) {
            // ... see namespacing guide
        }

        // This is how we export our public functions.
        C.foo = foo;
        function foo() { console.log("Foo!"); }

        C.bar = bar;
        function bar() { console.log("Bar!"); }

        // All other functions remain private:
        function baz() { console.log("No way to call me!"); }
    })();
{: class="language-js"}

That's all we need for the browser. The script is ready to use:

### my\_page.html

    <html>
        <head>
            <script src="my_module.js"></script>
        </head>
        <body>
            <script>
                // initialize my namespace
                Clusterfoo();
                // use functions
                Clusterfoo.myModule.foo(); // -> "Foo!"
                Clusterfoo.myModule.bar(); // -> "Bar!"
            </script>
        </body>
    </html>
{: class="language-html"}

## Step 2: Preparing Script for Node / Browserify

Next, we prepare our script so that it is compatible with the `require` API used
in node.js and browserify. This requires a bit more trickery, but nothing too
complicated. All we care about is dealing with either a `window` object or an
`exports` object:


### my_module.js

    // Our wrapper function now accepts two arguments.
    ;(function(window, exports) {
        var C = {};

        appendToNamespace("Clusterfoo", "myModule", C)
        function appendToNamespace(namespace, elementName, elementValue) {
            // ... see namespacing guide
        }

        // Attach our functions to either the namespace object
        // or the exports object.
        exports.foo = C.foo = foo;
        function foo() { console.log("Foo!"); }

        exports.bar = C.bar = bar;
        function bar() { console.log("Bar!"); }

    // If window/exports are undefined, simply pass in empty objects.
    })(typeof window !== "undefined" ? window : {},
       typeof exports !== "undefined" ? exports : {});
{: class="language-js"}

## Step 3: Using our Script

Our script is now ready! All we need to do is package it using npm and Bower:

    $ cd my_module
    $ npm init
    # npm will walk you through creating your package.

    $ bower init
    # same...

    $ git add --all && git commit
    $ git push origin master

Now all we need to do to use our script is install...

    $ npm install git://github.com/clusterfoo/my_module
    $ bower install clusterfoo/my_module

... and `require` our module (will also work in the browser if you use
browserify):

### some_script.js

    var myModule = require('my_module');

    myModule.foo() // "Foo!"
{: class="language-js"}

<a href="https://github.com/CLUSTERfoo/url-helpers">
Here's a sample module I've put together for demonstration.</a>

