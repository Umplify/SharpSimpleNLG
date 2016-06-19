# Stuff encountered on the simplenlg journey to Csharp

## Can't just rename from .java to .cs and recompile
* like, der. don't know why you would expect this to just work
* process was made easier by flattening the namespace hierarchy into one
* good Tests meant that quality could be checked at the end
* both languages as being typed, C- curly braced languages with many common features made the work easier (than say, a Python project)

## Time Budget
* 3.5 days for 20KLOC across 100 files to "0 error build" (rename .java to .cs, fix all the textual stuff)
* 0.5 days for additional "glue" code (Extension methods) and Unit tests
* 0.5 days for build to actually pop out something sensible (fixing the big picture issues)
* 3.5 days of Unit tests (~270) working, finding and fixing lower-level idiomatic Java style issues eg: Regex
* Estimate of 2KLOC per day

This project had no external library dependencies (apart from JUnit) and used relatively well-known Java idioms and mechanisms. Taking something more complex and larger would probably blow this budget somewhat.

As a key part of this project is a 1Mb XML data file [default-lexicon.xml](https://github.com/nickhodge/SharpSimpleNLG/blob/master/SharpSimpleNLG/lexicon/default-lexicon.xml) -and- the initial use-case is a client-side application, using [IKVM](https://www.ikvm.net/) with its additional download weight was not an option.

## Enums are objects in Java 
* which means that enums can be subclassed
* when using as Keys in ```Dictionary<K,V>``` (expando-objects) all is well. Cannot do this in CSharp as they are not object types!
* had to put wrap Enums (given unique ranges of Int values) into an object hierarchy; and use .ToString() on object to get a Key
* in a future version, would just use the Int values as keys for faster processing

## Regex is slightly different
* .NET doesnt support [character class subtraction](http://www.rexegg.com/regex-class-operations.html#intersection_workaround) the same was as Java
* so ```.*[b-z&&[^eiou]]y\b``` becomes ```.*[b-z-[eiou]]y\b```

## Extension methods in C# rock
* see [HelperExtensions](https://github.com/nickhodge/SharpSimpleNLG/blob/master/SharpSimpleNLG/helperextensions/HelperExtensions.cs)
* rather than changing lots of Java code to C#, just created Extension methods.
* for instance, Java ```list.get(0)``` maps to ```list[0]``` in C# without replacement
* if you do need to replace/fix in the future, you can look at the backreference from the Extension method to fix.

## But be careful with Semantics
* Java .put will create a new HashMap KV pair, or replace the Value if the key already exists
* In C#, you have to write around this - so the .put Extension method takes this into account
* Changing ```foreach``` over an Interable seems easy enough (and easier in C#!) but semantics of this over a ```Stack<T>``` is different in C# (first vs. last!) Solved by .ToList().Reverse()
* big difference in Java string.substring vs. C# string.SubString (length vs. position!)
* reading the Java docs, and ensuring the Extension methods follow the meanings became important for some deep bugs

## Object hierarchy calling convention
I must admit a little bit of defeat on this one. (this is for later research)

SPPhraseSpec inherits from PhraseElement inherits from NGElement which implements INGElement

Both SPPhraseSpec and NGElement have a ```.setFeature(object)``` method, but with different functions. In many places, a straight INGElement is passed around - and when you call .setFeature on an SPPhraseSpec cast as INGElement - the lower level class' method is called.

it seems this is not the case in Java?

This involved some checking and casting to work around this (and this fixed many a bug found in Testing)

## Resisting the Urge to LINQ and FUNC<> everything

I am glad I spent the time getting the base working, with Tests working - before attempting to optimise or start to use more modern C# style coding. 