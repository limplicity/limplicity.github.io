Hi Domenic,

lately a few ideas about the Loader API popped up in a project that I'm currently working on, and since the Loader API seems pretty 
important to me, I've spent some time to collect those ideas into something like a draft.

However I'm a bit shy about publishing those ideas directly to GitHub (aka. the open world), and I'd really love to hear your thoughts about 
that draft first. Please tell me if I missed something important or got something wrong!


Philosophy
----------

The Loader API is pretty important because it opens up the possibility to arbitrarily extend the behavior and capabilities of ES6 `import`. 
Therefore it should be crafted with future-proofness and extensibility in mind. Therefore it should be pretty generic and not imply too much 
detail about how `import'ing works internally (e.g. resolving, fetching, parsing, compiling, instantiating, etc.).

I think the most generic yet usable definition of `import` is as follows:

`import(src)` asynchronously returns an instantiated code module.

I don't think it's necessary to constrain the format of `src` in this definition to what the filesystem is capable of. I could also imagine 
a use case like this: `import('java:com.MyClass')` (e.g. from NodeJS somehow). This is where the Loader API comes in: It should give JS code 
a way to extend (or more generic: modify) the way that `import` works.

I'd like to propose to you the following draft:


Draft
-----

Let there be a class `Loader`, which provides a function `import` that can be used to dynamically import code.

If I'm interpreting your comment (here)[https://github.com/whatwg/loader/issues/147#issuecomment-229817849] correctly, you're already a fan 
of the idea to make the behavior of `import` immutable once it's defined.

This would imply that also the corresponding Loader object be immutable. Therefore, a new Loader object needs to be created each time we'd 
like to modify the behavior of `import`.

```
let l1 = new Loader(async (src) => {
	// ignoring `src`
		foo: 15,
		bar: x => x*x,
});

const foo = await l1.import('foo');
console.log(foo.bar(5));
```

Loaders should in some way be able to extend other (older) loaders, to retain (parts of) their functionality.

```
let l2 = new Loader(async (src) => src == 'foo'
	? { foo: 15, ... }
	: import(src)
);

const foo1 = await l2.import('foo');        // returns { foo: 15, ... }
const foo2 = await l2.import('./foo.js');   // l2 extends default loader,
                                            // therefore this also works (if ./foo.js is a valid ES6 module)
```

There is a default loader for every script file that defines the behavior of the script-local `import` (both static and dynamic imports).
Imagine we have two script files a.js and b.js with the following contents:

a.js
```
let l2 = new Loader(async (src) => src == 'foo'
	? { foo: 15, ... }
	: import(src)
);
let b = await import('./b.js', l2);
```

b.js
```
import foo from 'foo';
import bar from './bar.js';
```

a.js can define the behavior of `import` in b.js at the time that b.js is being loaded. This enables both
* extensibility: a.js can extend the behavior of `import` in b.js arbitrarily
* sandboxing: a.js can restrict access  (e.g. to prevent b.js from importing `fs` in NodeJS)


Conclusion
----------

Please let me hear what you think of this draft so far! ;)
(I know it's by far not complete, just a few ideas)

