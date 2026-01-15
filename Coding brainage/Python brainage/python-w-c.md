For alphachads https://docs.python.org/3/c-api/index.html
### 1. Reference Counting

| Primitive         | Purpose                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| `Py_INCREF(obj)`  | Increment refcount. Use when you want to keep a reference longer than the API guarantees. |
| `Py_DECREF(obj)`  | Decrement refcount. Object freed if hits 0. Must not be NULL.                             |
| `Py_XINCREF(obj)` | NULL-safe INCREF                                                                          |
| Py_XDECREF(obj)   | NULL-safe DECREF                                                                          |
| Py_CLEAR(obj)     | DECREF + set to NULL atomically (prevents double-free)                                    |
|                   |                                                                                           |

Compiler example: After parsing an AST node and inserting it into a parent node's children list, you'd Py_DECREF the child since the list now owns it.

---

### 2. Object Creation

|Primitive|Purpose|
|---|---|
|Py_BuildValue(format, ...)|Create tuple/list/dict/primitives from C values. Format string like printf.|
|PyLong_FromLong(n)|Create Python int from C long|
|PyFloat_FromDouble(d)|Create Python float from C double|
|PyUnicode_FromString(s)|Create Python str from C string (UTF-8)|
|PyBytes_FromStringAndSize(s, len)|Create Python bytes|
|PyBool_FromLong(n)|Create Python bool (0→False, else→True)|
|PyList_New(size)|Create list with size slots (all NULL initially)|
|PyTuple_New(size)|Create tuple with size slots|
|PyDict_New()|Create empty dict|
|Py_None, Py_True, Py_False|Singleton references (must INCREF if returning)|

Py_BuildValue format codes:

```text
"i"   → int
"l"   → long  
"d"   → double
"s"   → char* → str
"O"   → PyObject* (INCREFs it)
"N"   → PyObject* (steals reference, no INCREF)
"(ii)" → tuple of two ints
"[ii]" → list of two ints
"{s:i}" → dict {"key": int}
```

Compiler example: Building an AST node:

```c
// Node: {"type": "BinaryOp", "op": "+", "left": left_node, "right": right_node}

PyObject* node = Py_BuildValue(
    "{s:s, s:s, s:N, s:N}",
    "type", "BinaryOp",
    "op", "+",
    "left", left_node,   // N steals reference
    "right", right_node  // N steals reference
);
```

---

### 3. Object Access (Reading)

