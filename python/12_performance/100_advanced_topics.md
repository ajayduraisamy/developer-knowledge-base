# Advanced Topics - Pattern matching, AST, C extensions, PEP deep dives

## Introduction

Python is a deep language with many advanced features including metaclasses, AST manipulation, bytecode manipulation, C extensions, context variables, structural pattern matching, and more. These topics allow developers to extend, customize, and optimize Python at a fundamental level.

## Why It Is Important

Advanced Python knowledge enables developers to create frameworks, DSLs, and tools that would be impossible with basic Python. Understanding metaclasses, AST, and bytecode allows for metaprogramming. C extensions provide performance critical for scientific computing. Knowing PEPs and new features keeps your code modern and efficient.

## Syntax

`python
# Metaclass
class Meta(type):
    def __new__(mcs, name, bases, namespace):
        namespace['created_by'] = 'Meta'
        return super().__new__(mcs, name, bases, namespace)

class MyClass(metaclass=Meta):
    pass

# AST manipulation
import ast
tree = ast.parse('x = 1 + 2')
print(ast.dump(tree, indent=2))

# Structural pattern matching (Python 3.10+)
def match_example(value):
    match value:
        case 1:
            return 'one'
        case [x, y]:
            return f'list: {x}, {y}'
        case {'key': val}:
            return f'dict: {val}'
        case _:
            return 'other'

# Context variables
from contextvars import ContextVar
request_id: ContextVar[str] = ContextVar('request_id', default='unknown')

# Type narrowing
from typing import Union, Optional
def process(value: Union[int, str]) -> str:
    if isinstance(value, int):
        return str(value * 2)  # narrowed to int
    return value.upper()  # narrowed to str
`

## Examples

`python
import ast
import dis
import sys
from typing import Any, Dict, List, Optional, Union


def metaclass_example():
    print("Metaclass Example:")

    class ValidateMeta(type):
        def __new__(mcs, name, bases, namespace):
            for key, value in namespace.items():
                if key.startswith('validate_'):
                    print(f"  Found validator: {key}")
            return super().__new__(mcs, name, bases, namespace)

    class User(metaclass=ValidateMeta):
        def validate_email(self, email):
            return '@' in email

        def validate_age(self, age):
            return age >= 0

    print("  User class created with ValidateMeta")


metaclass_example()


def ast_example():
    print("\nAST Manipulation:")

    code = """
def greet(name):
    return f"Hello, {name}!"
"""
    tree = ast.parse(code)
    print(f"  AST dump:")
    print(f"  {ast.dump(tree, indent=2)}")


ast_example()
`

## Beginner Examples

`python
import ast
import dis
import sys
from typing import Union, Optional


def structural_pattern_matching():
    print("Structural Pattern Matching (Python 3.10+):")

    def process_value(value):
        match value:
            case 0:
                return "zero"
            case 1 | 2 | 3:
                return "small"
            case int(x) if x > 100:
                return "large int"
            case str(s) if len(s) > 10:
                return "long string"
            case [a, b, c]:
                return f"3-element list: {a}, {b}, {c}"
            case {'name': n, 'age': a}:
                return f"Person: {n}, {a}"
            case _:
                return "something else"

    print(f"  0 -> {process_value(0)}")
    print(f"  2 -> {process_value(2)}")
    print(f"  200 -> {process_value(200)}")
    print(f"  'hello world!!!' -> {process_value('hello world!!!')}")
    print(f"  [1, 2, 3] -> {process_value([1, 2, 3])}")
    print(f"  {{'name': 'Alice', 'age': 30}} -> {process_value({'name': 'Alice', 'age': 30})}")


structural_pattern_matching()


def type_narrowing():
    print("\nType Narrowing:")

    from typing import Union, Optional, List

    def process(data: Union[int, str, List[int]]) -> str:
        if isinstance(data, int):
            return f"Integer: {data * 2}"
        elif isinstance(data, str):
            return f"String: {data.upper()}"
        else:
            return f"List: sum={sum(data)}, len={len(data)}"

    print(f"  {process(42)}")
    print(f"  {process('hello')}")
    print(f"  {process([1, 2, 3])}")


type_narrowing()


def context_variables_basic():
    print("\nContext Variables:")

    from contextvars import ContextVar

    user_id: ContextVar[int] = ContextVar('user_id', default=0)

    def get_current_user():
        return user_id.get()

    def set_current_user(uid: int):
        user_id.set(uid)

    print(f"  Default user: {get_current_user()}")
    set_current_user(42)
    print(f"  After set: {get_current_user()}")


context_variables_basic()
`

