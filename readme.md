# GROUP 31 – Software Construction HS24 Assignment II
**Extended the LGL interpreter by adding support for extra data structures, operations, and a hierarchical function tracing system with timing analysis.**


## Table of Contents
- [Overview](#overview): Explanation and design decisions
- [Code Documentation](#code-documentation): Detailed overview of each method
- [Disclaimer](#disclaimer): Use of AI tools and commit distribution explanation



## Overview
This section outlines our general approach and the rationale behind key design decisions. 
For method-specific details, please refer to the [Code Documentation](#code-documentation). 

The basis for our interpreter was the source code provided in the book, which can also be found at https://third-bit.com/sdxpy/interp/
.
Our approach was to build on top of this existing implementation by adding new functions and features without modifying the original code structure.
This method made it easier to extend the interpreter step by step and will be especially useful for adding further functionalities in the future.


## Code Documentation
This section explains the new functions and functionalities we added to the interpreter.
Each addition was built on top of the existing base code to extend its behavior without changing the original implementation.

### Arithmetic Operations
The arithmetic functions perform various mathematical operations by evaluating the arguments passed to them.

`do_multiplication`: Similar to `do_add`, this function multiplies two numbers after ensuring that the correct number of arguments is provided.

```python
def do_multiplication(env, args):
    assert len(args) == 2
    return do(env, args[0]) * do(env, args[1])
```

`do_division`: Divides the first number by the second. It ensures exactly two arguments are provided, evaluates both, and returns the quotient.
```python
def do_division(env, args):
    assert len(args) == 2
    return do(env, args[0]) / do(env, args[1])
```

`do_modulo`: Calculates the remainder when dividing the first number by the second. It ensures exactly two arguments are provided, evaluates both, and returns the modulo result.
```python
def do_modulo(env, args):
    assert len(args) == 2
    return do(env, args[0]) % do(env, args[1])
```

`do_power`: Raises the first number to the power of the second. It ensures exactly two arguments are provided, evaluates both, and returns the result of the exponentiation.
```python
def do_power(env, args):
    assert len(args) == 2
    return do(env, args[0]) ** do(env, args[1])
```

## Comparison Operations

`do_less_than`: Checks if the first value is less than the second (`<`). It evaluates both arguments and returns `1` if the condition is true, otherwise `0`.
```python
def do_less_than(env, args):   # 
    assert len(args) == 2
    return 1 if do(env, args[0]) < do(env, args[1]) else 0
```

`do_greater_than`: Checks if the first value is greater than the second (`>`). It evaluates both arguments and returns `1` if true, otherwise `0`.
```python
def do_greater_than(env, args):   # >
    assert len(args) == 2
    return 1 if do(env, args[0]) > do(env, args[1]) else 0
```

`do_less_than_or_equal`: Checks if the first value is less than or equal to the second (`<=`). It evaluates both arguments and returns `1` if true, otherwise `0`.
```python
def do_less_than_or_equal(env, args):   # <=
    assert len(args) == 2
    return 1 if do(env, args[0]) <= do(env, args[1]) else 0
```

`do_greater_than_or_equal`: Checks if the first value is greater than or equal to the second (`>=`). It evaluates both arguments and returns `1` if true, otherwise `0`.
```python
def do_greater_than_or_equal(env, args):   # >=
    assert len(args) == 2
    return 1 if do(env, args[0]) >= do(env, args[1]) else 0
```

`do_equal`: Checks if both values are equal (`==`). It evaluates both arguments and returns `1` if they are equal, otherwise `0`.
```python
def do_equal(env, args):   # ==
    assert len(args) == 2
    return 1 if do(env, args[0]) == do(env, args[1]) else 0
```

`do_not_equal`: Checks if the values are not equal (`!=`). It evaluates both arguments and returns `1` if they are different, otherwise `0`.
```python
def do_not_equal(env, args):   # !=
    assert len(args) == 2
    return 1 if do(env, args[0]) != do(env, args[1]) else 0
```

## Boolean Operations

`do_and`: Performs a logical AND operation on two values. It evaluates both arguments and returns `1` only if both are true (non-zero), otherwise returns `0`.
```python
def do_and(env, args):
    assert len(args) == 2
    return 1 if (do(env, args[0]) and do(env, args[1])) else 0
```

`do_or`: Performs a logical OR operation on two values. It evaluates both arguments and returns `1` if at least one is true (non-zero), otherwise returns `0`.
```python
def do_or(env, args):
    assert len(args) == 2
    return 1 if (do(env, args[0]) or do(env, args[1])) else 0
```

`do_not`: Performs a logical NOT operation on a single value. It evaluates the argument and returns `1` if it is false (zero), otherwise returns `0`.
```python
def do_not(env, args):
    assert len(args) == 1
    return 1 if not do(env, args[0]) else 0
```

## Loop Support

`do_dountil`: Implements a do-until loop construct. It executes a body of code at least once, then checks a condition. The loop continues to execute the body and check the condition until the condition evaluates to `1` (true). This is similar to a do-while loop but with inverted logic. The function takes two arguments: the body (typically a sequence of operations) and the condition to check after each iteration. It returns `0` when the loop completes.
```python
def do_dountil(env, args):
    """
    Gebrauch:
    ["dountil",
        ["seq", ...body...],
        condition
    ]
    Runned body, checkt dann condition;
    wiederholt sich until condition ist true (==1).
    """
    # do ... until loop
    # args[0] = body, args[1] = condition
    assert len(args) == 2
    body = args[0]
    cond = args[1]
    
    done = False
    while not done:
        do(env, body)
        if do(env, cond) == 1:
            done = True
    return 0
```

## Array Operations

`do_array_new`: Creates a new array of a specified size, initialized with zeros. It evaluates the size argument to determine how many elements the array should contain, then uses a loop to append zeros to the array until it reaches the desired size. The function returns the newly created array.
```python
def do_array_new(env, args):
    """ Gebrauch: ["set", "A", ["array_new", 5]] """
    assert len(args) == 1
    size = do(env, args[0])
    assert isinstance(size, int) and size >= 0
    array = []
    i = 0
    while i < size:
        array.append(0)
        i = i + 1
    return array
```

`do_array_get`: Retrieves an element from an array at a specific index. It evaluates both the array and the index arguments, checks that the array is a list, and verifies that the index is within bounds. If the index is valid, it returns the element at that position; otherwise, it raises an IndexError.
```python
def do_array_get(env, args):
    """ Gebrauch:  ["set", "elem_A2", ["array_get", ["get", "A"], 2]] """
    assert len(args) == 2
    array = do(env, args[0])
    index = do(env, args[1])
    assert isinstance(array, list)
    if index < 0 or index >= len(array):
        raise IndexError(f"Index {index} out of bounds")
    return array[index]
```

`do_array_set`: Sets the value of an array element at a specific index. It evaluates the array, index, and value arguments, checks that the array is a list and the index is within bounds, then assigns the value to the specified position in the array. It returns the value that was set.
```python
def do_array_set(env, args):
    """ Gebrauch:  ["array_set", ["get", "A"], 0, 10] """ 
    assert len(args) == 3
    array = do(env, args[0])
    index = do(env, args[1])
    value = do(env, args[2])
    assert isinstance(array, list)
    if index < 0 or index >= len(array):
        raise IndexError(f"Index {index} out of bounds")
    array[index] = value
    return value
```

`do_array_size`: Returns the number of elements in an array. It evaluates the array argument, checks that it is a list, and returns its length.
```python
def do_array_size(env, args):
    """ Gebrauch:  ["set", "A_size", ["array_size", ["get", "A"]]] """
    assert len(args) == 1
    array = do(env, args[0])
    assert isinstance(array, list)
    return len(array)
```

`do_array_concat`: Concatenates two arrays into a single new array. It evaluates both array arguments, checks that both are lists, and returns a new array containing all elements from the first array followed by all elements from the second array.
```python
def do_array_concat(env, args):
    """ Gebrauch:  ["set", "C", ["array_concat", ["get", "A"], ["get", "B"]]] """
    assert len(args) == 2
    array1 = do(env, args[0])
    array2 = do(env, args[1])
    assert isinstance(array1, list) and isinstance(array2, list)
    return array1 + array2
```

## Set Operations

`do_set_new`: Creates a new empty set. It takes no arguments and returns a fresh set data structure that can be used to store unique values.
```python
def do_set_new(env, args):
    """Gebrauch: ["set", "S", ["set_new"]]"""
    assert len(args) == 0
    return set()
```

`do_set_insert`: Inserts a value into a set. It evaluates both the set and the value arguments, checks that the first argument is a set, then attempts to add the value. If the value was not already in the set, it adds it and returns `1`; if the value already existed, it returns `0`.
```python
def do_set_insert(env, args):
    """Gebrauch: ["set_insert", ["get", "K"], 1]"""
    assert len(args) == 2
    s = do(env, args[0])
    value = do(env, args[1])
    assert isinstance(s, set)

    if value not in s:
        s.add(value)
        return 1  
    else:
        return 0
```

`do_set_exists`: Checks whether a value exists in a set. It evaluates both the set and the value arguments, checks that the first argument is a set, and returns `1` if the value is found in the set, otherwise returns `0`.
```python
def do_set_exists(env, args):
    """Gebrauch: ["set", "check_2_in_K", ["set_exists", ["get", "K"], 2]]   """
    assert len(args) == 2
    s = do(env, args[0])
    value = do(env, args[1])
    assert isinstance(s, set)
    if value in s:
        return 1
    else:
        return 0
```

`do_set_size`: Returns the number of elements in a set. It evaluates the set argument, checks that it is a set, and returns its size.
```python
def do_set_size(env, args):
    """Gebrauch:  ["set", "size_K", ["set_size", ["get", "K"]]]  """
    assert len(args) == 1
    s = do(env, args[0])
    assert isinstance(s, set)
    return len(s)
```

`do_set_merge`: Merges two sets into a single new set containing all unique elements from both sets. It evaluates both set arguments, checks that both are sets, creates a new empty set, then iterates through both input sets to add all elements to the merged set. The function returns the newly created merged set.
```python
def do_set_merge(env, args):
    """Gebrauch: ["set", "M", ["set_merge", ["get", "K"], ["get", "L"]]]  """
    assert len(args) == 2
    s1 = do(env, args[0])
    s2 = do(env, args[1])
    assert isinstance(s1, set) and isinstance(s2, set)
    merged = set()
    for item in s1:
        merged.add(item)
    for item in s2:
        if item not in merged:
            merged.add(item)
    return merged
```

## Functional Programming

`do_func`: Creates a function object that can be used with functional programming operations like map, reduce, and filter. It takes two arguments: a list of parameter names and a body expression. The function returns a dictionary containing the parameter list and the body, which can later be applied to values. This allows users to define reusable function definitions in the language.
```python
def do_func(env, args):
    """ Gebrauch: ["set", "sq_func", ["func", ["n"], ["multiplication", "n", "n"]]] """
    assert len(args) == 2
    parameter = args[0]
    body = args[1]
    return {"parameter": parameter, "body": body}
```

`do_map`: Applies a function to each element of an array and returns a new array with the results. It evaluates both the array and the function arguments, then iterates through each element in the array. For each element, it creates a local environment where the function's parameter is bound to the current element, evaluates the function body in this local environment, and appends the result to a new array. The function also handles automatic variable resolution by wrapping bare variable names in `["get", ...]` syntax if they're not already operators.
```python
def do_map(env, args):
    """ Gebrauch: ["set", "res", ["map", ["get", "A"], ["get", "sq_func"]]] """
    assert len(args) == 2
    array = do(env, args[0])
    func = do(env, args[1])  # func = { "parameter": ["n"], "body": ["multiplication", "n", "n"] }
    assert isinstance(array, list)
    assert isinstance(func, dict)

    new_array = []
    parameter_name = func["parameter"][0]  # wäre "n" im Beispiel

    for element in array:
        # lokale Kopie damit wir die äußere Umgebung nicht verändern
        local_env = env.copy()
        local_env[parameter_name] = element  # z.B. {"n": 3}

        body = func["body"] # hier: ["multiplication", "n", "n"]

        # falls der Body nur Variablennamen enthält (z.B. "n"), mache sie automatisch mit ["get", ...]
        if isinstance(body, list):
            fixed_body = []
            for teil in body:
                if isinstance(teil, str) and teil not in OPS:
                    fixed_body.append(["get", teil])
                else:
                    fixed_body.append(teil)
            body = fixed_body

        value = do(local_env, body)  # z.B. ["multiplication", [["get", "n"],["get", "n"]]] und local_env={"n":3}
        new_array.append(value)

    return new_array
```

`do_reduce`: Reduces an array to a single value by repeatedly applying a binary function. It evaluates both the array and the function arguments, starting with the first element as the initial result. The function then iterates through the remaining elements, creating a local environment where the function's two parameters are bound to the current accumulated result and the current array element. It evaluates the function body with these bindings and updates the result. If the array is empty, it returns 0. The final accumulated result is returned after processing all elements.
```python
def do_reduce(env, args):
    """ Gebrauch: ["set", "summe_A", ["reduce", ["get", "A"], ["get", "do_add"]]] """
    assert len(args) == 2
    array = do(env, args[0])
    func = do(env, args[1]) # args[1] = ["get", "do_add"]
    assert isinstance(array, list)
    assert isinstance(func, dict)

    # fallas array leer
    if len(array) == 0:
        return 0

    # start with first element
    result = array[0]
    i = 1
    local_env = env.copy()
    while i < len(array):
        #local_env = env.copy()
        parameter1 = func["parameter"][0] # func = { "parameter": ["x", "y"], "body": ["add", ["get", "x"], ["get", "y"]]}
        parameter2 = func["parameter"][1]
        local_env[parameter1] = result
        local_env[parameter2] = array[i]
        result = do(local_env, func["body"])
        i = i + 1
    return result
```

`do_filter`: Filters an array by keeping only elements that satisfy a predicate function. It evaluates both the array and the function arguments, then iterates through each element in the array. For each element, it creates a local environment where the function's parameter is bound to the current element, evaluates the function body (which should return a boolean condition), and adds the element to the filtered result array only if the condition evaluates to `1` or `True`. The function returns a new array containing only the elements that passed the filter condition.
```python
def do_filter(env, args):
    """ Gebrauch: ["set", "filtered", ["filter", ["get", "A"], ["get", "larger_ten_func"]]] """
    assert len(args) == 2
    array = do(env, args[0])
    func = do(env, args[1]) # func = {"parameter": ["n"], "body": ["greater_than", ["get", "n"], 10]}
    assert isinstance(array, list)
    assert isinstance(func, dict)

    filtered = []
    parameter_name = func["parameter"][0] # wäre "n" im Bsp.
    for element in array:
        local_env = env.copy()
        local_env[parameter_name] = element
        condition = do(local_env, func["body"])
        if condition == 1 or condition is True:
            filtered.append(element)
    return filtered
```

## Trace Functionality

`TRACE_ENABLED`: A global flag that controls whether function call tracing is active. It is set to `False` by default but can be enabled by passing the `--trace` command-line argument when running the program.
```python
TRACE_ENABLED = False # hier by default FALSE, aber in der Main function wird dann richtig
```

`TRACE_LOG`: A global list that stores top-level function call entries. Each entry contains the function name, execution time, and any nested function calls. This list is used to print the complete trace tree at the end of program execution.
```python
TRACE_LOG = [] # A list of top-level function call entries, which store the name, time, and nested calls for printing later.
```

`TRACE_STACK`: A global stack (implemented as a list) used during execution to track which functions are currently running. This helps build a hierarchical tree structure of function calls, showing which functions were called from within other functions.
```python
TRACE_STACK = [] # A stack (list) used during execution to keep track of which functions are currently running. this helps build a hierarchical tree of calls.
```

`do_call`: Calls a user-defined function with given arguments. It evaluates the function name and all argument values, retrieves the function definition from the environment, and creates a local environment where the function's parameters are bound to the provided values. The function tracks execution time and builds a trace entry showing the function name and duration. If tracing is enabled, it maintains a hierarchical stack of function calls, recording parent-child relationships between nested calls. After executing the function body, it records the execution time in milliseconds and pops the trace entry from the stack. Top-level calls (those with no parent) are added to the trace log for later display.
```python
def do_call(env, args): # calls user defined function
    assert len(args) >= 1 #at least one argument... the name of the function 
    func_name = args[0] # ["call", "get_cube_power", ["get", "a"]]
    values = [do(env, a) for a in args[1:]] # obwohl ich list comprehensions verabscheue...

    func = env[func_name] # func = { "parameter": ["n"], "body": ["multiplication", "n", "n"] }
    assert isinstance(func, dict)
    parameters = func["parameter"]
    body = func["body"]

    start_time = time.perf_counter()

    # Lokale Environment vorbereiten
    local_env = env.copy()
    for i in range(len(parameters)):
        local_env[parameters[i]] = values[i] # Example: parameters = ["a", "b"], values = [10, 12] → local_env = {"a": 10, "b": 12, ...}

    # Stack tracing
    parent = TRACE_STACK[-1] if TRACE_STACK else None
    trace_entry = {"name": func_name, "children": [], "time": 0.0}
    if parent:
        parent["children"].append(trace_entry)
    TRACE_STACK.append(trace_entry)

    result = do(local_env, body)

    duration_ms = (time.perf_counter() - start_time) * 1000
    trace_entry["time"] = round(duration_ms, 3)
    TRACE_STACK.pop()

    # Nur Top-Level-Calls im Trace speichern
    if not TRACE_STACK:
        TRACE_LOG.append(trace_entry)

    return result
```

`do_print`: Prints a value to the console. It evaluates the single argument and prints it using Python's built-in `print` function, then returns the value that was printed.
```python
def do_print(env, args):
    """Gebrauch: ["print", ["get", "x"]]"""
    assert len(args) == 1
    val = do(env, args[0])
    print(val)
    return val
```

`print_trace_tree`: Recursively prints the hierarchical trace tree showing all function calls and their execution times. It takes a list of trace entries and an optional prefix string for indentation. For each entry, it prints the function name and execution time with appropriate indentation, then recursively prints all nested child calls with increased indentation to show the call hierarchy.
```python
def print_trace_tree(entries, prefix=""):
    for entry in entries:
        print(f"{prefix}+-- {entry['name']} ({entry['time']}ms)")
        print_trace_tree(entry["children"], prefix + "|   ")
```

`main`: The entry point of the program. It checks command-line arguments to determine if tracing should be enabled by looking for the `--trace` flag. It reads the program file (specified as the last command-line argument), parses it as JSON, executes it with an empty initial environment using the `do` function, and prints the final result. If tracing is enabled, it also prints the complete trace tree showing all function calls and their execution times.
```python
def main():
    global TRACE_ENABLED
    TRACE_ENABLED = "--trace" in sys.argv

    # Determine filename
    filename = sys.argv[-1]

    with open(filename, "r") as reader:
        program = json.load(reader)
    result = do({}, program)
    print(f"=> {result}")

    if TRACE_ENABLED:
        print_trace_tree(TRACE_LOG)
```

`if __name__ == "__main__"`: Standard Python idiom that ensures the `main()` function is only executed when the script is run directly, not when it's imported as a module.
```python
if __name__ == "__main__":
    main()
```

## Disclaimer
We aimed to distribute the workload as evenly as possible, and overall, this was successful. However, the commit count varies 
due to different committing habits. Additionally, Faiaz Islam did all commits to the README files via the GitLab WebApp, resulting 
in a higher number of commits on his part. 
All code was committed exclusively using Git, but for the README file we used the GitLab WebApp to make editing more convenient.