| Primitive                           | Returns     | Ownership                        |
| ----------------------------------- | ----------- | -------------------------------- |
| PyList_GetItem(list, i)             | PyObject*   | Borrowed (don't DECREF)          |
| PyList_GET_ITEM(list, i)            | PyObject*   | Borrowed, no bounds check        |
| PyTuple_GetItem(tuple, i)           | PyObject*   | Borrowed                         |
| PyDict_GetItem(dict, key)           | PyObject*   | Borrowed                         |
| PyDict_GetItemString(dict, "key")   | PyObject*   | Borrowed                         |
| PyLong_AsLong(obj)                  | C long      | N/A                              |
| PyFloat_AsDouble(obj)               | C double    | N/A                              |
| PyUnicode_AsUTF8(obj)               | const char* | Borrowed (valid while obj lives) |
| PyBytes_AsString(obj)               | char*       | Borrowed                         |
| PyObject_GetAttrString(obj, "attr") | PyObject*   | New reference (must DECREF)      |

Compiler example: Reading operator from token dict:

```c
PyObject* op = PyDict_GetItemString(token, "value");  // borrowed
const char* op_str = PyUnicode_AsUTF8(op);            // borrowed
if (strcmp(op_str, "+") == 0) { ... }
// No DECREF needed
```

---

### 4. Object Mutation (Writing)

|Primitive|Ownership of value|
|---|---|
|PyList_SetItem(list, i, obj)|Steals reference to obj|
|PyList_SET_ITEM(list, i, obj)|Steals, no bounds check|
|PyList_Append(list, obj)|Does NOT steal (INCREFs internally)|
|PyTuple_SetItem(tuple, i, obj)|Steals (only before tuple is used!)|
|PyDict_SetItem(dict, key, val)|Does NOT steal (INCREFs both)|
|PyDict_SetItemString(dict, "k", val)|Does NOT steal val|
|PyObject_SetAttrString(obj, "attr", val)|Does NOT steal|

The steal vs non-steal distinction is critical:

```c
// PyList_Append does NOT steal → must DECREF after

PyObject* item = PyLong_FromLong(42);  // refcnt=1
PyList_Append(list, item);              // refcnt=2
Py_DECREF(item);                        // refcnt=1, list owns it

// PyList_SetItem STEALS → must NOT DECREF after
PyObject* item = PyLong_FromLong(42);  // refcnt=1
PyList_SetItem(list, 0, item);          // refcnt=1, list owns it

// No DECREF!
```

Steal is like... you thought you got that increase in the reference-counter, but it got STOLEN from you (decremented), now you don't have to decrement it your self at the end of your block;
Natural course of use: for complicated initialization logic of stuff

---

### 5. Type Checking

|Primitive|Purpose|
|---|---|
|PyLong_Check(obj)|Is it an int?|
|PyFloat_Check(obj)|Is it a float?|
|PyUnicode_Check(obj)|Is it a str?|
|PyList_Check(obj)|Is it a list?|
|PyDict_Check(obj)|Is it a dict?|
|PyCallable_Check(obj)|Can it be called?|
|PyObject_IsInstance(obj, type)|isinstance() equivalent|
|Py_TYPE(obj)|Get type object|

Compiler example: Validating parsed literal:

```c
if (PyLong_Check(literal)) {
    // emit integer constant
} else if (PyFloat_Check(literal)) {
    // emit float constant
} else if (PyUnicode_Check(literal)) {
    // emit string constant
}
```

---

### 6. Calling Python from C

|Primitive|Purpose|
|---|---|
|PyObject_Call(callable, args_tuple, kwargs_dict)|General call|
|PyObject_CallObject(callable, args_tuple)|Call with args only|
|PyObject_CallFunction(callable, format, ...)|Call with Py_BuildValue-style args|
|PyObject_CallMethod(obj, "method", format, ...)|Call method|
|PyObject_CallFunctionObjArgs(callable, arg1, arg2, ..., NULL)|Call with PyObject* args|

Compiler example: Calling Python's error handler from C:

```c
PyObject* handler = PyObject_GetAttrString(compiler_obj, "report_error");
PyObject* result = PyObject_CallFunction(handler, "sis", 
    "SyntaxError", line_num, "Unexpected token");

Py_DECREF(result);
Py_DECREF(handler);
```

---

### 7. Error Handling

|Primitive|Purpose|
|---|---|
|PyErr_SetString(exc_type, msg)|Raise exception|
|PyErr_Format(exc_type, fmt, ...)|Raise with formatted message|
|PyErr_Occurred()|Check if exception pending|
|PyErr_Clear()|Clear pending exception|
|PyErr_Fetch(&type, &value, &tb)|Get exception details|
|PyErr_NoMemory()|Raise MemoryError|
|PyExc_RuntimeError, PyExc_ValueError, etc.|Built-in exception types|

Pattern:

```c
if (some_error_condition) {
    PyErr_SetString(PyExc_ValueError, "Invalid syntax at line 42");
    return NULL;  // Signal error to caller
}
```

All functions returning PyObject* return NULL on error. This is the convention.

---

### 8. Argument Parsing (for C functions called from Python)

| Primitive                                                      | Purpose                  |
| -------------------------------------------------------------- | ------------------------ |
| PyArg_ParseTuple(args, format, ...)                            | Parse positional args    |
| PyArg_ParseTupleAndKeywords(args, kwargs, format, kwlist, ...) | Parse pos + keyword args |
| PyArg_UnpackTuple(args, name, min, max, ...)                   | Simple unpacking         |

Format codes (similar to Py_BuildValue):

```c
static PyObject* parse_expr(PyObject* self, PyObject* args) {
    const char* source;
    int start_pos;
    if (!PyArg_ParseTuple(args, "si", &source, &start_pos)) {
        return NULL;  // Exception already set
    }
    // ... parse ...
}
```

---

### 9. Module & Function Definition

```c
// Define a C function callable from Python
static PyObject* my_parse(PyObject* self, PyObject* args) {
    // ...
}

// Method table
static PyMethodDef MyMethods[] = {
    {"parse", my_parse, METH_VARARGS, "Parse source code"},
    {NULL, NULL, 0, NULL}  // Sentinel
};

// Module definition
static struct PyModuleDef mymodule = {
    PyModuleDef_HEAD_INIT,
    "mycompiler",           // module name
    "A compiler module",    // docstring
    -1,                     // per-interpreter state size (-1 = global)
    MyMethods
};

// Module init (called on import)
PyMODINIT_FUNC PyInit_mycompiler(void) {
    return PyModule_Create(&mymodule);
}
```

---

### 10. NumPy-Specific (what your code uses)

|Primitive|Purpose|
|---|---|
|import_array()|Initialize NumPy C API (must call once)|
|PyArray_FROM_OTF(obj, type, flags)|Convert to ndarray with type/flags|
|PyArray_DATA(arr)|Get raw pointer to data buffer|
|PyArray_DIM(arr, n)|Get size of dimension n|
|PyArray_SimpleNew(ndim, dims, type)|Create new array|
|NPY_ARRAY_IN_ARRAY|Flag: contiguous, aligned, not writeable|
|NPY_DOUBLE, NPY_INT32, etc.|Type codes|

---

## Quick Reference: Ownership Patterns

|Pattern|Example|Your responsibility|
|---|---|---|
|New reference|PyList_New(), Py_BuildValue()|Must DECREF when done|
|Borrowed reference|PyList_GetItem(), PyDict_GetItem()|Must NOT DECREF|
|Stolen reference|PyList_SetItem(), Py_BuildValue "N"|Receiver owns it, don't DECREF|
|Returning to Python|return result;|Python takes ownership|

---

## Ownership Transfer

Key insight: "Ownership" just means "who is responsible for calling DECREF."

When you create an object, refcount starts at 1 — you own that reference. Ownership transfer means you give that responsibility to someone else.

### Scenario: Building a tree

```c
// Create child node
PyObject* child = Py_BuildValue("{s:s}", "type", "Literal");  // refcnt=1, YOU own it

// Create parent with children list
PyObject* children = PyList_New(0);           // refcnt=1, YOU own it
PyObject* parent = Py_BuildValue("{s:s, s:N}",
    "type", "BinaryOp",
    "children", children);                     // "N" steals → parent dict owns children list

// children now owned by parent, don't DECREF children
// Now add child to the list
```

Two ways to add child:
```c

// Option A: Append (does NOT steal)

PyList_Append(children, child);  // list INCREFs child → refcnt=2
Py_DECREF(child);                // YOUR ref gone → refcnt=1, list owns it

// Option B: SetItem (STEALS)
PyList_SetItem(children, 0, child);  // list takes YOUR ref → refcnt=1, list owns it

// Do NOT Py_DECREF(child) here!
```

After either option: The children list owns child. When parent dict is eventually freed, it DECREFs children, which DECREFs each element, which frees child. The whole tree cleans up recursively.

### Explicit ownership transfer pattern

If you have an object and want to "give it away":

```c
PyObject* node = create_node();  // refcnt=1, you own it

// Option 1: Use "N" in Py_BuildValue (steals)
PyObject* wrapper = Py_BuildValue("(N)", node);  // wrapper owns node now

// Option 2: Use PyList_SetItem (steals)
PyList_SetItem(list, idx, node);  // list owns node now

// Option 3: Manual transfer
PyDict_SetItem(dict, key, node);  // dict INCREFs → refcnt=2
Py_DECREF(node);                  // your ref gone → refcnt=1, dict owns it
```

The pattern for "Option 3" is how you transfer ownership when the API doesn't steal:

1. Let container `INCREF` it
2. `DECREF` your own reference
3. Container is now sole owner

---

## Memory Structure

### 1. Is PyList_New malloc'd contiguously?

No. A Python list is not contiguous data. It's:

```
// ...opus 4.5 drew this btw...
┌─────────────────────┐
│ PyListObject        │  ← malloc'd
│  - ob_refcnt        │
│  - ob_type          │
│  - ob_size (length) │
│  - allocated        │
│  - **ob_item ───────┼──────┐
└─────────────────────┘      │
                             ▼
                    ┌─────────────────────┐
                    │ PyObject* array     │  ← malloc'd (array of pointers)
                    │  [0] ──────────────────→ PyLongObject (42)     ← malloc'd
                    │  [1] ──────────────────→ PyUnicodeObject "hi"  ← malloc'd
                    │  [2] ──────────────────→ PyDictObject {...}    ← malloc'd
                    │  ...                │
                    └─────────────────────┘
```

Three separate allocations:

1. The PyListObject header
2. The array of PyObject* pointers (resized on growth)
3. Each element object (wherever they happen to be in heap)

Contrast with NumPy:

```
// ...opus 4.5 drew this btw...
┌─────────────────────┐
│ PyArrayObject       │  ← malloc'd
│  - ob_refcnt        │
│  - *data ───────────┼──────┐
│  - shape, strides   │      │
└─────────────────────┘      │
                             ▼
                    ┌─────────────────────┐
                    │ Raw data buffer     │  ← malloc'd (contiguous!)
                    │  [1.0][2.0][3.0]... │  ← actual doubles/ints/etc
                    └─────────────────────┘
```

NumPy stores raw values contiguously. Python lists store pointers to scattered objects.

Why this matters for performance:

- NumPy: `data[i]` → single pointer dereference, cache-friendly

- PyList: `ob_item[i]` → pointer to pointer, then unbox → cache-unfriendly

This is why the C code in your project extracts raw arrays via PyArray_DATA() and works with C primitives, not Python objects.

---

### 2. What is NULL-safe INCREF/DECREF?

Exactly what you guessed:

```c
// Py_DECREF implementation (simplified)
#define Py_DECREF(obj) \
    do { \
        if (--((PyObject*)(obj))->ob_refcnt == 0) \
            _Py_Dealloc(obj); \
    } while (0)
// Py_XDECREF implementation (simplified)  
#define Py_XDECREF(obj) \
    do { \
        if ((obj) != NULL) \
            Py_DECREF(obj); \
    } while (0)
```

`Py_DECREF(NULL)` → crash (dereferencing NULL to access ob_refcnt)
`Py_XDECREF(NULL)` → no-op (checks first)


When to use which:
```c
// You KNOW it's not NULL (just created it, or checked earlier)
PyObject* obj = PyList_New(0);
// ... use obj ...

Py_DECREF(obj);  // Fine, we know it's valid
// Cleanup code where things MIGHT be NULL (error paths)

void cleanup(MyContext* ctx) {
    Py_XDECREF(ctx->maybe_list);   // Could be NULL if init failed
    Py_XDECREF(ctx->maybe_dict);   // Could be NULL
}
```

The X variants are for defensive cleanup when you're not sure if allocation succeeded. The regular variants are for normal paths where you know the object exists.

---

### Summary Table

|Concept|Explanation|
|---|---|
|Ownership|= responsibility to DECREF|
|Transfer via steal|PyList_SetItem, Py_BuildValue "N" — takes your ref|
|Transfer via INCREF+DECREF|Container INCREFs, you DECREF — container now sole owner|
|PyList memory|Header + pointer array + scattered objects|
|NumPy memory|Header + contiguous data buffer|
|Py_DECREF|Crashes on NULL|
|Py_XDECREF|NULL-safe (no-op if NULL)|
