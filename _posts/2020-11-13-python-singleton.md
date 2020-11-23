---
title: "Python singleton - the one and only"
layout: single
published: true
---

What is so hard about creating a singleton in Python? That's the question of the day after having done some quick browsing in search of inspiration for the most pythonic way to implement the singleton pattern. The problem? A surprising amount of articles and tutorials written by professional software developers got it wrong (and some even proposed implementations that lead to even more instances being created than would've been without their "singleton" pattern). We can't have that now, can we?


# How objects are created in Python 3
The main issue seems to be the authors don't have firm grasp of how instance creation works in Python. Specifically, they failed to realize it's not `__init__` that's creating the actual instance. Just look at its simplest signature, `def __init__(self)`. In its simplest form, it takes an instance of the class, so something must have created the instance before `__init__` is called. Looking at the [documentation](https://docs.python.org/3/reference/datamodel.html#object.__new__), it becomes clear that it's the `__new__` method that's called to create a new instance of a class.

The documentation goes on to explain that:

> If `__new__()` is invoked during object construction and it returns an instance or subclass of cls, then the new instance’s `__init__()` method will be invoked like `__init__(self[, ...])`, where self is the new instance and the remaining arguments are the same as were passed to the object constructor.

> If `__new__()` does not return an instance of cls, then the new instance’s `__init__()` method will not be invoked.


From this, it's clear that we need to do more than just modify the `__init__` function to prevent an instance from being created. In particular, we need to override the default `__new__` function.


# Why singletons?
Before designing a singleton, this post deserves a few words on why you'd want a singleton and, consequently, the reasons behind the design decisions below.

So, the three main reasons you'd want only one instance are that 1) object creation is slow and/or expensive, 2) resources are limited (memory, connections etc.), 3) the same instance needs to be used everywhere. There may be more reasons, but all situations I can think of right now typically falls under one (or all) of these categories in one way or the other. 

## The shortcomings of existing solutions/articles
The suggested patterns I found when browsing the web all met the requirements of the third case. However, they generally didn't guarantee correctness in the second case and typically failed completely to avoid unnecessary object creation.

Below, I propose some basic designs that meet the requirements of all these three cases.


# Designing a basic singleton
Either you can acquire the singleton explicitly or implicitly. Which one you opt for is up to you, but if you're familiar with the zen of Python, you'll know that _explicit is better than implicit_. Thus, although the explicit way may be considered more Pythonic, I'll give examples of each.


## Implicit singleton
There really isn't much to it. You need to override the default `__new__` magic function such that the instance is created the first time you call `Singleton()`, but for each subsequent invocation the existing instance is returned.

```python
class Singleton:
    __instance = None

    def __new__(cls, *args, **kwargs):
        if cls.__instance is None:
            cls.__instance = super().__new__(cls, *args, **kwargs)
            # Initialization code goes here
        return cls.__instance
```

Do note that the initialization code is also put in the `__new__` method so that it's only run once and not every time you retrieve the instance. If you put it in `__init__`, the instance will be "re-initialized" each time you run `Singleton()`. 


## Explicit singleton
There are a few variations of the explicit version. For instance, some implementations that you'll come across will require you to create the `Singleton` object first and then call `get_instance()` on it. Other implementations will just make it a `@classmethod`.

### The wrong way
An actual example of lazy instantiation taken from the first page returned for by Google for a quite generic search looked something like this:

```python
class Singleton:
    __instance = None

    @classmethod
    def get_instance(cls):
        if cls.__instance is None:
            cls.__instance = Singleton()
        return cls.__instance
```

The post then goes on to explain how to use this singleton. I quote:

```python
# Class initialized, but object not created
s = Singleton()
s1 = Singleton()
```

So, what's wrong with this? After all, it seems quite similar to the example of the implicit singleton above.

Well, not really. Let's see for ourselves by adding a print-outs in its `__new__` method without changing the object creation functionality.

`singleton.py`
```python
class Singleton:
    __instance = None

    def __new__(cls, *args, **kwargs):
        print("New instance created")
        return super().__new__(cls, *args, **kwargs)

    @classmethod
    def get_instance(cls):
        if cls.__instance is None:
            cls.__instance = Singleton()
        return cls.__instance
```

