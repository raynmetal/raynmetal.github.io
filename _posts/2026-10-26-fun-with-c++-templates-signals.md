---
layout: post
title:  "Fun With C++ Templates -- Signals"
date:   2026-10-26 17:30:00 +0530
categories: blog technical
tags: [c++, metaprogramming, templates, observer pattern, signals, events]
excerpt_separator: <!--end excerpt-->
---

Signals are a popular pattern that help you decouple systems that publish
notifications about some event taking place, from systems that need to be
notified _when_ that event takes place.

<!--end excerpt-->

## Introduction

### Scenario #1: ECommerce Transaction Processor

Say you're writing a server application for a storefront.  Your customer has just
made a purchase online, money has left their account, and you've received confirmation
from your payment provider that it's entered yours.

```c++
void TransactionProcessor::process(Transaction& transaction) {
    transaction.mProvider.awaitResponse();

    if(!transaction.mProvider.paymentConfirmed()) {
        transaction.mStatus = TransactionStatus::FAILED;
    } else {
        transaction.mStatus = TransactionStatus::COMPLETED;
    }

    mDatabase.updateTransaction(transaction);
}
```

So the transaction has been completed, the payment has been recorded.  But shouldn't
the customer be notified in some manner?  We have their email, so I guess we could
send them an email of the confirmation:

```c++
// ...
    transaction.mProvider.awaitResponse();

    if(!transaction.mProvider.paymentConfirmed()) {
        transaction.mStatus = TransactionStatus::FAILED;
        mEmailer.notifyTransactionFailure(transaction);
    } else {
        transaction.mStatus = TransactionStatus::COMPLETED;
        mEmailer.notifyTransactionSuccess(transaction);
    }

    mDatabase.updateTransaction(transaction);
// ...
```

Great!  But what if we wanted to send them an SMS as well?  Emails are _generally_
reliable, but redundancy, especially when it comes to matters of money, is good.

```c++
// ...
    if(!transaction.mProvider.paymentConfirmed()) {
        transaction.mStatus = TransactionStatus::FAILED;
        mEmailer.notifyTransactionFailure(transaction);
        mSMSer.notifyTransactionFailure(transaction);
    } else {
        transaction.mStatus = TransactionStatus::COMPLETED;
        mEmailer.sendTransactionSuccesss(transaction);
        mSMSer.notifyTransactionFailure(transaction);
    }
// ...
```

Oh right, these days practically everyone has a smartphone, and the customer made
the purchase through our app.  We should send them a push notification too...

You can see how this is getting rather cumbersome.  Our `TransactionProcessor`
shouldn't have to do much more than receive payment confirmation and update the
transaction's status in our records, but it's been made responsible for both doing
that _and_ notifying the customer about it.

There are now _four whole lines_ that have nothing to do with actually "processing"
the transaction, and there will be more for each new system that needs to know when
a transaction takes place.

### Scenario #2: Bullet Damage in a Shooter Game

You're making a shooter game.  A bullet has just collided with an enemy, and so the
enemy needs to take damage from it.

```c++
void BulletScript::onCollision(Collider& other) {
    // if you don't have health, you don't take damage
    if(!other.getObject().hasScript<HealthScript>()) {
        return;
    }

    // the health script handles health values
    auto& health { other.getObject().getScript<HealthScript>() };
    health.applyDamage(mDamage);

    // bullet disappears from the world on collision or timeout
    deferDestroySelf();
}
```

Well, we're not done yet.  We want a hit sound to play.  Maybe we'd like some stats
to be updated.  Maybe the player's aiming cross should show a marker to let them know
they've hit something?

```c++
// ...
    health.applyDamage(mDamage);

    SoundSystem.play(SoundSystem::BULLET_HIT);
    StatsManager.updateBulletHit(*this, other.getObject());
    PlayerUI.showHitMarker();

// ...
```

Our little bullet script is doing quite a lot, now, isn't it?

### What are signals?

Whatever I'm calling a signal, is simply an implementation of the [observer pattern](https://en.wikipedia.org/wiki/Observer_pattern).
It allows us to decouple systems that _publish_ events, from systems that must perform
an action upon _receiving_ those events.

