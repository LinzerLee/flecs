# FAQ
Frequently asked questions.

## What is an ECS?
See the [ECS FAQ](https://github.com/SanderMertens/ecs-faq)

## Why is Flecs written in C?
There are a lot of reasons, but the main ones are:
- Faster compile times, especially when compared with header-only C++ libraries
- C can be called from almost any programming language
- C is super portable and has a tiny runtime so you can run it on a toaster if you want
- The rules and limitations of the ECS aren't dictated by any type system
- You can create a zero-overhead C++ API on top of C, but not the other way around

## Can I use Flecs with C++14 or higher?
You can! Even though the C++ API is C++11, you can use it with any revision at or above 11.

## Can I use std::vector or other types inside components?
You can! Components can contain almost any C++ type.

## What is an archetype?
Archetype-based refers to the way the ECS stores components. It means that all entities with the same components are stored together in an archetype. This provides efficient CPU cache utilization and allows for things like vectorization and fast querying.

Other examples of archetype implementations are Unity DOTS, Unreal Mass and Bevy ECS.

For more information, see this blog: https://ajmmertens.medium.com/building-an-ecs-2-archetypes-and-vectorization-fe21690805f9

## How does Flecs compare with EnTT?
Flecs and EnTT both are ECS libraries, but other than that they are different in almost every way, which can make comparing the two frameworks tricky. When you are comparing Flecs and EnTT, you can _generally_ expect to see the following:

- Add/remove operations are faster in EnTT
- Single component queries are faster in EnTT
- Multi-component queries are faster in Flecs
- Bulk-creating entities are faster in Flecs
- Entity destruct are faster in Flecs (especially for worlds/registries with lots of components)
- Iterating a single entity's components are faster in Flecs

When doing a benchmark comparison don't rely on someone elses numbers, always test for your own use case!

## Is Flecs used for commercial projects?
Yes, Flecs is used commercially.

## Why are my queries so slow?
This is likely because you're creating a query in a loop. Queries should be created once, and iterated often.

## Can Flecs be compiled to web assembly?
Yes it can! See the [quickstart manual](Quickstart.md) for more information.

## Why am I getting an “use of undeclared identifier 'FLECS__E …’” compiler error?
This happens in C if the variable that holds the component id can't be found. This example shows how to fix it: https://github.com/SanderMertens/flecs/tree/master/examples/c/entities/fwd_declare_component

## What is the difference between add & set? Why do both exist?
An `add` just adds a component without assigning a value to it. A `set` assigns a value to the component. Both operations ensure that the entity will have the component afterwards.

An `add` is used mostly for adding things that don't have a value, like empty types/tags and most relationships. If `add` is used with a component that has a constructor, adding the component will invoke its constructor.

Additionally you can use `emplace` to construct a component in place in the storage (similar to `std::vector::emplace`).

## Can Flecs serialize components?
Yes it can! See the reflection examples:

- https://github.com/SanderMertens/flecs/tree/master/examples/c/reflection
- https://github.com/SanderMertens/flecs/tree/master/examples/cpp/reflection

## Why is Flecs so large?
If you look at the size of the flecs.c and flecs.h files you might wonder why they are so large. There are a few reasons:

- The files contain a _lot_ of comments and in-code documentation!
- Flecs has a small core with a lot of addons. The flecs.c and flecs.h files are the full source code, including addons.
- The flecs.h file contains the full C++ API.
- Flecs implements its own data structures like vectors and maps, vs. depending on something like the STL.
- C tends to be a bit more verbose than other languages.

Not all addons are useful in any project. You can customize a Flecs build to only build the things you need, which reduces executable size and improves build speed. See the quickstart on how to customize a build.

## Why does the explorer not work?
See the troubleshooting section of: https://github.com/SanderMertens/ecs_benchmark/blob/master/README.md

If it's still not working, check the output of the browser console and post it in an issue or on Discord!

## How do I detect which entities have changed?
Flecs has builtin change detection. Additionally, you can use an `OnSet` observer to get notified of changes to component values. See the query change detection and observer examples for more information.

## Are relationships just a component with an entity handle?
No, relationships are a deeply integrated feature that is faster in many ways than a component with an entity handle. See the [relationship manual](Relationships.md) for more information.

## Can I create systems outside of the main function?
You can! Systems can be created from any function, except other systems.

## Can I use my own scheduler implementation?
You can! The flecs scheduler (implemented in the pipeline addon) is fully optional and is built on top of Flecs. You can create a custom pipeline, or a custom schedule implementation that does something else entirely than what Flecs provides.

## Can I use Flecs without using systems?
You can! Systems are an optional addon that can be disabled. You can build applications that just use Flecs queries.

## Why does the lookup function not find my entity?
This is likely because the entity has a parent. A lookup by name requires you to provide the full path to an entity, like:

```c
ecs_lookup_fullpath(world, "parent.child");
```

or in C++:

```cpp
world.lookup("parent::child");
```

## Can I add or remove components from within a system?
You can! By default ECS operations are deferred when called from inside a system, which means that they are added to a queue and processed later when it's safe to do so. Flecs does not have a dedicated command API, if you call an operation from a system it will be automatically added to the command queue!