```python
>>> from singleton import Singleton
>>> s = Singleton()
New instance created
>>> print(s._Singleton__instance)
None
>>> s1 = Singleton()
New instance created
>>> print(s1._Singleton__instance)
None
>>> s1.get_instance()
New instance created
<test.Singleton object at 0x1034f8250>
```

As it turns out, this is effectively the opposite of lazy instantiation - we first created two instances of `Singleton` (i.e. `s` and `s1`), but none of them was assigned to `__instance`. When we finally called `get_instance()`, a third instance was created. That's not what we were aiming for.

So, what if we didn't instantiate `Singleton` before we called `get_instance()` (which is a class method), would that achieve what we want?

```python
>>> from singleton import Singleton
>>> s = Singleton.get_instance()
New instance created
>>> s1 = Singleton.get_instance()
>>> s
<singleton.Singleton object at 0x1015a9250>
>>> s1
<singleton.Singleton object at 0x1015a9250>
```

Used this way, it actually works as intended. The class attribute `__instance` gets properly assigned to an instance of `Singleton`. However, I have two issues with this. Firstly, there's nothing preventing a user of your class from instantiating `Singleton` and then calling `get_instance()` on it. Secondly, it's quite awkward to call `Singleton()` from within `Singleton`. It interferes with how you can override `__new__` and `__init__`. For instance, if you want to prevent a programmer from calling `Singleton()` directly, the above implementation will limit your options since the `Singleton()` within the class itself will invoke `__new__` and `__init__`.


### The correct way
So, what's a better way?

`singleton.py`
```python
class Singleton:
    __instance = None

    def __init__(self):
        raise RuntimeError("Class can't be instantiated directly. 'Call get_instance()'")

    @classmethod
    def get_instance(cls):
        if cls.__instance is None:
            cls.__instance = cls.__new__(cls)
            # Initialization code goes here or in __new__.
        return cls.__instance
```

This design makes it impossible to instantiate the class directly. Do note how the `Singleton()` in `get_instance(cls)` has been changed to `super().__new__(cls)`, or we'd have hit the runtime error in `__init__()`. Also note the use of `cls.__init__` instead of `super().__init__`. The difference is insignificant in this case, but should you have overridden the default `__new__` in the class, you'd want to call the correct `__new__`.

Actually, it's not completely impossible to create an instance of `Singleton`. If you really wanted to create an instance, you could do:

```python
>>> from singleton import Singleton
>>> instance = Singleton.__new__(Singleton)
```

I'm sure you don't have to bother with this case, but let's fix it just for the fun of it.


### An even better way
In the above version the, if you try to create an instance of `Singleton` directly, you'll get a RuntimeError. However, this error occurs first _after_ the instance has been created (and you could get around it if you really wanted to). Thus, optimally, the error should be raised before we even try to create the object.

```python
class Singleton:
    __instance = None

    def __new__(cls, *args, **kwargs):
        raise RuntimeError("Class can't be instantiated directly. 'Call get_instance()'")

    @classmethod
    def get_instance(cls):
        if cls.__instance is None:
            cls.__instance = super().__new__(cls)
            # Initialization code goes here 
        return cls.__instance
```

Now, if you try to create a `Singleton` the wrong way, the error will be raised before the construction even begins.

What about creating an instance of Singleton directly? Well, if you try the workaround from above:

```python
>>> Singleton.__new__(Singleton)
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "~/singleton.py", line 5, in __new__
    raise RuntimeError(
RuntimeError: Class can't be instantiated directly. 'Call get_instance()'
```

That's all good, but you didn't think it'd be that easy, did you? If you really wanted an instance of `Singleton`, you can still use:

```python
>>> object.__new__(Singleton)
<singleton.Singleton object at 0x101687520>
```

Of course, any initialization code in `get_instance()` would be skipped, but a `Singleton` object has been created nevertheless.


### The best way?
Thus far, I haven't even mentioned metaclasses. While a class' `__new__` and `__init__` methods are concerned with how instances are created, metaclasses are concerned with how the classes themselves are created.

