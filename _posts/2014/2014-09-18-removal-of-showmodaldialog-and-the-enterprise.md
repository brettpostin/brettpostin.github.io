---
title:  "Removal of showModalDialog() and the Enterprise"
date:   2014-09-18 20:32:17 +0000
categories: Development
tags: chrome showModalDialog
---

If you weren’t already aware, Google have made the decision to remove the `showModalDialog()` API from Chrome, which came into force with version 37. You can read the announcement for the move here:

[Disabling showModalDialog](http://blog.chromium.org/2014/07/disabling-showmodaldialog.html)

And a discussion on the issues here:

[Intent to Remove: window.showModalDialog()](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/xh9fPX0ijqk%5B1-25-false%5D)

In addition to this, Mozilla have also indicated that they are to drop the API, so we better get used to it.

Whilst I understand the justifications for the removal, that doesn’t help us developers who work and maintain enterprise scale applications that make heavy use of this API.

At my company we have a significant amount of legacy code that is reliant on the blocking nature of `showModalDialog()`. Whilst we have centralised the calls to `showModalDialog()` down to 3 lines of code, we extensively rely on this code to provide feedback from modal user prompts (a quick search of the solution reveals around 2400 instances).

We could rip out `showModalDialog()` fairly easily and replace it with a Javascript/css based alternative, that’s not a problem. The issue we face is that all of the calling code will no longer be blocking/synchronous. For example…

```javascript
if(doConfirm(...)) {
    ...
} else {
    ...
}
```

… will simply fall through due to the introduction of a non-blocking alternative.

We also cannot use the in-built blocking methods (`alert`, `confirm`) as the dialog buttons are customised in many cases and are also styled to fit in with our application.

After much searching and many tears I eventually came to the conclusion that refactoring large swathes of code was inevitable. However in order to minimise the amount of work involved we employed a number of techniques that others may find useful. Note that cross-browser compatibility and future proofing were pre-requisites for the eventual solution outlined below.

# \<dialog\> Polyfill

The `<dialog>` element is part of the HTML5 specification and is designed to allow developers to provide popups and modal dialogs in their web applications. However [browser implementation support](http://caniuse.com/#feat=dialog) is currently limited.

So our first step was to introduce a [\<dialog\> polyfill](https://github.com/GoogleChrome/dialog-polyfill).

# showModalDialog() Polyfill

The next step was to introduce a customised version of this [showModalDialog() polyfill](https://github.com/niutech/showModalDialog).

This polyfill makes use of the `<dialog>` element and Promises to replicate the behaviour of `showModalDialog()`. In order to keep the synchronous nature of the code however, you need to leverage ECMAScript 6 Generators.

# Async/Await

Whilst the use of generator functions and the yield keyword provide the synchronous behaviour we were looking for, it still involved a lot of refactoring.

We eventually decided on adopting a proposed ECMAScript 7 feature, **async/await**. You can read more [here](http://jakearchibald.com/2014/es7-async-functions/).

In short, the async/await keywords are syntactic sugar over generators, the yield keyword and a spawn function.

As a result the amount of refactoring work is reduced, going from this:

```javascript
function doDomething() {
    return spawn(function *() {
        if(yield doConfirm(...)) {
            ...
        } else {
            ...
        }
    });
}
```

To this:

```javascript
async function doDomething() {
    if(await doConfirm(...)) {
        ...
    } else {
        ...
    }
}
```

# Traceur Compiler

The good news is that the traceur compiler brings Promises, Generators and the Async/Await features to other browsers by compiling them down to ECMAScript 5.

If you can precompile your source code then you simply need to include the traceur runtime in your pages. However if like us this is not possible, you must include the compiler which comes as a hefty **~500kb dependency**.

As an enterprise scale web application we decided to take the hit on this with the plan to remove it once the required features become more widely supported.

# Conclusion

The solution proposed above no doubt has its downsides. However when faced with such a large and immediate threat it offers an alternative to rewriting large portions of synchronous code to be asynchronous.

It also makes use of ECMAScript features that are strong candidates for future specifications. These will hopefully gain native browser support, potentially avoiding the need for future refactoring pain.