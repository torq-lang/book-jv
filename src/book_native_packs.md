# Native Packs

Torq is a dynamic *companion language* that runs within the JVM and integrates easily with its runtime. A *Torq Pack* is a construct for declaring, bundling, and exporting items for integration. A pack contains classes, objects, and methods and makes them available for import.

## ArrayListPack

As an example object, consider the builtin Torq type named `ArrayList` that corresponds to the well-known Java `ArrayList`. The constructs in the pack are named by convention:

* The type name is `ArrayList`
* The pack name is `ArrayListPack`
* The class name is `ArrayListCls`
* The object name is `ArrayListObj`
* Object methods begin with `obj`, such as `objSize`
* Class methods begin with `cls`, such as `clsNew`

The `ArrayList` implementation is found in the Java file `org.torqlang.local.ArrayListPack`.

## TimerPack

As an example actor, consider the builtin actor named `Timer`:

* The type name is `Timer`
* The pack name is `TimerPack`
* The actor record is `TimerPack.TIMER_ACTOR`

The `Timer` implementation is found in the Java file `org.torqlang.local.TimerPack`.

## NorthwindDbPack

In this section, we create a native pack as a wrapper for the [NorthwindDb actor](./book_northwind_database.md) created previously.