This is in fact such a common requirement, that [some frameworks,](https://docs.djangoproject.com/en/6.1/topics/signals/)
and even [some languages,](https://learn.microsoft.com/en-us/dotnet/standard/events/)
support some or the other version of it.

> **Caution**
>
> While signals help you decouple mostly unrelated parts of your program, you
> should use them carefully.  Litter your codebase with too many of them, and
> your programs flow of execution becomes hard to follow.
>
> As a rule of thumb, use _signals_ in functions where the intended _observer_ has
> _no direct relationship_ with the task being performed.  If both systems do
> need to collaborate closely in order to perform the function's primary task,
> then they should both be invoked directly.
>
> Also ensure that the observer's code remains extremely short, so that it is
> processed nearly instantaneously and doesn't slow down the publisher's primary
> task.  Signals are messages sent and processed synchronously.

## Signal, Observer, Tracker

### Implementation

There's an expanded version of my signals implementation, along with a driver
driver program, which you can tinker with [on godbolt.](https://godbolt.org/z/4czv4edao)
If that link expires, you can check it out on my [git repo.](https://github.com/raynmetal/cpp-experiment-snippets/blob/main/ObserverPatternImplementation.cpp)

The signal publisher and signal observer _each get a single base class._  This
is to allow pointers to publishers and observers to be passed around without
knowledge of their particular type.

The actual functionality of the signal or the observer is contained in the templated
Signal and SignalObserver types.

#### Signal Publisher

```c++
// Class providing base class pointer for signal publishers.  This
// is in case the signal's life should be tied to another object.
class ISignal {
public:
    // virtual method, overridden with checks in templated signal class
    virtual void registerObserver(std::weak_ptr<ISignalObserver> observer)=0;
};

// The actual, templated Signal publisher implementation.
template <typename ...TArgs>
class Signal: public ISignal {
public:
    void emit (TArgs... args);
    void registerObserver(std::weak_ptr<ISignalObserver> observer) override;

private:
    Signal() = default;

    std::set<
        std::weak_ptr<SignalObserver<TArgs...>>,
        std::owner_less<std::weak_ptr<SignalObserver<TArgs...>>>
    > mObservers {};
};
```

**Why do we make the Signal a templated type?**  Well, that's because the notion of
an "event" is rather flexible.  When something happens, all of the information about
the happening should be pushed into a packet, and sent across to all parties that
are interested in it.  The particular information, and therefore the shape of the 
signal, would depend on the context of the signal.

My interpretation of this is that an "event" is just a list of parameters, passed
to the observer's handler function when it is called.  Each parameter can be of
any type.

Only observers whose function signature matches that of the publisher, is permitted
to subscribe to that signal.  To represent this generically, I make use of C++'s
[function and template parameter packs](https://en.cppreference.com/cpp/language/pack)

**How do we know who's observing this signal?**  The answer is, we _store_ it!  Through
the `registerObserver()` method.

```c++
template <typename ...TArgs>
inline void Signal<TArgs...>::registerObserver(std::weak_ptr<ISignalObserver> observer) {
    // we actually have 2 checks here; one that ensures the pointer hasn't expired,...
    assert(!observer.expired() && "Cannot register a null pointer as an observer");

    // ... the other to ensure the observer's signature matches ours.
    mObservers.insert(std::static_pointer_cast<SignalObserver<TArgs...>>(observer.lock()));
}
```

**How do we send a signal, then?**

We just call `mSignal.emit(/* your args here */)`.  `emit()` simply iterates through each
subscribed observer, discarding those that have expired, and invoking the observer handler
on the ones that haven't.

```c++
template <typename ...TArgs>
void Signal_<TArgs...>::emit (TArgs ... args) {
    //observers that will be removed from the list after this signal has been emitted
    std::vector<std::weak_ptr<SignalObserver_<TArgs...>>> expiredObservers {};

    for(auto observer: mObservers) {
        // lock means that this observer is still active
        if(std::shared_ptr<SignalObserver_<TArgs...>> activeObserver = observer.lock()) {
            (*activeObserver)(args...);

        // go to the purge list
        } else {
            expiredObservers.push_back(observer);
        }
    }

    // remove dead observers
    for(auto expiredObserver: expiredObservers) {
        mObservers.erase(expiredObserver);
    }
}
```

#### Signal observer

```c++
class ISignalObserver {};

template <typename ...TArgs>
class SignalObserver: public ISignalObserver {
public:
    void operator() (TArgs... args);
private:
    // note that it's discouraged to use `std::function`, but I wrote
    // this when I didn't know any better.
    SignalObserver(std::function<void(TArgs...)> callback);
    std::function<void(TArgs...)> mStoredFunction {};
};
```

There's not a whole lot going on here.  The observer mainly just exists so that the
signal publisher knows whom to message when an event happens, making this a kind of
mailing address.

The most interesting thing here is that we store a function, `mStoredFunction`, that
we invoked when we receive a signal.

```c++
template <typename ...TArgs>
inline void SignalObserver_<TArgs...>::operator() (TArgs ... args) { 
    mStoredFunction(args...);
}
```

#### Extra: Signal... Tracker?

If you visit the [godbolt implementation](https://godbolt.org/z/4czv4edao) or the one
hosted on [GitHub,](https://github.com/raynmetal/cpp-experiment-snippets/blob/main/ObserverPatternImplementation.cpp)
you'll notice that there's quite a bit more going on there than I've written about here.

The reason for that, and the reason we even need the `ISignal` and `ISignalObserver` base
classes, is so that an intermediate entity, the `SignalTracker`, can manage the advertisement
of publishers and their connections with observers, without knowledge of the publisher or
observer's particular type.

In [ToyMaker,](https://github.com/raynmetal/toymaker) this was desirable because it allowed
me to specify which publisher, as a string, was connected with which observer, also as a
string.  This allowed me to describe those connections as part of my engine's scene files.
The intermediate objects in this case are the scene objects whose paths are `from` and
`to`, while the publisher's and observer's names are `signal` and `observer` respectively.

```jsonc
{
    // ...

    "connections": [

        // ...

        {
            "from": "/viewport_UI/ui_container/ui_panel/next_turn/@UIButton",
            "signal": "ButtonReleased",

            "to": "/@UrUIView",
            "observer": "ButtonClickedObserved"
        },

        // ...
    ]
}
```

The main changes from the implementation described here are:

1. Rename `Signal<typename ...TArgs>` --> `Signal_<typename ...TArgs>`

2. Rename `SignalObserver<typename ...TArgs>` --> `SignalObserver_<typename ...TArgs>`

3. `Signal_` and `SignalObserver_` are no longer intended to be created directly by
a user.  Their creation and lifetimes are instead managed by `SignalTracker`.

4. We create new non-copyable, non-movable `Signal` and `SignalObserver` classes, which
take in a reference to a `SignalTracker` in their constructor, as well as their own string
names.

## In Practice:

### Scenario #1

We (as the `TransactionProcessor`) no longer need to worry about _everyone_ who needs to
know a transaction has taken place; we can leave the handling of that to our special
`Signal` class.

```c++
void TransactionProcessor::process(Transaction& transaction) {
    transaction.mProvider.awaitResponse();

    if(!transaction.mProvider.paymentConfirmed()) {
        transaction.mStatus = TransactionStatus::FAILED;
    } else {
        transaction.mStatus = TransactionStatus::COMPLETED;
    }

    mDatabase.updateTransaction(transaction);

    mSignalTransactionEnd.emit(transaction);
}
```

Whoever cares to hear about the end of a transaction can just subscribe to
`TransactionProcessor::mSignalTransactionEnd`, like so:

```c++


void setup() {
    transactionProcessor.mSignalTransactionEnd.registerObserver(
        emailSystem.mObserverTransaction
    );
    transactionProcessor.mSignalTransactionEnd.registerObserver(
        smsSystem.mObserverTransaction
    );

    // .. and as many other observers as we like
}
```

### Scenario #2

Now our bullet can just say that it hit something, and leave everything else
to the its observer's handlers!

```c++
void BulletScript::onCollision(Collider& other) {
    // if you don't have health, you don't take damage
    if(!other.getObject().hasScript<HealthScript>()) {
        return;
    }

    mSignalHit.emit(getObject(), other.getObject());

    // the health script handles health values
    auto& health { other.getObject().getScript<HealthScript>() };
    health.applyDamage(mDamage);


    // bullet disappears from the world on collision or timeout
    deferDestroySelf();
}
```

Who  is listening for this is none of the bullet damage method's business.

```c++
// when and how this function runs is beyond the scope for this article, but it
// wouldn't be hard to imagine that this is done as part of the bullet's instantiation
void bulletSpawned(Bullet& bullet) {
    bullet.mSignalHit.registerObserver(StatsManager.mObserverBulletHit);
    bullet.mSignalHit.registerObserver(PlayerUI.mObserverBulletHit);
    bullet.mSignalHit.registerObserver(SoundSystem.mObserverBulletHit);
}
```

## References

- [C++ Template Parameter Packs and Functions](https://cppreference.patrickfasano.com/en/cpp/language/parameter_pack.html)
-- A very thorough, if a bit dense, description of the behaviour of C++'s template parameter packs.

- [C#'s Delegates and Events Feature](https://learn.microsoft.com/en-us/dotnet/standard/events/)
-- One of the first places I encountered this concept.

