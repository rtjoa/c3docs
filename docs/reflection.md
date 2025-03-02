# Reflection

C3 allows both compile time and runtime reflection.

During compile time the type information may be directly used as compile time constants, the same data is then available dynamically at runtime.

*Note that not all reflection is implemented in the compiler at this point in time.*

## Compile time reflection

During compile time there are a number of compile time fields that may be accessed directly.

### Compile time variables

#### typeid

Returns the typeid for the given type. Typedefs will return the typeid of the underlying type. The typeid size is the same as that of an `iptr`.

    typeid x = Foo.typeid; 

#### $nameof

The basic name of the type or variable without module prefixes.

    struct Foo { ... }
    define Bar = Foo;
    int x;
    string a = $nameof(int[4]); // => "int[4]"
    string b = $nameof(Foo) // => "Foo"
    string c = $nameof(Bar); // => "Foo" 
    string d = $nameof(x); // => "x"

#### $qnameof

Same as $nameof, but includes the full module path: e.g. "baz::bar::Foo".

    module bar;
    struct Foo { ... }
    int x;
    string a = $qnameof(int[4]); // => "int[4]"
    string b = $qnameof(Foo); // => "bar::Foo"
    string c = $qnameof(Foo[4]); // => "bar::Foo[4]" 
    string d = $nameof(x); // => "bar::x"


#### $extnameof

Similar to $qnameof, this give the name of the type or variable or function 
as exported by the linker.

    module bar;
    int x;
    fn void test(int x) { }
    string a = $extnameof(x); // => "bar.x";
    string b = $extnameof(test); // => "bar.test"
    string c = $extnameof("test"); // => "bar.test"


#### $sizeof

Returns the size in bytes needed to store the type.

    struct Foo { long a; long b; }
    usize x = $sizeof(Foo); // 16
    usize y = $sizeof(int); // 4
    usize z = $sizeof(Foo, a); // 8

#### $alignof

Returns the alignment in bytes needed for the type.

    struct Foo { long a; long b; }
    usize x = $alignof(Foo); // 16
    usize y = $alignof(int); // 4
    usize z = $alignof(Foo, a); // 8

** The following are not yet implemented **

#### kindof

Returns the TypeKind of the variable.

```
struct Foo { ... }
union Bar { ... }
TypeKind a = Foo.kindof; // STRUCT
TypeKind b = Bar.kindof; // BAR
TypeKind c = int.kindof; // INTEGER
```

#### elementType

*Only available for array, vararray and subarray types.*
Returns the element (underlying) type of an array, vararray or subarray.

```
struct Foo { ... }
string x = Foo[].elementType.name; // "Foo"
string y = Foo[4].elementType.name; // "Foo"
string z = Foo[*].elementType.name; // "Foo"
```

#### baseType

*Only available for pointer types and failables.*
Returns the type the pointer points to. E.g. for `int*` the base type is `int`.

```
struct Foo { ... }
string x = (Foo*).baseType.name; // "Foo"
```

#### elements

*Only available for enum types.*
Returns an array containing the enum values in an enum.

```
enum Foo
{
    BAR,
    BAZ
}
string x = Foo.elements[0].name; // "BAR"
```

#### errors

*Only available for error types.*
Returns an array containing the error values in an enum.

```
errtype FooErr
{
    BAD_BAR,
    NO_BAZ
}
string x = Foo.errors[0].name; // "BAD_BAR"
```

#### fields

*Only available for struct types.*
Returns an array containing the fields in a struct.

```
struct Foo
{
    int x;
    Foo* z;
}
string x = Foo.fields[1].name; // "z"
```

#### variants

*Only available for union types.*
Returns an array of types representing the possible variants of the union.

```
union Foo
{
    int x;
    Foo* z;
    double d;
}
string x = Foo.variants[2].name; // "d"
```

#### signed

*Only available for integer types.*
Returns true for a signed number.

```
bool s1 = int.signed; // => true
bool s2 = uint.signed; // => false
```

#### length

*Only available for array types.*
Returns the length of the array.

```
usize len = int[4].length
```

#### associatedValues

*Only available for enums.*
Returns an array containing the types of associated values if any.

```
enum Foo : int(double d, string s)
{
    BAR(1.0, "normal"),
    BAZ(2.0, "exceptional")
}
string s = Foo.associatedValues[0].name; // "double"
```

#### returnType

*Only available for function types.*
Returns the type of the return type.

```
define TestFunc = fn int(int, double);
string s = TestFunc.returnType.name; // "int"
```

#### params

*Only available for function types.*
Returns a list of all parameters.

```
define TestFunc = fn int(int, double);
string s = TestFunc.params[1].name; // "double"
```

#### errors

*Only available for functions.*
Returns a list containing all errors returned, or nil otherwise. An empty list
is returned on not returning. 

*NOTE: This may change.*
```
errtype SomeError 
{
    FOO 
}
errtype SomeOtherError 
{ 
    BAR
}

fn void! foo()
{
    if (someReason()) return SomeError.FOO!;
    bar()!!;
}
fn void! bar()
{
    return SomeOtherError.BAR!;
}

string s = foo.errors[1].name; // "SomeOtherError.FOO"
int errors = bar.errors.size == 1;
```


## Runtime reflection

During runtime it's also possible to retrieve information by way of a typeid. Using typeid may implicitly cast into a `TypeInfo *`. This is a struct that contains data for the underlying type:

```
struct TypeData
{
    typeid typeId;
    TypeKind kind;
    int size;
    int alignment;
    char* name;
    char* fullName;
}
```

This structure is the base type, and the actual struct will can be `TypeError`, `TypeArray`, `TypeInteger` and so on. The definition of these are found in the module `system::builtin`.

The `builtin` module further offers several ways to search for different types, returning the `TypeData *` directly:

* `functionByName`
* `structByName`
* `unionByName`
* `enumByName`
* `typeByName`
* `errorByName`
* `opaqueByName`
* `moduleByName`

The TypeData substructs further offers functions for retrieving fields by name and other conveniences.

## TypeKind

The TypeKind enum contains the basic kinds of types:

* `VOID` 
* `INTEGER`
* `REAL`
* `BOOL`
* `POINTER`
* `UNION`
* `ENUM`
* `ERROR`
* `ARRAY`
* `VARARRAY`
* `SUBARRAY`
* `FUNC`
* `TYPEID`
* `STRING`
* `OPAQUE`
* `FAILABLE`
