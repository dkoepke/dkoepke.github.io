---
layout: post-no-feature
title: "The Secret Lifetimes of C++ Objects"
description: "Understanding object lifetimes and avoiding the proliferation of the shared_ptr"
category: articles
tags: [c++, shared_ptr, lifetimes, objects, OOP, object-oriented, unique_ptr, smart pointers, pointers, memory management, memory leaks, cpp, c++11, c++14, c++17, modern c++, programming, software engineering]
---

Back in the Spring of 1793, the French Revolution was in full swing, and the French National Convention published a decree that, in its original French, went like this, &ldquo;Ils doivent envisager qu’une grande responsabilité est la suite inséparable d’un grand pouvoir.&rdquo; This admonition, translated and transformed, wove its way through political and religious history: echoing in the House of Commons by the early 19th century, called out from the pulpit 40 years later, intoned in 1906 by no less a figure than Churchill. Until, at last, it arrived to the American pop conscious in 1964 via the auspicious vehicle of the <cite>Spiderman</cite> comic books: &ldquo;With great power comes great responsibility.&rdquo;

This adage could be the motto for C++. Here you have a language that lets you do many things, to micro-optimize and to waste cycles for no reason, to under-think and to over-complicate, to write a lot of code to do very little. This strange mix of the positives and negatives, the unchecked power, and the difficulty of wielding it to good result, continues to birth new languages three decades on. The success of Ruby, Python, Java, and Go may all be attributed to where C++ (and C before it) failed. But what's often glossed over is that C++, too, has evolved and is evolving. The clean and idiomatic code you once wrote is now quite bad by modern standards: we have begun to take on the responsibility that follows inseparably from great power.

Consider the following classical C++ code:

```cpp
class Emulator
{
public:
    Emulator()
    : m_cpu(new CPU())
    , m_fpu(new FPU())
    , m_gpu(new GPU())
    , m_dma(new Memory())
    { }

    ~Emulator()
    {
        delete m_cpu;
        delete m_fpu;
        delete m_gpu;
    }

    void raiseInterrupt(IRQ irq)
    {
        m_cpu->raiseInterrupt(irq);
    }

    // ...

private:
    CPU* m_cpu;
    FPU* m_fpu;
    GPU* m_gpu;
    Memory* m_dma;  // for Direct Memory Access, obviously
};
```

At one point in time, this is uncontroversial and just obviously how you do things (save the memory leak; did you catch it?). Today, it immediately fails code review as soon as the word `new` shows up. The prohibition here is not really against that particular keyword but against the fact that we have procedurally implemented a contract that we can declaratively enforce:

```cpp
class Emulator
{
public:
    Emulator()
    : m_cpu(std::make_unique<CPU>())
    , m_fpu(std::make_unique<FPU>())
    , m_gpu(std::make_unique<GPU>())
    , m_dma(std::make_unique<Memory>())
    { }

    // No need for ~Emulator() any more

    void raiseInterrupt(IRQ irq)
    {
        m_cpu->raiseInterrupt(irq);  // this code is the same
    }

private:
    std::unique_ptr<CPU> m_cpu;
    std::unique_ptr<FPU> m_fpu;
    std::unique_ptr<GPU> m_gpu;
    std::unique_ptr<Memory> m_dma;
};
```

The contract here has to do with lifetimes, and it's one of the major responsibilities we have long shirked in C++. When are objects born? When do they die? What object (or objects) dictate their existence? In the world of manual memory management, we `new` them up when the code dictates they must exist, and we find some place to `delete` them that seems like it won't crash or leak memory, and we pass around references and pointers without considering the union of different lifetimes. In any medium-sized, classical C++ codebase, you can spend countless hours analyzing whether objects will all be alive when they're needed and fixing the inevitable crashes that occur when they aren't. The lifetimes of our C++ objects are secrets even from the people that wrote the code.

In modern C++, we write the contract more explicitly. In both versions, the emulator is the owner of the CPU, FPU, GPU, and Memory objects, and it dictates their lifetimes. But in our modern version, by making these instance variables of type `std::unique_ptr<T>`, we have declared within the code that the instances do not outlive the emulator (unless their ownership is transferred--more on that later). We still must choose the moment of creation and deal with nullability, but we no longer need to explicit `delete` the objects. They will go away when the unique pointers that own them go away.

### Sharing is not caring

Here's a nasty surprise: opting into smart pointers does not relieve you of your responsibility to understand your objects' lifetimes. Objects need to be passed to each other, and the question often becomes, when we're using a smart pointer, how do we pass them around? What do we do with this classical C++ code?