## Intermediate Examples

`python
import ast
import dis
import sys
import inspect
from typing import Any, Dict, List, Optional
from contextvars import ContextVar


class MetaClassInDepth:
    def __init__(self):
        pass

    def metaclass_use_cases(self):
        print("Metaclass Use Cases:")
        print("  1. Singleton pattern")
        print("  2. ORM model definition (SQLAlchemy, Django)")
        print("  3. Registering subclasses automatically")
        print("  4. Adding methods/properties to all subclasses")
        print("  5. Validation of class definitions")

    def singleton_metaclass(self):
        print("\nSingleton Metaclass:")

        class SingletonMeta(type):
            _instances: Dict[type, Any] = {}

            def __call__(cls, *args, **kwargs):
                if cls not in cls._instances:
                    cls._instances[cls] = super().__call__(*args, **kwargs)
                return cls._instances[cls]

        class Database(metaclass=SingletonMeta):
            def __init__(self):
                self.connected = False

            def connect(self):
                self.connected = True

        db1 = Database()
        db2 = Database()
        print(f"  db1 is db2: {db1 is db2}")


meta = MetaClassInDepth()
meta.metaclass_use_cases()
meta.singleton_metaclass()


def ast_transform_example():
    print("\nAST Transform Example:")

    class AddLogger(ast.NodeTransformer):
        def visit_FunctionDef(self, node):
            log_call = ast.parse(
                f'print(f"Calling {node.name}")'
            ).body[0]
            node.body.insert(0, log_call)
            return node

    code = """
def add(a, b):
    return a + b
"""
    tree = ast.parse(code)
    transformer = AddLogger()
    transformed = transformer.visit(tree)
    ast.fix_missing_locations(transformed)
    compiled = compile(transformed, '<ast>', 'exec')
    namespace = {}
    exec(compiled, namespace)
    result = namespace['add'](3, 4)
    print(f"  Result: {result}")


ast_transform_example()


def bytecode_inspection():
    print("\nBytecode Inspection:")

    def simple_function(x):
        y = x + 1
        return y * 2

    print("  Bytecode for simple_function:")
    dis.dis(simple_function)


bytecode_inspection()
`

## Advanced Examples

