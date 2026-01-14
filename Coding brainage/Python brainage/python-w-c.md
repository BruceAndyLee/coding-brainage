### 1. Reference Counting

| Primitive         | Purpose                                                                                   |
| ----------------- | ----------------------------------------------------------------------------------------- |
| `Py_INCREF(obj)`  | Increment refcount. Use when you want to keep a reference longer than the API guarantees. |
| `Py_DECREF(obj)`  | Decrement refcount. Object freed if hits 0. Must not be NULL.                             |
| `Py_XINCREF(obj)` | NULL-safe INCREF                                                                          |
| Py_XDECREF(obj)   | NULL-safe DECREF                                                                          |
| Py_CLEAR(obj)     | DECREF + set to NULL atomically (prevents double-free)                                    |

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