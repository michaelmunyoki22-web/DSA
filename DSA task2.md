

---

## Part (a): Algorithm for Computing Factorials

To compute $n!$ iteratively using previous factorials in order (from $1!$ up to $n!$), we can maintain an updated running product.

### Pseudocode Algorithm

```text
Algorithm ComputeFactorials(n):
    Input: An integer n >= 0
    Output: Factorials from 0! up to n!

    If n < 0 then
        Display "Error: Factorial is not defined for negative numbers."
        Return
    EndIf

    Declare an array or list 'factorials' of size n + 1
    Set factorials[0] = 1   // Base case: 0! = 1

    Display "0! = 1"

    For i = 1 to n Do:
        Set factorials[i] = factorials[i - 1] * i
        Display i + "! = " + factorials[i - 1] + " x " + i + " = " + factorials[i]
    EndFor

    Return factorials

```

---

## Part (b): Description of Stack Data Structure and Real-Life Applications

### What is a Stack?

A **Stack** is a linear data structure that follows the **LIFO (Last-In, First-Out)** principle. This means the last element added (pushed) onto the stack will be the first one removed (popped).

#### Core Operations:

* **Push**: Adds an element to the top of the stack.
* **Pop**: Removes and returns the top element of the stack.
* **Peek/Top**: Returns the top element without removing it.
* **isEmpty**: Checks whether the stack contains any elements.

---

### Real-Life Applications of Stacks

#### Computer Science Applications:

1. **Function Call Management (Call Stack)**: Keeps track of active subroutines, local variables, and return addresses during nested or recursive function calls.
2. **Undo/Redo Mechanisms**: Text editors and design programs use stacks to record state history so users can revert actions in reverse chronological order.
3. **Expression Evaluation & Conversion**: Used by compilers to convert infix expressions (e.g., $A + B$) to postfix/prefix format and evaluate arithmetic expressions using operator precedence.
4. **Browser Back/Forward Buttons**: Navigated URLs are pushed onto a stack so hitting "Back" pops the current page and restores the previously visited page.

#### Everyday Physical Analogs:

1. **A Stack of Plates**: Placed one on top of another; the top plate placed last is the first one taken off.
2. **Pez Candy Dispenser**: The last candy pushed down into the spring container is the first one released.

---

## Part (c): Python Implementation Using Stacks (Forward and Backward)

Below is a complete Python program that demonstrates factorial calculation using two approaches:

1. **Forward Stack**: Pushes base values first ($1, 2, \dots, n$) and computes sequentially moving forward.
2. **Backward Stack**: Emulates recursion/call stack by pushing $n$ down to $1$, then popping backward to compute the product.

```python
class Stack:
    """Standard Stack implementation using Python list."""
    def __init__(self):
        self.items = []

    def push(self, val):
        self.items.append(val)

    def pop(self):
        if not self.is_empty():
            return self.items.pop()
        raise IndexError("Pop from empty stack")

    def peek(self):
        if not self.is_empty():
            return self.items[-1]
        return None

    def is_empty(self):
        return len(self.items) == 0

    def size(self):
        return len(self.items)


def forward_stack_factorial(n):
    """
    Computes factorials in forward order (1! to n!) using a Stack.
    Pushes 1, 2, ..., n onto stack, then processes from 1 upwards.
    """
    if n < 0:
        return "Factorial not defined for negative numbers."
    if n == 0:
        return 1

    temp_stack = Stack()
    # Push elements in reverse so top of stack gives 1, 2, ..., n when popped
    for i in range(n, 0, -1):
        temp_stack.push(i)

    current_factorial = 1
    print("\n--- Forward Stack Processing ---")
    while not temp_stack.is_empty():
        num = temp_stack.pop()
        prev_fact = current_factorial
        current_factorial *= num
        print(f"{num}! = {prev_fact} x {num} = {current_factorial}")

    return current_factorial


def backward_stack_factorial(n):
    """
    Simulates recursive stack execution (Backward processing).
    Pushes n down to 1, then pops to multiply backward: 1 * 2 * ... * n.
    """
    if n < 0:
        return "Factorial not defined for negative numbers."
    if n == 0:
        return 1

    call_stack = Stack()
    
    # Unwinding phase: push n down to 1
    current = n
    while current >= 1:
        call_stack.push(current)
        current -= 1

    print("\n--- Backward Stack Processing (Call Stack Simulation) ---")
    result = 1
    while not call_stack.is_empty():
        val = call_stack.pop()
        prev_result = result
        result *= val
        print(f"Popped {val} -> Product = {prev_result} x {val} = {result}")

    return result


# Demonstration / Driver Code
if __name__ == "__main__":
    number = 5
    print(f"=== Computing Factorial for n = {number} ===")

    fact_forward = forward_stack_factorial(number)
    print(f"\nFinal Result (Forward Stack): {number}! = {fact_forward}")

    fact_backward = backward_stack_factorial(number)
    print(f"\nFinal Result (Backward Stack): {number}! = {fact_backward}")

```