`python
import ast
import dis
import sys
import struct
import inspect
from typing import Any, Dict, List, Optional, Union, Type
from contextvars import ContextVar


class AdvancedMetaProgramming:
    def __init__(self):
        pass

    def metaclass_with_validation(self):
        print("Metaclass with Field Validation:")

        class Field:
            def __init__(self, field_type, required=True):
                self.field_type = field_type
                self.required = required

        class ModelMeta(type):
            def __new__(mcs, name, bases, namespace):
                fields = {}
                for key, value in list(namespace.items()):
                    if isinstance(value, Field):
                        fields[key] = value
                        namespace.pop(key)
                namespace['_fields'] = fields
                return super().__new__(mcs, name, bases, namespace)

        class Person(metaclass=ModelMeta):
            name = Field(str, required=True)
            age = Field(int, required=True)
            email = Field(str, required=False)

        print(f"  Person fields: {Person._fields}")

    def ast_code_analysis(self):
        print("\nAST Code Analysis:")

        class ComplexityAnalyzer(ast.NodeVisitor):
            def __init__(self):
                self.complexity = 0

            def visit_If(self, node):
                self.complexity += 1
                self.generic_visit(node)

            def visit_While(self, node):
                self.complexity += 1
                self.generic_visit(node)

            def visit_For(self, node):
                self.complexity += 1
                self.generic_visit(node)

            def visit_FunctionDef(self, node):
                self.generic_visit(node)

        code = """
def complex_function(x):
    if x > 0:
        for i in range(x):
            if i % 2 == 0:
                print(i)
    while x > 0:
        x -= 1
"""
        tree = ast.parse(code)
        analyzer = ComplexityAnalyzer()
        analyzer.visit(tree)
        print(f"  Cyclomatic complexity: {analyzer.complexity}")

    def context_var_deep(self):
        print("\nContext Variables in Depth:")
        request_id = ContextVar('request_id')
        user_id = ContextVar('user_id')

        def handle_request(req_id, uid):
            request_id.set(req_id)
            user_id.set(uid)
            process_request()

        def process_request():
            print(f"  Processing request {request_id.get()} for user {user_id.get()}")

        handle_request('REQ-001', 42)
        handle_request('REQ-002', 100)

    def c_extension_basics(self):
        print("\nPython C Extension Basics:")
        print("  File structure:")
        print("""
    // mymodule.c
    #include <Python.h>

    static PyObject* myfunction(PyObject* self, PyObject* args) {
        int x;
        if (!PyArg_ParseTuple(args, "i", &x))
            return NULL;
        return PyLong_FromLong(x * 2);
    }

    static PyMethodDef MyMethods[] = {
        {"myfunction", myfunction, METH_VARARGS, "Doubles input"},
        {NULL, NULL, 0, NULL}
    };

    static struct PyModuleDef mymodule = {
        PyModuleDef_HEAD_INIT,
        "mymodule",
        NULL,
        -1,
        MyMethods
    };

    PyMODINIT_FUNC PyInit_mymodule(void) {
        return PyModule_Create(&mymodule);
    }
        """)

    def python_312_features(self):
        print("\nPython 3.12+ New Features:")
        print("  1. Type parameter syntax: def max[T](a: T, b: T) -> T")
        print("  2. override decorator: @override")
        print("  3. Improved error messages")
        print("  4. Perf profiling improvements")
        print("  5. More efficient bytecode")


adv = AdvancedMetaProgramming()
adv.metaclass_with_validation()
adv.ast_code_analysis()
adv.context_var_deep()
adv.c_extension_basics()
adv.python_312_features()
`

## Real-World Use Cases

`python
import ast
import sys
from typing import Any, Dict, List, Optional


def orm_implementation():
    print("Real-world: ORM Implementation with Metaclasses")
    print("  SQLAlchemy, Django ORM use metaclasses")
    print("  Define models declaratively")
    print("  Metaclass converts to SQL schemas")


def code_analysis_tools():
    print("Real-world: Code Analysis Tools")
    print("  Pylint, Flake8 use AST")
    print("  MyPy uses AST for type checking")
    print("  Black uses AST for code formatting")


def web_frameworks():
    print("Real-world: Web Frameworks")
    print("  Flask: decorators for routing")
    print("  FastAPI: type annotations for validation")
    print("  Django: metaclasses for model definition")


def dependency_injection():
    print("Real-world: Dependency Injection")
    print("  Use metaclasses for automatic dependency resolution")
    print("  Register implementations by interface")


def serialization_libraries():
    print("Real-world: Serialization")
    print("  Marshmallow: field definitions with validation")
    print("  Pydantic: type annotation-based validation")
    print("  attrs/dataclasses: auto-generated methods")


orm_implementation()
code_analysis_tools()
web_frameworks()
dependency_injection()
serialization_libraries()
`

## Common Mistakes

