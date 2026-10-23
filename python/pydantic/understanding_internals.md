# Pydantic Basics: Models, Annotations and Defaults

> Applies to Pydantic v2.

## How Pydantic works under the hood

Pydantic v2 is built on top of `pydantic-core`, a validation engine written in Rust.
When a Pydantic model class is defined, Pydantic reads the class's type annotations
and builds a **core schema** from them. This schema is passed to `pydantic-core`,
which turns it into a fast validator and serializer.

## Type annotations

Type annotations are stored in the class's `__annotations__` dict and act as metadata.
Python itself does not enforce types at runtime; tools like `mypy` use annotations
for **static** type checking. Unlike `mypy`, Pydantic reads the annotations
**at runtime** (when the class is created) to build its validators.

## Required vs. not-required fields

If a subclass of `BaseModel` defines a field `name: str`, then `str` determines how
`name` is validated. Because no default is given, the field is **required**.
If a default is given, the field is **not required**: when the field is omitted
at instance creation, the default value is used.


## Instances don't share field defaults

In a plain Python class, an attribute with a default in the class body is a
**class attribute**, shared by all instances. Assigning to it through an instance
(`obj.x = ...`) creates an instance attribute that shadows the class attribute,
whatever the type. The bug appears with **mutable** defaults: *mutating* the value
in place changes the single shared object, so every instance sees the change.

Pydantic avoids this. If a default is not hashable (e.g. a list or dict), Pydantic
deep-copies it for each new instance, so every instance gets its own value.
Hashable defaults like `str` or `int` are not copied, but since they cannot be
mutated, sharing them is harmless.


## `default_factory`

`default_factory` takes a **callable** (function, lambda, or class) that is called
to produce a fresh default each time an instance is created, instead of using one
value fixed at class-definition time. This is useful for values that must be
computed per instance, such as a creation timestamp:

```python
from datetime import datetime
from pydantic import BaseModel, Field

class Event(BaseModel):
  
    created_at: datetime = Field(default_factory=datetime.now)

    # Wrong: datetime.now() runs once at class definition,
    # so every instance would get the same timestamp
    # created_at: datetime = datetime.now()
```

Things to keep in mind:

- **Performance:** the factory is a Python call made on every instance creation,
  outside the Rust core. Usually negligible, but expensive factories can add up
  when creating very large numbers of instances.
- **Validation:** like regular defaults, the factory's output is **not validated**
  unless `validate_default=True` is set.
- **Using other fields:** the factory can take one argument, `data`, a dict of the
  fields already validated. Only fields defined **before** it are available,
  because fields are validated in definition order.

