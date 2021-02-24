---
description: A brief introduction to Python unittest module
---

# Unit Testing

It's a common practice to test the output/behaviour of certain pieces of code \(units\) to verify that execute what we expect. That is particulary useful when many developers program on the same repository at the same time, in order to immediately be aware if a modification or a feature breaks some other functionalities introducing bugs.

Unit tests are particulary useful to test non-void functions that accept an input and return a known output, but more complex tests can also be performed with different paradigms.  
In this section, a basic guide about how to perform unit tests on Python is provided.

### Unit test with Python

Python provides the `unittest` module to perform such functionality with easy pain. Test **must** be written in a file name `test_[name].py` in order to be easily found during continus testing pipelines. Here the bare-minimum skeleton is shown

`test_example.py`

```python
# test_example.py

import unittest

class MyTestCase(unittest.TestCase):
    def test_first_example(self):
        # Perform some operations
        self.assert_(expression_1)
        self.assert_(expression_2)
        
    def test_first_example(self):
        # Perform some operations
        self.assert_(expression_3)

```

As shown in the example before, within a TestCase we can write multiple test defining a function per them of them.

### Create a Unit Test

Let's see a real example:  
Suppose we have a math library `math.py`with some functions that we want to test

```python
# math.py

def add_subtract(a, b, operation="add"):
    
    if operation == "add":
        return a + b
        
    elif operation == "subtract":
        return a - b
    
    else:
        raise ValueError("operation can be 'add' or 'subtract'")

    
def multiply_divide(a, b, operation="multiply"):
    
    if operation == "multiply":
        return a * b
        
    elif operation == "divide":
        return a / b * 2            # ERROR!
        
    else:
        raise ValueError("operation can be 'multiply' or 'divide'")
```

We can create a test file within a folder `tests` which contain also a `__init__.py` \(**important!**\). Our project will look like

```text
- my-project/
    |_ math.py
    |_ tests/
        |_ __init__.py
        |_ test_math.py
```

Let's know create the `test_math.py` file which contains the tests.

```python
# test/test_math.py

import unittest
from math import add_subtract
from math import multiply_divide


class MathLibraryTest(unittest.TestCase):
    
    def test_add_subtract(self):
        a = 10
        b = 5
        
        x = add_subtract(a, b, "add")
        y = add_subtract(a, b, "subtract")
        
        self.assertEqual(x, 15)
        self.assertEqual(y, 5)
        
    
    def test_multiply_divide(self):
        
        a = 2
        b = 8
        
        x = multiply_divide(a, b, "multiply")
        y = multiply_divide(a, b, "divide")
        
        self.assertGreater(x, 1)
        self.assertLess(y, 1)
        
        
    def test_multiply_divide_2(self):
        a = 2
        b = 4
        
        x = multiply_divide(a, b, "multiply")
        y = multiply_divide(a, b, "divide")
        
        self.assertEqual(x, 8)
        self.assertEqual(y, 0.5)
```

### Perform the Test

To perform the test, _if we respected the naming convention_, it is easy as run the following command

```text
python -m unittest discover
```

Note the keyword `discover`: the unittest module will search for all the files named as `test_[name].py`**.**

In out example above, the output of the previous command will be

```text
~ python -m unittest discover

======================================================================
FAIL: test_multiply_divide_2 (tests.test_math.MathLibraryTest)
----------------------------------------------------------------------
Traceback (most recent call last):
  File "/home/user/my-project/tests/test_math.py", line 39, in test_multiply_divide_2
    self.assertEqual(y, 0.5)
AssertionError: 1.0 != 0.5

----------------------------------------------------------------------
Ran 3 tests in 0.001s

FAILED (failures=1)
```

telling us that 2 out of 3 tests passed and one failed, allowing us to easily find the bug.