`python
import ast
from typing import Any


def mistake_1_overusing_metaclasses():
    print("Mistake 1: Overusing metaclasses")
    print("  Metaclasses add complexity")
    print("  Use decorators or inheritance when possible")


def mistake_2_unsafe_ast_execution():
    print("Mistake 2: Unsafe AST execution")
    print("  exec(compile(ast_node, ...)) can be dangerous")
    print("  Validate AST before execution")


def mistake_3_pattern_matching_order():
    print("Mistake 3: Wrong pattern matching order")
    print("  More specific patterns must come first")
    print("  Use _ as catch-all at the end")


def mistake_4_context_var_in_async():
    print("Mistake 4: Not using ContextVar.copy_context()")
    print("  In async code, context doesn't automatically propagate")
    print("  Use contextvars.copy_context() for coroutines")


def mistake_5_ignoring_type_narrowing():
    print("Mistake 5: Not using type narrowing")
    print("  Type checkers need isinstance/type assertions")
    print("  Use TypeGuard for custom narrowing")


def mistake_6_complex_c_extensions():
    print("Mistake 6: Writing C extensions when not needed")
    print("  Cython or ctypes may suffice")
    print("  C extensions require C API knowledge")


mistake_1_overusing_metaclasses()
mistake_2_unsafe_ast_execution()
mistake_3_pattern_matching_order()
mistake_4_context_var_in_async()
mistake_5_ignoring_type_narrowing()
mistake_6_complex_c_extensions()
`

## Best Practices

`python
import ast
from typing import Any, Type, TypeVar


def best_practice_1_prefer_decorators():
    print("Best Practice 1: Prefer decorators over metaclasses")
    print("  Decorators are simpler and more composable")
    print("  Metaclasses can lead to metaclass conflicts")


def best_practice_2_validate_ast():
    print("Best Practice 2: Validate AST before execution")
    print("  Never exec() unvalidated AST")
    print("  Use ast.literal_eval when possible")


def best_practice_3_pattern_matching_guards():
    print("Best Practice 3: Use guards in pattern matching")
    print("  match value:")
    print("    case int(x) if x > 0:")
    print("      print('positive')")


def best_practice_4_context_var_isolation():
    print("Best Practice 4: Isolate context variables")
    print("  Each request gets its own context")
    print("  Use contextlib.contextmanager for cleanup")


def best_practice_5_type_hints_always():
    print("Best Practice 5: Always use type hints")
    print("  Enables better tooling and documentation")
    print("  Use TypedDict, Literal, Final, Protocol")


def best_practice_6_stay_updated():
    print("Best Practice 6: Stay updated with PEPs")
    print("  Each Python version brings new features")
    print("  Read What's New in Python X.Y")


def best_practice_7_understand_before_using():
    print("Best Practice 7: Understand before using advanced features")
    print("  AST, bytecode, metaclasses are powerful but complex")
    print("  Document why you're using them")


best_practice_1_prefer_decorators()
best_practice_2_validate_ast()
best_practice_3_pattern_matching_guards()
best_practice_4_context_var_isolation()
best_practice_5_type_hints_always()
best_practice_6_stay_updated()
best_practice_7_understand_before_using()
`

## Interview Questions

`python
def interview_q1():
    print("Q: What are metaclasses and when would you use them?")
    print("A: Metaclasses are classes of classes. Used for ORMs,")
    print("   singletons, validation, and framework development.")


def interview_q2():
    print("Q: What is AST and how can it be used?")
    print("A: Abstract Syntax Tree. Used for code analysis,")
    print("   transformation (codemods), and compilation.")


def interview_q3():
    print("Q: How does Python 3.10+ pattern matching work?")
    print("A: match/case with destructuring, guards, and")
    print("   type checking. Similar to switch on steroids.")


def interview_q4():
    print("Q: What are context variables?")
    print("A: Variables scoped to execution context.")
    print("   Used for request-scoped data in async apps.")


def interview_q5():
    print("Q: How do you write a C extension for Python?")
    print("A: Include Python.h, define functions with PyArg_Parse,")
    print("   create method table, create module definition.")


def interview_q6():
    print("Q: What is type narrowing?")
    print("A: Refining a type within a code block.")
    print("   isinstance(), TypeGuard, x is not None.")


def interview_q7():
    print("Q: Name some important PEPs.")
    print("A: PEP 8 (style), PEP 484 (types), PEP 557 (dataclasses),")
    print("   PEP 572 (walrus operator), PEP 634 (pattern matching).")


interview_q1()
interview_q2()
interview_q3()
interview_q4()
interview_q5()
interview_q6()
interview_q7()
`

