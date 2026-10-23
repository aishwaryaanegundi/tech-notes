pydantic comprises of `pydantic-core` which is written in `Rust`
When a pydantic model class is defined,  pydantic uses the type annotations to create a `core-schema` which is passed on to `pydantic-core`

**Type Annotations**: Stored as a dict `__annotations__` acts as a metadata. `python` itself does not care about the types of the variables at the runtime. type annotations are used by tools like `mypy` to statically check the types. Unlike `mypy`, `pydantic` reads the type annotations at the runtime to build the validators.

Suppose a subclass of `BaseModel` defines a field `name:str` then `str` becomes a validator and since no default is mentioned the field is required. If a default is mentioned then the field is optional and is pydantic create a copy of the field for every instance of the class.

#### Unlike plain python class instances, pydantic model class instances do not share class attributes
Notice that pydantic model class behaves differently compared to the plain python class. In plain python class, if a field/attribute is defined with default, it becomes a class attribute that is shared across all instances. For immutable types like str, float a re assignment creates a instance attribute which overrides a class attribute. But for muatable types like lists, reassigning or mutating is shared across instances which can be buggy. But every instance of pydantic class gets its own instance of attributes with default irrespective of whether it is mutable or not. Pydantic model class attribute defaults only determine whether the field is required when creating instances or optional. If the field is not passed while creating an instance, then the attribute is added to the instance with default value.    

#### Need for default_factory
`default_factory` allows to pass a method as a default for a field. The method is called to compute the default value at the time of instance creation as opposed to a normal default value which is created once at class definition time and is reused by copying for every instance creation. This is useful for cases like time tracking where field `time_stamp: time.now()` should record time at the creation of instance. The `default_factory` method is is executed in python on every instance creation which can be performance bottle neck if using for large scale creation of instances. The output of `default_factory` is not validated if not `validat_default=True` is set. `default_factory` also accepts one field  from its class which is defined before it, hence, validated at the time of consumption by the factory method

