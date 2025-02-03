---
layout: post
title:  "The wren programming language"
description: "I talk all about the work I've been doing on the side writing my own programming language, and the language I discovered along the way: Wren."
date: 2024-11-19 00:26:00 -0600
categories: blog programming-languages kona wren
excerpt: "I've been writing my own programming language, named Kona, and along the way I discoverd and really fell in love with Wren."
image: /assets/blog/the-wren-programming-language/wren.svg
image_alt: A little bird, a Wren, with the word, wren, written on it.
author: Karl Oscar Weber
---

Hi Friends.

A while ago I declared online that I really really hated Javascript. Lately, I still don't like it, but my hate just doesn't seem to go as deep. Then I declared that I would replace it with something better. In that pursuit I began working on my own language, named [Kona](https://konascript.org). Intended to be compiled to WASM, and shipped in your browser like a 200KB party favor.

Kona isn't ready yet. In fact I've barely begun the compiler/interpreter/VM. The thing about writing a programming language is that you need to know how to do that. If you don't know how to do that, well... then you can't do it. So I decided to learn.

I began by reading the book [Crafting Interpreters](https://craftinginterpreters.com), which is absolutely excellent. But before I could finish I needed to go learn **C**, A language I had never written. Then I went and I learned [Lua](https://lua.org), Thinking I could write all of Kona, in Lua, and ship it as a single binary. I got pretty far there, and learned a lot. But then I stumbled upon [Wren](https://wren.io), a small, object oriented language written by Robert Nystrom the author behind crafting interpreters.

I was immediately intrigued. It's a small, fast scripting language. comparable in speed to Lua, But with a heavily simplified object oriented syntax. Purpose built to be embedded in other applications, like Lua, but with a simpler API.

The best thing about this language, is how *EASY* it is to learn it's internals. Volumes of comments and descriptions populate the source code, making it easy to poke around and understand how it all works. In my pursuit of writing Kona, it was never my intention of building the whole thing from scratch. I mean, I want to *Finish* the project after all. I was content to start by writing Kona's interpreter in Lua, then learning C and porting it to that. Now I'm emboldened to write Kona's compiler in C, and target Wren's VM, initially, To execute the code.

Anyways, to actually ship the project I want the project to be much smaller than fully a new language. Targeting a VM with unique syntax seems like a good tradeoff. Although, as I go along, I know I'll find myself making those changes, and writing my own unique VM.

## How does a programming language work anyway?

I should explain how we get a new language at all.

Computer processors each have their own dialect of instructions. x86, RISC-V, ARM, etc... Learning to write programs for each specific architecture is arduous. So some smart folks decided that they would write a program that would transform simpler instructions, into the specific instructions of a processor. A compiler.

Compilers have quite a few parts, but the top most part, called the front end, scans code in it's particular syntax, and translates it into tokens. String, Number, Name, Operator. Then takes these tokens and translates them into the machine code of whatever computer your targeting. There are other concepts that are captured too, like scope, and upvalues, etc..

Scripting languages are a bit different. Instead of compiling the code to native machine code, scripting languages compile your code into an intermediate, but internally represented Byte Code. The scripting language then runs the byte code through it's Virtual Machine or VM, and executes the program accordingly.

The journey to get from your code, to tokens, to byte code, and then finally to be executed, offers a host of benefits or disadvantages, depending on your choice of language. New languages are made every day, or existing ones are improved. All in the pursuit to make certain tasks easier.

Consider this code:
```
class Person {
  name { _name }
  construct new(name) { _name = name }
}
var jim = Person.new("Jimmy")
// > jim
System.print(jim.name)
// > Jimmy
```

We have words, symbols, spaces, curly braces, all sorts of stuff. Along with a litany of rules and practices you'll just need to learn about the language. All in the pursuit to make our lives a bit easier. These rules and how it influences the way you write your programs, the syntax, determines what's easy, what's possible, and what's enjoyable. That is the sweet spot I'm trying to pursue.

### Tokens

The parser takes source code and converts it to tokens. Consider this:
```
var thing = "whatever"
```

Which, when parsed, produces the following tokens:
```
[  VARIABLE] => var 0..2
[      NAME] => thing 4..8
[ASSIGNMENT] => = 10..10
[    STRING] => whatever 12..21
```

Now, Wren's compiler is a one-pass compiler. That means these intermediate tokens are never really created, and Byte Code is emitted directly. Wren's compiler has a single token look-ahead, look-behind, So it only really knows what's immediately ahead and behind a token. This severely limits certain syntax possibilities, but in turn, it makes the parser very simple.

Consider this structure:
```
class Nuts {
  go { _nuts }
  go=(v) { _nuts = v }
  construct new() {}
}
```

We define a class, using a keyword **class**, and add some setters, getters, and a constructor. In wren, the method signature is part of it's name, and all properties, called fields, are private, so you need to add setters and getters.

Instead of marking the methods with `def` or `func` or something else, they are simply Naked `names` with blocks after them. `name + block` is a simple pattern for making a method. Now let's call some methods:
```
var nutty = Nuts.new()
nutty.go
// > null
nutty.go = "Nuts!"
nutty.go
// > Nuts!
```

Here we show some more goodies. A var keyword to tell the compiler that nutty is a new variable. The compiler emits bytecode to create that space, and then later it's assigned. Because the Wren compiler is a single pass, with only limited look ahead, we need to emit these tokens:
```
[   VARIABLE] => var 0..2
[       NAME] => nutty 4..8
[ ASSIGNMENT] => = 10..10
[      CLASS] => Nuts 12..15
[        DOT] => . 16..16
[       NAME] => new 17..19
[ LEFT_PAREN] => ( 20..20
[RIGHT_PAERN] => ) 21..21
```

Now whitespace is usually ignored, so we don't add those, but we do want to know what each token is. You can tell from the first token and the second, that a `VARIABLE` declaration immediately followed by a `NAME`. This means that we want to set aside some storage. The bytecode to do that is then emitted. The next token is `ASSIGNMENT`, a binary operator. It takes the next `STATEMENT`, and assigns it to the name that came before. At this point we know that what comes next needs to be a valid statement, so the compiler begins interpreting the next tokens assuming this. If it runs into an unexpected situation, it can emit an error.

It may not seem immediately recognizable, but a single pass compiler means that certain features just aren't feasible, or pretty difficult. Additionally, to keep the syntax simple, certain other features are not as easy. Consider free floating functions:
```
myFunction()
```

In an object oriented language like Wren, we have a name token, and an opening and closing paren token. The name is resolved at runtime, and the following tokens become part of the name to make an invocation. But what is it being invoked on? In Wren the parameter list, or at least it's arity, are part of a methods signature, so a free floating function, as an object, would have to be implicitly called against `this`, like other naked methods are. But what is _this_? If you call `myFunction()` inside of a method in a class or outside in a free space what is the value of `this`? It's different in each situation.

This is where the design of the language and it's syntax begin to have consequences. What do we want to make easy in a language? And what are the consequences for making something easy.

Anyways. I really like Wren, and I'm excited to see where that language goes. It is a bit restricted from what I'd like to do. Classes can't be reopened, method and function calling is explicit, which is nice, but restricts certain patterns. The compiler is single pass which means that complicated features like optional parens are not feasible.

## My Goal

Kona is intended for building UIs and interfaces on the web. A general purpose programming language, sure, but focused on a single platform. That focus means that syntax limitations, and in effect the design of the language, needs to be reconsidered at every step. i'm aiming for a multi-pass compiler to give some more complicated syntax features. Built in JSON, and HTML parsing. Built in HTML templates, Builders, class reopening, functions as first class citizens, and object tree diffing.

The target is the web, a Programming language VM as a WASM module that listens to and interacts with the webpage through the available APIs. We'll see how far I get.

-kow
