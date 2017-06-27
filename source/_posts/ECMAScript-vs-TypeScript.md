---
title: ECMAScript vs TypeScript
date: 2017-06-26 23:00:36
tags:
---

I was tasked recently to investigate a tech upgrade, for an existing system that started about 5 years ago.  
That included various aspects of the system, however I will only be focusing on the language investigation for the purpose of this post.

## App initial notes:  
* It was one of the company's first SPAs
* Written in TypeScript _version-very-old_
* All the devs code in Visual Studio (server side as well) for Windows

Keep in mind that I don't know much of this system's critical domain. Also I have worked on a fair spread of apps using TypeScript / ECMAScript 2015 / good 'ol JavaScript (ES5). So hopefully that counts in my favour of having an unbiased opinion, initially, at least...

## New scaffold

After deciding on the framework, I set of to start a new app full of piss and vinegar.  
Almost immediately I was faced with the question: _ES2015 or TypeScript ?_

No problem, let's go with the current app's language and stick with TypeScript. After all that should make our lives somewhat easier by just copying the current files and leaving their format as `*.ts`. Sure we still have to adapt some of the actual code later on, but for now let's just get it to transpile.  

Now if you have ever tried to upgrade any application's framework / runtime engine / target platform, which is ~ 5 years out of date, you will know this is never just a simple flip of the switch. The rabbit holes are deep and dark, but eventually you might find some light at the end of that tunnel.  I was then left with a couple of broken / commented out references to be included.

## 3rd Party Libraries

Right, scaffold complete. Next up: include some packages.  
Almost all applications and packages has some dependencies on other libraries, and should really use some kind of package manager or CDN today. This however, was not commonly used or available a few years back, with most packages simply checked in with their source control.

We have since come a long way with the help of Node & [npm](https://www.npmjs.com/), and I was going to take full advantage of that.  
On previous projects I have used [tsd](https://github.com/DefinitelyTyped/tsd), which now had a big warning: **DEPRECATED: TSD is deprecated, please use [Typings](https://github.com/typings/typings)**. It was never my favourite package manager, with a lot of packages missing or often out-of-date definition files, so no tears shed.  

Fair enough, on to [Typings](https://github.com/typings/typings) it is then. However this **_also_** has a deprecation notice regarding TypeScript@2.0 and stating: _...some definitions on DefinitelyTyped may not work with the Typings approach because of new TypeScript features..._. Red flags raised.  

## ECMAScript 2015 Prototype

At this point I was exhausted and thought about possibly converting over to ES2015 JavaScript. The syntax is mostly the same and you get most of the benefits as you would in TypeScript with the tools available today. Obviously you lose the strict typing (except for types declared with `any` which this project had plentiful), but then you also gain the dynamic flexibility (which I personally really like).  

As a prototype, I updated the scaffold for ES2015 and used the [Babel transpiler](https://babeljs.io/). After a quick copy-paste & minimal find-replace, the code was actually compiling. Not running yet, just successfully transpiled. Progress at last.  

**Package manager round 2:** simple. `npm install` those bad-boys, configure some modules, and it was done. A working upgraded example we can immediately continue working on, but would this actually be acceptable for the team?  
Initially during the demo, some team-members didn't even notice the language change. Most seemed unfazed with this change, others actually excited, and a couple understandably concerned.

## Conclusion  

Eventually the move to ES2015 was accepted and embraced by the team. With the preferred editor now being VS Code, some can even do away with full Visual Studio and Windows if they prefer. They already have the knowledge to continue with the proper ECMAScript standards, and existing good procedures (PR & QA) in place.  

Debugging TypeScript has always been another massive pain. For an established existing application, it's preferred to have browser debugging available when having to make domain functionality changes. In the end, ease and practicality outweighed the benefit of type-safety.  

Having a look at the [ECMAScript 2015 Compatibility Table](http://kangax.github.io/compat-table/es6/), we could probably start doing away with ES5 completely in the near future. Deprecated package managers and incompatible Typings features, causing builds to fail and multiple headaches, could be the start of TypeScript's death.