## Coding Challenges

`python
import ast
import dis
import sys
from typing import Any, Dict, List, Optional


def challenge_1_create_metaclass():
    print("Challenge 1: Create a debug metaclass")

    class DebugMeta(type):
        def __new__(mcs, name, bases, namespace):
            print(f"Creating class: {name}")
            print(f"  Methods: {[k for k in namespace if callable(namespace[k])]}")
            return super().__new__(mcs, name, bases, namespace)

    class MyService(metaclass=DebugMeta):
        def do_work(self):
            pass

        def cleanup(self):
            pass

    print("  MyService created with debug output")


def challenge_2_ast_find_strings():
    print("Challenge 2: Find all string literals in code with AST")

    class StringFinder(ast.NodeVisitor):
        def __init__(self):
            self.strings = []

        def visit_Constant(self, node):
            if isinstance(node.value, str):
                self.strings.append(node.value)
            self.generic_visit(node)

    code = """
def greet():
    name = "World"
    message = f"Hello, {name}!"
    return message
"""
    tree = ast.parse(code)
    finder = StringFinder()
    finder.visit(tree)
    print(f"  String literals found: {finder.strings}")


def challenge_3_pattern_match_json():
    print("Challenge 3: Parse JSON with pattern matching")

    def parse_json_response(data: dict) -> str:
        match data:
            case {'status': 'ok', 'data': {'id': id_, 'name': name}}:
                return f"Success: {name} (ID: {id_})"
            case {'status': 'error', 'message': msg}:
                return f"Error: {msg}"
            case {'status': 'pending', 'estimated_time': t}:
                return f"Pending, ETA: {t}s"
            case _:
                return "Unknown response"

    ok_response = {'status': 'ok', 'data': {'id': 1, 'name': 'Alice'}}
    error_response = {'status': 'error', 'message': 'Not found'}
    pending_response = {'status': 'pending', 'estimated_time': 30}
    print(f"  OK: {parse_json_response(ok_response)}")
    print(f"  Error: {parse_json_response(error_response)}")
    print(f"  Pending: {parse_json_response(pending_response)}")


def challenge_4_type_narrowing_practice():
    print("Challenge 4: Practice type narrowing")

    from typing import Union, List, Optional

    def process_items(items: Union[List[int], List[str], None]) -> int:
        if items is None:
            return 0
        if all(isinstance(x, int) for x in items):
            return sum(items)
        return len(items)

    print(f"  [1, 2, 3]: {process_items([1, 2, 3])}")
    print(f"  ['a', 'b']: {process_items(['a', 'b'])}")
    print(f"  None: {process_items(None)}")


challenge_1_create_metaclass()
challenge_2_ast_find_strings()
challenge_3_pattern_match_json()
challenge_4_type_narrowing_practice()
`

## Summary

Advanced Python topics include metaclasses for class customization, AST manipulation for code analysis/transformation, bytecode for low-level optimization, C extensions for performance, context variables for async scoping, pattern matching for expressive data handling, and type narrowing for safer code. Knowing these enables building frameworks, tools, and high-performance applications.

## Related Topics

- CPython Internals (93_cpython_internals.md)
- Cython (94_cython.md)
- Numba (95_numba.md)
- Profiling (91_profiling.md)
- Big O Notation (99_big_o.md)
