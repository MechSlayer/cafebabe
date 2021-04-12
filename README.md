cafebabe
========

cafebabe is a parser for .class files, which are generated by the javac compiler and other compilers targeting the JVM.
It supports the class file format from Java 16 (March 2021).

There are a bunch of different class file parsers available on crates.io. Main differentiators for `cafebabe` are:
- *Support for Java 16.*
  As of this writing, others generally support Java 8 / Java 11.
- *A better user API*.
  Most other parsers expose the raw u16 values of the constant pool, which means you as the user end up being responsible for doing the constant pool lookup.
  This also means you have to do type validation (e.g. make sure the constant pool entry you're reading is actually a classinfo type, or whatever).
  `cafebabe` does all this internally and returns the "primitive" type you get after doing the constant pool lookups.
  Consider the example of reading a classinfo member.
  With most other parsers, you get a u16 from the parser, with which you have to:
  (a) do a constant pool lookup,
  (b) verify it's a classinfo type,
  (c) extract the u16 from the classinfo,
  (d) do another constant pool lookup,
  (e) verify it's a utf8 type,
  (f) read the string.
  With `cafebabe` you just get the string right off the bat, since all the type checking and lookups were done during the parsing phase.
- *Zero-copy of string data*.
  All strings are returned as `Cow<'a, str>` which are tied to the lifetime of your input class bytes.
  This means there's zero copying for strings anywhere in `cafebabe` itself, except in the rare case where a string is in the subset of modified "java utf-8" that isn't regular utf-8.
- *Minimal dependencies*.
  The crate only has a couple of dependencies, and neither of those pull in any transitive dependencies.

Current status
--------------
The main parsing code is fully implemented. All structures (including attributes) described in Chapter 4 of the JVM spec are supported.
The entire `modules` file of the OpenJDK 16 distribution can be parsed without errors.
`cafebabe` will do some kinds of validation/checking at parse time, but not everything described in Chapter 4 of the JVM spec.
It doesn't even do all the things described in section 4.8 ("Format checking"), although a reasonable goal for this project is to implement that part fully.

Q&A
---
*Why is the project called `cafebabe`?*

Because the first 4 bytes in any valid class file are a magic identifier with the value 0xCAFEBABE.