For instance, [this](https://stackoverflow.com/a/6966942) answer on stackoverflow lists some really good points why you'd want to separate a singleton into a (meta)class that takes care of the instantiation and a class concerned with the "actual" functionality. From a maintainability point of view, I think the _single responsibility principle_ alone is reason enough to prefer a metaclass.

Unless we define a class with a metaclass other than `type`, the class object (not to be confused with an instance of the class) will be of type `type`. As a simple example:

```python
>>> class A:
...     pass
...
>>> type(A)
<class 'type'>
```

What really happens when you use the round-bracket syntax to instantiate a class is that the Python interpreter interprets it as calling the class, which in this case is `Singleton`. At this point, another magic mehtod comes into play - the `__call__` method. When _calling_ a class of type `type`, the call resolves to `type.__call__()`. It's this function that calls `__new__` and, if appropriate, `__init__`.

This is the actual source of `type_call` of the CPython language:

`Objects/typeobject.c`
```c
static PyObject *
type_call(PyTypeObject *type, PyObject *args, PyObject *kwds)
{
    PyObject *obj;
    PyThreadState *tstate = _PyThreadState_GET();

#ifdef Py_DEBUG
    /* type_call() must not be called with an exception set,
       because it can clear it (directly or indirectly) and so the
       caller loses its exception */
    assert(!_PyErr_Occurred(tstate));
#endif

    /* Special case: type(x) should return Py_TYPE(x) */
    /* We only want type itself to accept the one-argument form (#27157) */
    if (type == &PyType_Type) {
        assert(args != NULL && PyTuple_Check(args));
        assert(kwds == NULL || PyDict_Check(kwds));
        Py_ssize_t nargs = PyTuple_GET_SIZE(args);

        if (nargs == 1 && (kwds == NULL || !PyDict_GET_SIZE(kwds))) {
            obj = (PyObject *) Py_TYPE(PyTuple_GET_ITEM(args, 0));
            Py_INCREF(obj);
            return obj;
        }

        /* SF bug 475327 -- if that didn't trigger, we need 3
           arguments. But PyArg_ParseTuple in type_new may give
           a msg saying type() needs exactly 3. */
        if (nargs != 3) {
            PyErr_SetString(PyExc_TypeError,
                            "type() takes 1 or 3 arguments");
            return NULL;
        }
    }

    if (type->tp_new == NULL) {
        _PyErr_Format(tstate, PyExc_TypeError,
                      "cannot create '%.100s' instances",
                      type->tp_name);
        return NULL;
    }

    obj = type->tp_new(type, args, kwds);
    obj = _Py_CheckFunctionResult(tstate, (PyObject*)type, obj, NULL);
    if (obj == NULL)
        return NULL;

    /* If the returned object is not an instance of type,
       it won't be initialized. */
    if (!PyType_IsSubtype(Py_TYPE(obj), type))
        return obj;

    type = Py_TYPE(obj);
    if (type->tp_init != NULL) {
        int res = type->tp_init(obj, args, kwds);
        if (res < 0) {
            assert(_PyErr_Occurred(tstate));
            Py_DECREF(obj);
            obj = NULL;
        }
        else {
            assert(!_PyErr_Occurred(tstate));
        }
    }
    return obj;
}
```

Skipping the initial special case and checks, we see the lines

```c
obj = type->tp_new(type, args, kwds);
```

and

```c
/* If the returned object is not an instance of type,
   it won't be initialized. */
if (!PyType_IsSubtype(Py_TYPE(obj), type))
    return obj;
```

Recalling the beginning of this post, this is the implementation of the part of the specification that says that if the object created in `tp_new` is an instance or subclass of the class, the instance will be initialized by calling the corresponding `__init__` method on it.

Now, conceptually, what we want to do is modify the sequence in `__call__` so that the object created the first time is reused for all subsequent calls. Luckily, though, we can't mess with attributes of `type` like that. The way around it is to create a new metaclass and override the default `__call__` method.

```python
class SingletonMetaclass(type):
    __instances = {}
    def __call__(cls, *args, **kwargs):
        if cls not in cls.__instances:
            cls.__instances[cls] = super().__call__(*args, **kwargs)
        return cls.__instances[cls]
```



If you want to get some more details about this, check out this excellent [post](https://eli.thegreenplace.net/2012/04/16/python-object-creation-sequence) or [this](https://realpython.com/python-metaclasses/#defining-a-class-dynamically) higher-level explanation of metaclasses and class instantiation sequence.



# Conclusion
If there's one thing you should remember from all this, it's that in Python 3 objects are created in the `__new__` method. Thus, if you want to avoid creating unnecessary objects or need to manage which object is given/returned to a consumer of your class, this is typically where you should start looking.


