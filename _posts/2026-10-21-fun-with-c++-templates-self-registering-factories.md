---
layout: post
title:  "Fun With C++ Templates -- Self Registering Factories"
date:   2026-10-21 14:00:00 +0530
categories: blog technical
tags: [C++, Metaprogramming, Templates, Factory Pattern]
excerpt_separator: <!--end excerpt-->
---

In this article, I talk about how I create and maintain a centralized, augmentable list of
deserialization methods for resources used by [ToyMaker.](https://raynmetal.github.io/project/personal/2026/10/14/toymaker-engine.html)

<!--end excerpt-->

You can try out my code [here, on godbolt.](https://godbolt.org/z/dx1on394s)  In case the
link expires, the same code is available in a file on my [C++ snippets github repo.](https://github.com/raynmetal/cpp-experiment-snippets/blob/main/SelfRegisteringFactory.cpp)

## The Problem

At some point, the situation I found myself in was this: I had a number of resources, each
very different from the next, that I wanted to use in my engine.  3D models, sounds, fonts,
images, etc.  Each resource, in turn, had a number of different ways in which it can be
created.  A sphere model, for example, may be [generated procedurally,](https://raynmetal.github.io/toymaker/classToyMaker_1_1StaticMeshSphereLatLong.html)
or might be [loaded from from a file.](https://raynmetal.github.io/toymaker/classToyMaker_1_1StaticModelFromFile.html)

The overall flow would be this - the game developer imports the assets they want to use for
a particular scene (by [specifying it as a JSON object in an array).](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2scene__system.html#autotoc_md70)
When a named asset is needed by a scene object, it would refer to it by its name.  The
[Resource Database](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2core_2resource__database.html)
would then dispatch a call to the appropriate constructor class for that kind of asset.
The database would need to know about every type of resource it might be asked to
construct.

A first pass at a solution might look something like this:

```c++
// file: resource_database.hpp

class ResourceDatabase {
    // ...

    enum ResourceConstructors {
        MODEL_FROMFILE,
        MODEL_FROMSPHERELATLONG,
        SOUND_FROMFILE,
        ...
    };

    std::shared_ptr<IResource> constructResource(const nlohmann::json& description) {
        switch(description.at("constructor")) {
            case MODEL_FROMFILE:
                return constructModelFromFile(description);
                break;
            case MODEL_FROMSPHERELATLONG:
                return constructModelFromLatLong(description);
                break;
            case SOUND_FROMFILE:
                return constructSoundFromFile(description);
                break;

            // ...

            default:
                assert(false && "Unrecognized resource constructor!");
        }
    };

    // ...
};
```

Which, if every kind of resource a user might ever want to describe was known ahead of
time, and this list wasn't expected to change, would be fine.

But if we're looking further ahead, there are a couple of problems.

1.  Every time an engine maintainer wants to add a new resource type, or resource
constructor, they would need to return to the `resource_database.hpp` header and modify
it.  They'd need to define the constructor in one file, and add an enum for it in the
database header.

2.  What if an app/engine plugin developer wants to specify new resource types or
constructors?  Once the engine is built, the resources and constructors it supports is
practically set in stone.  The user would _have_ to programatically call their custom
constructors, and miss out on any resource management improvements ResourceDatabase might
provide.

It would be nice if resources and constructors had a way of automatically making themselves
known to the ResourceDatabase, and if they could be defined outside the library itself.

## The Solution

Enter the [self registering factory.](https://raynmetal.github.io/toymaker/md_docs_2toymaker-engine_2core_2resource__database.html)
Through a bit of [template metaprogramming](https://www.youtube.com/watch?v=Am2is2QCvxY)
trickery, utilizing the [dynamic initialization step for static data members](https://en.cppreference.com/cpp/language/initialization)
executed as a part of a program's start up, we can make it so that resource and
constructor declarations declared in one file automatically report themselves to our
central ResourceDatabase.

### Resource Database

The ResourceDatabase (right now, just a resource constructor map) is an implementation
of the [singleton pattern.](https://en.wikipedia.org/wiki/Singleton_pattern) 
It exposes two static functions: `RegisterFactory` and `RegisterFactoryMethod` to append
a resource type or constructor to its private map of such.

```c++
class ResourceDatabase {
public:
    // returns the one (1) global instance of ResourceDatabase available on this program.
    static ResourceDatabase& getInstance() {
        static ResourceDatabase resourceDatabase {};
        return resourceDatabase;
    }

    // registers a single _type_ of resource ResourceDatabase may construct
    static void RegisterFactory (std::string name, std::unique_ptr<IResourceFactory> pFactory) {
        getInstance().mFactories[name] = std::move(pFactory);
    }

    // registers a single _constructor_ used to construct some type of resource.
    static void RegisterFactoryMethod (std::string resource, std::string method, std::unique_ptr<IResourceFactoryMethod> pFactoryMethod) {
        getInstance().mFactories[resource]->mFactoryMethods[method] = std::move(pFactoryMethod);
    }

    std::map<std::string, std::unique_ptr<IResourceFactory>> mFactories {};

private:
    ResourceDatabase() = default;
};
```

It's actually just a glorified map of resource factories, one per type.

### Registrator

There's a special support class we use to ensure that each resource type and constructor
actually calls `RegisterFactory` and `RegisterFactoryMethod` respectively.  It's called
the `Registrator`.

This is the part of the machinery that makes the automagic registration possible.  It's
basically a static global variable that _must_ be initialized for each template subclass
of the `ResourceFactory` and `ResourceFactoryMethod` classes.  Since this variable _must_
be initialized, and since we make it so that _part of its initialization procedure_ is
to _make its owner class call into_ `ResourceDatabase`, _all_ resources are registered
with the database before the first line of `main()` begins to execute.

```c++
// forces implementation of static function RegisterSelf, called here.
template<typename TRegisterable>
class Registrator {
public:
    Registrator() {
        std::cout << "Inside registrator ctor" << std::endl;
        TRegisterable::registerSelf();
    }
    void emptyFunc() {
    }
};
```

### Resource and Resource Factory

A mapping from a particular type of resource, to the various methods used for its
construction.

```c++

// Provides a concrete base class for all resources
class IResource {
public:
    virtual ~IResource()=default;
};

// Provides a concrete base class for all resource factories, stored as pointers on
// ResourceDatabase
class IResourceFactory {
public:
    virtual ~IResourceFactory()=default;
    std::map<std::string, std::unique_ptr<IResourceFactoryMethod>> mFactoryMethods {};
};

// Concrete base class for resource constructors
class IResourceFactoryMethod;

template<typename TResource>
class ResourceFactory: public IResourceFactory {
public:
    ResourceFactory() {
        std::cout << "Output of ctor" << std::endl;
    }
};


template <typename TDerived>
class Resource: public IResource {
public:
    static std::string getName () {
        return TDerived::getName();
    }
    static void registerSelf() {
        ResourceDatabase::RegisterFactory(getName(), std::make_unique<ResourceFactory<TDerived>>());
    }
protected:
    Resource(int explicitlyInitializeMe) { s_registrator.emptyFunc(); }
private:
    inline static Registrator<Resource<TDerived>> s_registrator = Registrator<Resource<TDerived>>();
};
```

Note `Resource<TDerived>`'s declaration.  When the class it declared, it also allocates
storage for a single static variable, `s_registrator`.  `s_registrator`, as part of
its instantiation, calls `Resource::registerSelf()`, here defined as:

```c++
    static void registerSelf() {
        ResourceDatabase::RegisterFactory(getName(), std::make_unique<ResourceFactory<TDerived>>());
    }
```

Since the static variable must be initialized _before_ the execution of the first line of
`main()`, all resources have resource factories known by the `ResourceDatabase`.  Here,
`StringResource` is defined in the same file as `Resource<TDerived>` -- but even if it
were defined in another file (compiler willing), it would be registered.  `registerSelf`
ends up calling `ResourceDatabase::RegisterFactory()`.

This is the magic of the `Registrator<TRegisterable>` class in action.

### Resource Constructor

```c++
// allows us to have base class pointers for resource constructors
class IResourceFactoryMethod {
public:
    virtual ~IResourceFactoryMethod()=default;
    virtual std::unique_ptr<IResource> createResource(std::string params) = 0;
};

template<typename TResource, typename TDerivedMethod>
class ResourceFactoryMethod: public IResourceFactoryMethod {
public:
    static void registerSelf() {
        ResourceDatabase::RegisterFactoryMethod(TResource::getName(), TDerivedMethod::getName(), std::make_unique<TDerivedMethod>());
    }
protected:
    ResourceFactoryMethod(int explicitlyInitializeMe) {
        std::cout << "Output of factory method constructor" << std::endl;
        s_registrator.emptyFunc();
    }
private:
    static inline Registrator<ResourceFactoryMethod<TResource, TDerivedMethod>> s_registrator = Registrator<ResourceFactoryMethod<TResource, TDerivedMethod>>{};
};
```

Each resource constructor, here `ResourceFactoryMethod<TResource, TDerivedMethod>`,
represents one way in which a resource might be constructed.  It works in the same way
as the `Resource<TDerived>` class, except its `registerSelf()` calls the
`ResourceDatabase::RegisterFactoryMethod()` function.

## Usage

### String Resources

For the demonstration, we only create one resource type, and two resource methods.  These
are the `StringResource` resource class, and `StringResourceFromString` and
`StringResourceFromInt` constructors.

```c++
// Specialization of the resource class, made automatically visible
// to the ResourceDatabase singleton by its registrator
class StringResource: public Resource<StringResource> {
public:
    StringResource(std::string params): Resource<StringResource>{0}, mResource {params} {}
    StringResource(): Resource<StringResource>{0} {}

    std::string mResource {};
    static std::string getName() {
        return "String";
    }
};

// Definition of a factory method, automatically made visible to the
// factory for StringResources
class StringResourceFromString: public ResourceFactoryMethod<StringResource, StringResourceFromString> {
public:
    StringResourceFromString() 
    : ResourceFactoryMethod<StringResource, StringResourceFromString>{0} 
    {}

    static std::string getName() {
        return "FromString";
    }

    std::unique_ptr<IResource> createResource(std::string params) override {
        std::cout << "from FromString" << std::endl;
        std::unique_ptr<StringResource> strRes { std::make_unique<StringResource>() };
        strRes->mResource = params;
        return strRes;
    }
};

// Another factory method
class StringResourceFromInt: public ResourceFactoryMethod<StringResource, StringResourceFromInt> {
public:
    StringResourceFromInt()
    : ResourceFactoryMethod<StringResource, StringResourceFromInt>{0}
    {}

    static std::string getName() {
        return "FromInt";
    }
    std::unique_ptr<IResource> createResource(std::string params) override {
        std::cout << "from FromInt" << std::endl;
        std::unique_ptr<StringResource> strRes { std::make_unique<StringResource>() };
        strRes->mResource = mStrings[std::stoi(params)];
        return strRes;
    }

    const std::vector<std::string> mStrings {"Haha", "This should", "be fun.", "(I think)", "Woohooo"};
};
```

From the top, _before even the first line of `main()` is executed._

1. The `s_registrator` of `StringResource`'s base class `Resource<StringResource>`
begins initialization.

2. `Resource<StringResource>::s_registrator` calls `Resource<StringResource>::registerSelf()`.

3. `Resource<StringResource>::registerSelf()` calls `ResourceDatabase::RegisterFactory()`.

4. `Resource<StringResource>::s_registrator`'s initialization is complete.

5. The `s_registrator` of `StringResourceFromString`'s base class
`ResourceFactoryMethod<StringResource, StringResourceFromString>` begins initialization.

6. `ResourceFactoryMethod<StringResource, StringResourceFromString>::s_registrator` calls
`ResourceFactoryMethod<StringResource, StringResourceFromString>::registerSelf()`.

7. `ResourceFactoryMethod<StringResource, StringResourceFromString>::registerSelf()` calls
`ResourceDatabase::RegisterFactoryMethod()`.

8. `ResourceFactoryMethod<StringResource, StringResourceFromString>::s_registrator`'s
initialization is complete.

9. The `s_registrator` from `StringResourceFromInt`'s base cl- _... I think you get the
idea._

### Demo Driver

Scrolling down through the source file on [godbolt,](https://en.cppreference.com/cpp/language/initialization)
you'll encounter the driver program that demonstrates this interesting piece of
functionality.

```c++
int main() {
    std::cout << "In main\n";

    std::cout << "Printing known resource types and their constructors: \n";
    for(const auto& pair: ResourceDatabase::getInstance().mFactories) {
        std::cout << "\tfactory:" << pair.first << std::endl;
        for(const auto& methodPair: pair.second->mFactoryMethods) {
            std::cout << "\t\tmethod:" << methodPair.first << std::endl;
        }
    }
    std::cout << std::endl;

    using typeMethodParams = std::tuple<std::string, std::string, std::string>;

    // These tuples act as serialized resource descriptions; they could
    // be read from a JSON or XML file.
    std::vector<typeMethodParams> resourceDescriptions { 
        {"String", "FromInt", "1"},
        {"String", "FromString", "Two"},
        {"String", "FromInt", "3"},
        {"String", "FromInt", "4"},
    };

    std::cout << "Printing resource descriptions and created resources: \n";
    for(const auto& description: resourceDescriptions) {
        // It's the factory methods job to deserialize resource descriptions
        std::shared_ptr<StringResource> strResource { 
            std::static_pointer_cast<StringResource, IResource>(
                ResourceDatabase::getInstance().mFactories[std::get<0>(description)]
                    ->mFactoryMethods[std::get<1>(description)]
                    ->createResource(
                        std::get<2>(description)
                    )
            )
        };
        std::cout << "\tresource description: " << std::get<0>(description) << ", " << std::get<1>(description) << ", " << std::get<2>(description) << "\n";
        std::cout << "\tcreated string: " << strResource->mResource << std::endl;
    }

    return 0;
}
```

So we have `ResourceDatabase` -- a singleton acting as our store for resource constructors.
We start by querying the database about the resource types its aware of, as well as the
constructors that can make each type.

```c++
int main() {
    // ...

    std::cout << "Printing known resource types and their constructors: \n";
    for(const auto& pair: ResourceDatabase::getInstance().mFactories) {
        std::cout << "\tfactory:" << pair.first << std::endl;
        for(const auto& methodPair: pair.second->mFactoryMethods) {
            std::cout << "\t\tmethod:" << methodPair.first << std::endl;
        }
    }
    std::cout << std::endl;

    // ...
}

/*

OUTPUT
Printing known resource types and their constructors:
    factory:String
        method:FromInt
        method:FromString

*/
```


We have a list of resource descriptions, acting as the resource description store in this
example.

```c++
    using typeMethodParams = std::tuple<std::string, std::string, std::string>;

    // These tuples act as serialized resource descriptions; they could
    // be read from a JSON or XML file.
    std::vector<typeMethodParams> resourceDescriptions { 
        {"String", "FromInt", "1"},
        {"String", "FromString", "Two"},
        {"String", "FromInt", "3"},
        {"String", "FromInt", "4"},
    };
```

It's worth noting that the `String`-`FromString` constructor directly interprets the
resource parameter as a string, passing it to `std::string`'s constructor.
`String`-`FromInt` on the other hand uses the resource parameter as an index into the
constructor's private string array:

```c++
class StringResourceFromInt: public ResourceFactoryMethod<StringResource, StringResourceFromInt> {
public:

    // ...

    const std::vector<std::string> mStrings {"Haha", "This should", "be fun.", "(I think)", "Woohooo"};
};
```

The driver finally _uses_ those descriptions to actually construct the resources:

```c++
int main() {

    // ...

    // we fake ResourceDatabase's dispatch here, because that's not the part we're
    // interested in just at this moment.

    std::cout << "Printing resource descriptions and created resources: \n";
    for(const auto& description: resourceDescriptions) {
        // It's the factory methods job to deserialize resource descriptions
        std::shared_ptr<StringResource> strResource { 
            std::static_pointer_cast<StringResource, IResource>(
                ResourceDatabase::getInstance().mFactories[std::get<0>(description)]
                    ->mFactoryMethods[std::get<1>(description)]
                    ->createResource(
                        std::get<2>(description)
                    )
            )
        };
        std::cout << "\tresource description: " << std::get<0>(description) << ", " << std::get<1>(description) << ", " << std::get<2>(description) << "\n";
        std::cout << "\tcreated string: " << strResource->mResource << std::endl;
    }

    // ...

}

/*
OUTPUT
Printing resource descriptions and created resources: 
from FromInt
    resource description: String, FromInt, 1
    created string: This should
from FromString
    resource description: String, FromString, Two
    created string: Two
from FromInt
    resource description: String, FromInt, 3
    created string: (I think)
from FromInt
    resource description: String, FromInt, 4
    created string: Woohooo
*/
```

You'll also see that the [ToyMaker's resource database header](https://github.com/raynmetal/toymaker/blob/main/ToyMaker_Main/include/toymaker/engine/core/resource_database.hpp)
follows an extremely similar pattern, but fitted to the engine's requirements.

Feel free to modify [the example,](https://godbolt.org/z/d1T7o14zj) adding
more resources and constructors to it as you please!  I hope you'll find it a fairly
flexible implementation, and can be used with effect outside of game engines.

## References

- [Learn CPP](https://www.learncpp.com/) -- The site where I got my start with C++.

- [Walter E. Brown's "Modern Template Metaprogramming: A Compendium" from CppCon 2014](https://www.youtube.com/watch?v=Am2is2QCvxY)
-- Which taught me from first principles how template metaprogramming actually works.