```cpp
class CPU
{
public:
    // ...

private:
    Memory* m_dma;
};
```

To get to the somewhat surprising answer, let's consider our options. The first option that we can readily eliminate is to make this also a `std::unique_ptr<T>`:

```cpp
class CPU
{
public:
    // ...

private:
    std::unique_ptr<Memory> m_dma;
};
```

The problem here is that we then need to transfer ownership of the memory from the emulator to the CPU:

```cpp
Emulator()
: m_cpu(std::make_unique<CPU>())
, m_fpu(std::make_unique<FPU>())
, m_gpu(std::make_unique<GPU>())
, m_dma(std::make_unique<Memory>())
{
    m_cpu->memory(std::move(m_dma));

    // But now this->m_dma is invalidated!
}
```

When we move the unqiue pointer, we lose our own reference. That is the whole point of the unique pointer: that there is exactly one owner of the object. Faced with this, you might reach for the other smart pointer that C++11 gives you, the `std::shared_ptr<T>`:

```cpp
class Emulator
{
public:
    Emulator()
    : m_cpu(std::make_shared<CPU>())
    , m_fpu(std::make_shared<FPU>())
    , m_gpu(std::make_shared<GPU>())
    , m_dma(std::make_shared<Memory>())
    {
        // The CPU and Emulator can share ownership of m_dma
        m_cpu->memory(m_dma);
    }

    // No need for ~Emulator() any more
    // ...

private:
    std::shared_ptr<CPU> m_cpu;
    std::shared_ptr<FPU> m_fpu;
    std::shared_ptr<GPU> m_gpu;
    std::shared_ptr<Memory> m_dma;
};
```

I often find myself wishing that shared pointers didn't exist. They were added to the language because objects can sometimes have complex lifetimes that are not statically knowable. They are a niche tool, but now they litter many modern C++ codebases because, by using them, it seems like you can ignore your responsibility to understand your objects' lifetimes. The shared pointer has it handled for you. The objects will all be cleaned up when their references go out of scope. Except that, aside from the semantics here being completely wrong, and you possibly incurring atomic reference count operations for no reason at all, it's a dangerous lie:

```cpp
class Memory
{
public:
    Memory(const std::shared_ptr<CPU>& cpu)
    : m_cpu(cpu)
    {
        // ...
    }

    // ...

private:
    std::shared_ptr<CPU> m_cpu;
    // ...
};

class Emulator
{
public:
    Emulator()
    : m_cpu(std::make_shared<CPU>())
    , m_fpu(std::make_shared<FPU>())
    , m_gpu(std::make_shared<GPU>())
    , m_dma(std::make_shared<Memory>(m_cpu))
    {
        m_cpu->memory(m_dma);
    }

    // ...
};
```

You have now very concisely implemented a memory leak. The CPU has two owners: the emulator and the memory. The memory has two owners: the emulator and the CPU. When the emulator is destructed, the reference counts are decremented. Now there is one owner of the CPU: the memory. And there is one owner of the memory: the CPU. Neither are reachable from your code at this point, but both sit there in memory. The destructors won't be called. You've created a reference cycle.

The advice I give around `std::shared_ptr<T>` is that it's the surest sign you need to think *more* about what you're doing. You have more responsibility with shared pointers to understand what's going on, not less. In many cases, the existence of shared ownership is a good indication that your design can be improved. The moment you reach for a `std::weak_ptr<T>` to avoid cycles is the moment you should really stop to contemplate what you're doing with your life and the lives of your objects. It's not that these tools are never needed; it's that they're far more rarely the right choice than common practice makes it seem.

### Pointing the right way

What then do we do to pass around our references? All we have arrived at is that our code can perhaps work with shared pointers if we follow the edict to better understand our lifetimes and avoid circular references. But what's odd here is that we do understand our lifetime. We were able to succinctly state it earlier:

<blockquote>the emulator is the owner of the CPU, FPU, GPU, and Memory objects, and it dictates their lifetimes</blockquote>

The `std::unique_ptr<T>` remains the right choice to express that contract. The solution to our impasse is simply to rely on the fact that the CPU isn't an owner of the Memory: it has access to it, but it has nothing to say about the lifetime of that object. The only part of the contract that is not expressed within the implementation of the CPU class--that the Memory exists as long as the CPU exists--is enforced by the emulator itself because it is an owner. Therefore, our correct solution for our existing design is just this:

```cpp
class CPU
{
public:
    // ...

    void memory(Memory* dma) { m_dma = dma; }

private:
    Memory* m_dma;
};

class Emulator
{
public:
    : m_cpu(std::make_unique<CPU>())
    , m_fpu(std::make_unique<FPU>())
    , m_gpu(std::make_unique<GPU>())
    , m_dma(std::make_unique<Memory>())
    {
        m_cpu->memory(m_dma.get());
    }

    // ...

private:
    std::unique_ptr<CPU> m_cpu;
    std::unique_ptr<FPU> m_fpu;
    std::unique_ptr<GPU> m_gpu;
    std::unique_ptr<Memory> m_dma;
};
```

That is, the classical C++ remains the right way to do what we want with the `CPU` class. The `CPU` has its lifetime externally managed, and it does not manage the lifetime of the `Memory` it is given.

We might feel queasy about handing out raw pointers like this, but it is actually more correct to do so than the abuse of smart pointers we'd earlier contemplated. We simply need to adjust our expectations about what raw pointers mean. In classical C++, a raw pointer communicated that you may or may not have to worry about ownership and lifetime. In modern C++, a raw pointer communicates that wherever you got the reference from has specified (or, transitively, inherited) a lifetime policy. The raw pointer concisely communicates the idea that the guarantees are written elsewhere: they are the caller's responsibility. Your class needs to be aware of how that policy's enforcement might impact your implementation, but it does not need to do any enforcement on its own. Somewhat sensibly, the raw pointer serves as a pointer to the human reader: look elsewhere.

### Writing better contracts

We have cause to still be unhappy with this code. It's not that we have handed out a raw pointer, it's not even much about lifetimes because we have our contract written. What we have done is handed the `CPU` access to the `Memory` at any point in time. It can do whatever it wants, including modifying its internal state.

The contract we have here is wide open: the lifetime is guaranteed for you, now do whatever you want. In the presence of threads, we have the possibility of synchronization issues. Even without concurrency or parallelism to concern ourselves over, we are left with the inevitability of more and more methods using more and more functionality of the instances they reference. And, yes, we have allowed for programmer error to cause problems around the misuse of the object's lifetime. And for what? Just to do a bit less typing, we have opened Pandora's box.

So let's fix this. In one of our iterations, we played with the idea of giving the `Memory` a reference to the `CPU`. This might be needed to deliver an interrupt:

```cpp
void Memory::page_fault()
{
    // ...
    m_cpu->raiseInterrupt(IRQ::PAGE_FAULT);
}

void Emulator::run()
{
    while (1)
    {
        // ...
        if (page_not_found)
        {
            m_dma->page_fault();
        }
    }
}
```

When we dismissed `std::shared_ptr<CPU>` as the type for `Memory`'s `m_cpu`, it was both because this introduced a circular reference and because the semantics--the idea that `Memory` owns the `CPU`--was wrong. We can fix both (and also obviate our original motiviation for considering shared pointers) by observing that we can simply pass the `CPU` to the methods that need it. This makes tight coupling harder on us and avoids dire programmer errors.

Moreover, since these methods assume that the CPU exists (is not null, is valid), we can also use a non-nullable reference instead of a pointer:

```cpp
void Memory::page_fault(CPU& cpu)
{
    cpu.raiseInterrupt(IRQ::PAGE_FAULT);
}

void Emulator::run()
{
    while (1)
    {
        // ...
        if (page_not_found)
        {
            m_dma->page_fault(*m_cpu);
        }
    }
}
```

The emulator is calling `page_fault` and passing in its `CPU` that it has created in its constructor and that, by the contract enforced with `std::unique_ptr<T>` (in the absence of `reset` or `std::move`) will live until the emulator is destructed. Therefore, if `page_fault` does not start a thread that holds the `CPU` reference, the implicit contract around a synchronous function call resolves our worries about us screwing up the lifetimes: `CPU` is alive when we pass it to `page_fault`, therefore it's alive during `page_fault`'s execution.

The exercise is to understand lifetimes and ownership, so that we may write them out clearly and coherently. In our fictional emulator, ownership is entirely dictated by the top-level class, and all lifetimes are neatly dictated from there. Since C++ lacks Rust's tools for static verification of lifetimes, we need instead to expose the secret lifetimes of our objects through the simple concepts of unique ownership, "look elsewhere" raw pointers, and, if we really can't properly scope the lifetime up-front, shared ownership. When you are writing C++, this is your responsibility to the people that will use your software, and to your coworkers and your future self that will inherit your codebase. Use your powers wisely.
