# Object-Oriented Programming in C++ -- Assignment 2

**Course Outcome:** CO2  
**Name:** Lakshmi Narayanan NTN

**Register Number:** 192524353

**Language:** C++17

---

## About This Assignment

This repository contains the solutions and explanations for Assignment 2 of Object-Oriented Programming in C++. The assignment covers important C++ concepts such as parameter passing, references, pointers, function overloading, classes, arrays of objects, static members, friend functions, and performance analysis.

Each question includes the relevant C++ implementation, expected output, and an explanation of the concepts used.

> **Note:** The predicted outputs and performance expectations should be checked by compiling and running the programs in a C++17-compatible compiler. Actual benchmark timings depend on the computer and compiler settings.

---

## Table of Contents

- [Q1 -- Call by Value, Reference and Pointer](#q1----call-by-value-reference-and-pointer)
- [Q2 -- Return by Reference and Its Dangers](#q2----return-by-reference-and-its-dangers)
- [Q3 -- Default Arguments and Function Overloading](#q3----default-arguments-and-function-overloading)
- [Q4 -- Class with an Array Inside It](#q4----class-with-an-array-inside-it)
- [Q5 -- Static Members and Arrays of Objects](#q5----static-members-and-arrays-of-objects)
- [Q6 -- Friend Functions and Inline Functions](#q6----friend-functions-and-inline-functions)
- [Q7 -- Diagnose, Redesign and Validate](#q7----diagnose-redesign-and-validate)

---

## Q1 -- Call by Value, Reference and Pointer

### 1.1 Three Implementations

This section compares three ways of passing arguments to functions in C++: call by value, call by reference, and call by pointer.

```
#include <iostream>

void concessionValue(long long fee, int percent) {
    fee -= fee * percent / 100;
}

void concessionRef(long long& fee, int percent) {
    fee -= fee * percent / 100;
}

void concessionPtr(long long* fee, int percent) {
    *fee -= *fee * percent / 100;
}

int main() {
    long long f = 45000;

    concessionValue(f, 10);
    std::cout << "value : " << f << '\n';

    concessionRef(f, 10);
    std::cout << "ref   : " << f << '\n';

    f = 45000;
    concessionPtr(&f, 10);
    std::cout << "pointer: " << f << '\n';
}
```

**Predicted stored fee:**

- Call by value: `45000` -- the original value remains unchanged.
- Call by reference: `40500` -- the original value is modified.
- Call by pointer: `40500` -- the original value is modified through the pointer.

### 1.2 Explanation

- **Call by value:** The function receives a copy of the argument. Changes made inside the function affect only that copy.
- **Call by reference:** The parameter acts as an alias for the caller's variable. Changes made through the reference affect the original variable.
- **Call by pointer:** The function receives an address. Using the dereference operator (`*`) allows it to modify the original variable.
- References and pointers can modify the caller's variable, while ordinary value parameters cannot.
- `const T&` is useful when a function needs to read a large object without copying it or modifying it.

### 1.3 Design Decision

For applying multiple concessions, returning the updated fee is a useful design:

```
long long applyConcession(long long fee, int percent) {
    return fee - fee * percent / 100;
}
```

Example:

```
fee = applyConcession(applyConcession(fee, 10), 15);
```

This approach avoids modifying the original variable inside the function. It is easy to test and allows function calls to be chained.

The concession rule should also specify whether percentages are applied successively or added together. For example, successive concessions change `45000` to `40500` and then to `34425`.

### 1.4 Predict the Output

```
#include <iostream>

void change(int a, int& b, int* c) {
    a += 10;
    b += 10;
    *c += 10;
}

int& bigger(int& p, int& q) {
    return p > q ? p : q;
}

void show(int x, int y = 5, int z = 10) {
    std::cout << x + y + z << ' ';
}

int main() {
    int a = 1, b = 2, c = 3;

    change(a, b, &c);
    std::cout << a << ' ' << b << ' ' << c << '\n';

    bigger(a, c) = 100;
    std::cout << a << ' ' << c << '\n';

    show(1);
    show(1, 2);
    show(1, 2, 3);
    std::cout << '\n';
}
```

**Predicted Output:**

```
1 12 13
1 100
16 13 6
```

**Explanation:**

- `a` is passed by value, so the original `a` remains `1`.
- `b` is passed by reference, so it becomes `12`.
- `c` is modified through a pointer, so it becomes `13`.
- `bigger(a, c)` returns a reference to the larger value. Assigning `100` changes `c`.
- `show(1)` uses both default arguments and prints `16`.
- `show(1, 2)` uses the default value of `z` and prints `13`.
- `show(1, 2, 3)` uses all supplied arguments and prints `6`.

### 1.5 Complete the Code

```
void applyConcession(long long& fee, int percent) {
    fee -= fee * percent / 100;
}
```

**Expected values:**

```
after call by value : 45000
after call by reference: 40500
```

### 1.6 Time and Space Comparison

| Passing style | Time | Copy cost |
|---|---|---|
| By value | O(1) for a fixed-size type | Copies the value |
| By reference | O(1) | No object copy |
| By const reference | O(1) | No object copy; read-only access |

For a `long long`, copying is inexpensive. For a large object such as a `Student` record, using a reference or const reference avoids copying the complete object.

---

## Q2 -- Return by Reference and Its Dangers

### 2.1 Implementation

A function can return a reference to an existing array element. This allows the caller to modify that element directly.

```
#include <iostream>

int& highest(int marks[], int n) {
    int idx = 0;

    for (int i = 1; i < n; ++i) {
        if (marks[i] > marks[idx]) {
            idx = i;
        }
    }

    return marks[idx];
}

const int& highestConst(const int marks[], int n) {
    int idx = 0;

    for (int i = 1; i < n; ++i) {
        if (marks[i] > marks[idx]) {
            idx = i;
        }
    }

    return marks[idx];
}

int highestCopy(const int marks[], int n) {
    return highestConst(marks, n);
}
```

### 2.2 Returning a Reference to a Local Variable

```
int& dangling() {
    int local = 42;
    return local;
}
```

This is incorrect because `local` is destroyed when the function returns. The returned reference no longer refers to a valid object. Using it results in undefined behaviour.

A compiler may produce a warning such as:

```
warning: reference to local variable 'local' returned
```

Compile with warnings enabled:

```
g++ -std=c++17 -Wall -Wextra q2.cpp
```

### 2.3 Return by Reference, Const Reference and Copy

| Return type | Can the caller modify the result? | Effect on original array |
|---|---|---|
| `int&` | Yes | Can modify the array |
| `const int&` | No | Cannot modify through the reference |
| `int` | Modifies only the returned copy | Original array remains unchanged |

### 2.4 Vector Reallocation and Reference Safety

```
std::vector<int> v = {56, 78, 91};
int& top = highestVec(v);

v.push_back(99);
top += 2;
```

A vector may allocate a new memory block when its capacity is exceeded. If reallocation occurs, previously obtained references, pointers, and iterators become invalid.

A safer approach is to store an index and access the element again after the vector changes.

### 2.5 Alternative: Index or Pointer

```
int highestIndex(const int a[], int n);
int* highestPtr(int a[], int n);
```

| Return type | Search time | Space | Behaviour after vector growth |
|---|---|---|---|
| `int&` | O(n) | O(1) | May become invalid |
| `int*` | O(n) | O(1) | May become invalid |
| `int` index | O(n) | O(1) | Can be reused if the element is still present at that index |

**Conclusion:** An index is often safer when a vector may grow, provided the element is not erased or reordered.

---

## Q3 -- Default Arguments and Function Overloading

### 3.1 Understanding the Ambiguity

Consider these function declarations:

```
double charge(double weight);
double charge(double weight, bool express);
double charge(double weight, bool express = false, bool insured = false);
```

These declarations can make calls ambiguous:

- `charge(2.0)` can match the first and third functions.
- `charge(2.0, true)` can match the second and third functions.

The compiler cannot select one unique best function for these calls.

### 3.2 Redesigned Implementation

A single function with default arguments can handle the optional express and insurance charges. Separate overloads can handle grams and a `Parcel` object.

```
#include <iostream>

struct Parcel {
    double kg;
    bool express;
    bool insured;
};

const double RATE_PER_KG = 50.0;
const double EXPRESS_FEE = 30.0;
const double INSURANCE_FEE = 20.0;

double charge(double kg, bool express = false, bool insured = false) {
    double c = kg * RATE_PER_KG;

    if (express) c += EXPRESS_FEE;
    if (insured) c += INSURANCE_FEE;

    return c;
}

double charge(int grams) {
    return charge(grams / 1000.0);
}

double charge(const Parcel& p) {
    return charge(p.kg, p.express, p.insured);
}

int main() {
    Parcel p{2.0, true, false};

    std::cout << charge(2) << ' '
              << charge(2.0f) << ' '
              << charge(2.0) << ' '
              << charge(2.0, true) << ' '
              << charge(2.0, true, true) << ' '
              << charge(p) << '\n';
}
```

### 3.3 Predicted Selections and Output

| Function call | Selected overload | Result |
|---|---|---|
| `charge(2)` | `charge(int grams)` | 0.1 |
| `charge(2.0f)` | Double overload | 100 |
| `charge(2.0)` | Double overload | 100 |
| `charge(2.0, true)` | Double overload | 130 |
| `charge(2.0, true, true)` | Double overload | 150 |
| `charge(p)` | `charge(const Parcel&)` | 130 |

**Predicted Output:**

```
0.1 100 100 130 150 130
```

### 3.4 Explanation

The compiler selects an overloaded function by considering the number of arguments and the conversions required.

The general preference is:

1. Exact match
2. Promotion
3. Standard conversion
4. User-defined conversion

If there is no unique best match, the call is ambiguous.

Default arguments must be supplied from right to left. Once a parameter has a default value, all parameters to its right must also have default values.

### 3.5 Alternative Design

A `Parcel` structure can group related options together:

```
struct Parcel {
    double kg = 0;
    bool express = false;
    bool insured = false;
};

double charge(const Parcel& p);
```

This design is easier to extend when new options are added. For example, a new `fragile` field can be added without making function calls harder to read.

---

## Q4 -- Class with an Array Inside It

### 4.1 Savings Account Implementation

The `SavingsAccount` class stores account details, the current balance, and a fixed-size transaction history.

```
#include <iostream>

class SavingsAccount {
    static const int MAX_TX = 10;

    long accNo;
    double balance;
    double tx[MAX_TX];
    int count;
    int start;

    void record(double signedAmount);

public:
    SavingsAccount(long no, double opening = 0.0)
        : accNo(no), balance(opening), count(0), start(0) {}

    double getBalance() const {
        return balance;
    }

    void deposit(double amt);
    bool withdraw(double amt);
    void miniStatement() const;
};

void SavingsAccount::record(double v) {
    if (count < MAX_TX) {
        tx[(start + count) % MAX_TX] = v;
        ++count;
    } else {
        tx[start] = v;
        start = (start + 1) % MAX_TX;
    }
}

void SavingsAccount::deposit(double amt) {
    if (amt <= 0) return;

    balance += amt;
    record(amt);
}

bool SavingsAccount::withdraw(double amt) {
    if (amt <= 0 || amt > balance) return false;

    balance -= amt;
    record(-amt);
    return true;
}

void SavingsAccount::miniStatement() const {
    std::cout << "Account " << accNo << " (last " << count << ")\n";

    for (int i = 0; i < count; ++i) {
        double v = tx[(start + i) % MAX_TX];
        std::cout << (v >= 0 ? " CR " : " DR ")
                  << (v >= 0 ? v : -v) << '\n';
    }

    std::cout << " Balance: " << balance << '\n';
}

int main() {
    SavingsAccount a(1001, 500);

    for (int i = 1; i <= 12; ++i) {
        (i % 3 == 0) ? a.withdraw(10 * i) : a.deposit(100 + i);
    }

    a.miniStatement();
}
```

### 4.2 Why the Array Is Private

The transaction array is private to protect the class data.

The following conditions must remain valid:

- `0 <= count <= MAX_TX`
- The balance must match the opening balance plus the sum of recorded transactions.
- Transactions must be stored and displayed in the correct order.

If the array were public, outside code could modify transaction history without updating the balance. Keeping it private ensures that deposits and withdrawals control the changes.

### 4.3 Shifting Array vs Circular Buffer

**Shifting approach:**

When the array is full, existing elements are moved one position to make room for the new transaction. This takes O(k) time.

**Circular buffer approach:**

The oldest transaction is overwritten, and the `start` index is advanced. No elements need to be shifted, so insertion takes O(1) time.

| Design | Insertion time when full | Space |
|---|---|---|
| Shifting fixed array | O(k) | k elements and count |
| Circular buffer | O(1) | k elements, count and start |
| `std::deque` | O(1) amortised | Dynamic allocation and additional overhead |

### 4.4 Design Decision

For a capacity of 50 transactions, the circular buffer is suitable. Changing `MAX_TX` from 10 to 50 is sufficient because the implementation uses modulo arithmetic with `MAX_TX`.

The circular buffer provides constant-time insertion and avoids moving existing elements.

---

## Q5 -- Static Members and Arrays of Objects

### 5.1 Design Explanation

In a bus ticket system, the issued ticket count belongs to the entire class, while the passenger name and seat number belong to individual ticket objects.

- The issued count should be a static data member.
- The capacity can be declared as a static constant.
- Seat number and passenger name should be non-static members.
- A static member function can access static members directly.
- A static member function does not have a `this` pointer.
- An array of objects calls the constructor for each element.

### 5.2 Complete the Code

```
explicit BusTicket(const std::string& n) : seat(0), name(n) {
    if (issued < CAPACITY) {
        ++issued;
        seat = issued;
    }
}

static int count() {
    return issued;
}
```

The static data member must be defined once outside the class:

```
int BusTicket::issued = 0;
```

### 5.3 Predicted Output

```
101 103 104 4
```

**Explanation:**

- `next` is initialized to 100.
- Creating an array of three objects invokes the constructor three times.
- The generated IDs are 101, 102 and 103.
- The fourth object receives ID 104.
- `issued()` returns `next - 100`, which is 4.

### 5.4 Expected Ticket Output

```
Asha: Seat 1
Bala: Seat 2
Chitra: Seat 3
Dev: Waiting
Issued: 3
```

### 5.5 Static Member vs Global Variable vs TicketOffice

| Design | Encapsulation | Supports multiple buses? |
|---|---|---|
| Static member | Controlled by the class | No, one counter per class |
| Global variable | Anyone can change it | No |
| `TicketOffice` object | Counter is private to the office | Yes, one office per bus |

A `TicketOffice` object is useful when several buses need separate ticket counters. It also makes testing easier because each test can create a fresh office.

---

## Q6 -- Friend Functions and Inline Functions

### 6.1 Complete the Code

The following functions calculate the difference between two times and check whether one time is earlier than another.

```
int minutesBetween(const Time& a, const Time& b) {
    int ma = a.h * 60 + a.m;
    int mb = b.h * 60 + b.m;

    return ma > mb ? ma - mb : mb - ma;
}

bool earlier(const Time& a, const Time& b) {
    return a.h < b.h || (a.h == b.h && a.m < b.m);
}
```

**Expected Output:**

```
205 false
```

**Explanation:**

- 9:45 is equal to 585 minutes.
- 13:10 is equal to 790 minutes.
- The difference is 205 minutes.
- `earlier(arrival, departure)` checks whether 13:10 is earlier than 9:45, so the result is `false`.

### 6.2 Correctness Tests

| Time A | Time B | `minutesBetween` | `earlier(A, B)` |
|---|---|---:|---|
| 9:45 | 13:10 | 205 | true |
| 13:10 | 9:45 | 205 | false |
| 8:30 | 8:30 | 0 | false |
| 11:50 | 12:10 | 20 | true |
| 12:10 | 11:50 | 20 | false |

### 6.3 Why Use a Friend Function?

- A friend function can access private and protected members of the class that declares it as a friend.
- It is not a member function and does not have a `this` pointer.
- It is called like a normal function, for example `minutesBetween(a, b)`.
- It is useful when an operation treats two objects equally.

### 6.4 Inline Functions

The `inline` keyword allows the compiler to consider replacing a function call with the function body. However, it does not force the compiler to perform inlining.

For small functions called frequently, inlining may reduce function-call overhead and allow further compiler optimizations.

### 6.5 Benchmark

The assignment compares inline and non-inline versions using `std::chrono::steady_clock`.

Compile using:

```
g++ -std=c++17 -O0 bench.cpp -o b0
g++ -std=c++17 -O2 bench.cpp -o b2
```

Record the actual results from the system used:

| Optimization level | Inline time (ms) | Non-inline time (ms) |
|---|---|---|
| `-O0` | To be measured | To be measured |
| `-O2` | To be measured | To be measured |

The results depend on the compiler and hardware. The `inline` keyword alone does not guarantee that a function will be inlined.

### 6.6 Deployment Decision

Member functions such as `minutesTo()` and `isEarlier()` keep the logic inside the `Time` class and maintain encapsulation. A friend function should be used when a non-member function genuinely needs access to private data.

---

## Q7 -- Diagnose, Redesign and Validate

### 7.1 Selected Program

The selected program is the `SavingsAccount` mini-statement from Q4. The original transaction history uses a shifting array. The requirement is to increase the transaction capacity from 10 to 50 and potentially more.

### 7.2 Original Shifting Version

```
void recordShift(double v) {
    if (count == MAX_TX) {
        for (int i = 1; i < MAX_TX; ++i) {
            tx[i - 1] = tx[i];
        }

        tx[MAX_TX - 1] = v;
    } else {
        tx[count++] = v;
    }
}
```

### 7.3 A. Diagnose the Limitation

When the array is full, every new transaction requires moving `k - 1` elements.

Therefore, the insertion operation takes O(k) time. As the capacity increases, the time required for insertion also increases.

The reason is that the oldest transaction is always kept at index 0, so the remaining elements must be shifted to make room.

### 7.4 B. Design Objective

The redesigned program should satisfy these requirements:

- Adding a transaction should take O(1) time, independent of the capacity.
- At capacity `k = 1000`, the circular version should aim to be at least 10 times faster than the shifting version for 1,000,000 transactions.
- The mini-statement output must remain identical.
- Additional memory must not exceed one integer.

### 7.5 C. Revised Solution

The circular buffer uses a `start` variable to identify the oldest transaction.

- When space is available, insert at `(start + count) % k`.
- When the array is full, overwrite `tx[start]`.
- Advance the start index using `(start + 1) % k`.
- Display the transactions using `tx[(start + i) % k]`.

This approach avoids shifting elements and keeps the output order unchanged.

**Circular Buffer Logic:**

```
class SavingsAccount {
    // tx[K], count, start

    void record(double v) {
        if (count < K) {
            tx[(start + count) % K] = v;
            count++;
        } else {
            tx[start] = v;
            start = (start + 1) % K;
        }
    }
};
```

**Advantages:**

- Insertion takes O(1) time.
- Only one additional integer is required.
- The fixed-size array is retained.
- The transaction display order remains unchanged.

**Trade-offs:**

- Modulo arithmetic makes the code slightly harder to understand.
- The array is not stored in chronological order in memory.
- Functions that require chronological data must access it through the circular indexing logic.

### 7.6 D. Prediction

The shifting version is expected to take more time as the capacity increases because it moves existing elements.

The circular buffer is expected to maintain nearly constant insertion time as the capacity changes.

These are predictions and must be compared with actual benchmark measurements.

### 7.7 E. Implement and Validate

Use the same test driver for both versions:

- Insert 1,000,000 transactions.
- Test capacities of 10, 50, 200 and 1000.
- Measure execution time using `std::chrono::steady_clock`.
- Compare the final mini-statement outputs.
- Build with `-O2`.
- Repeat each measurement 3 to 5 times and calculate the median.

**Benchmark Results:**

| Capacity (k) | Shifting time (ms) | Circular time (ms) | Speed-up |
|---:|---|---|---|
| 10 | To be measured | To be measured | To be calculated |
| 50 | To be measured | To be measured | To be calculated |
| 200 | To be measured | To be measured | To be calculated |
| 1000 | To be measured | To be measured | To be calculated |

### 7.8 F. Defend the Final Design

The circular buffer is preferable when the transaction capacity is large because it avoids repeated element shifting.

**Additional costs:**

- One extra integer (`start`).
- A modulo operation during indexing.
- A less straightforward memory layout.

**When shifting may still be preferable:**

For very small capacities, such as 10, shifting may be simple enough and its performance cost may be negligible. It also keeps the array in chronological order.

**Conclusion:**

The final design should be evaluated using the measured benchmark results. If the expected speed-up is not achieved, the difference should be explained using the compiler optimization, hardware, and benchmark behaviour.

---

## Conclusion

This assignment demonstrates important Object-Oriented Programming concepts in C++. It compares different parameter-passing methods, explains reference safety, applies function overloading and default arguments, and uses classes to manage arrays and objects.

It also explores static members, friend functions, inline functions, and performance optimization using a circular buffer. The assignment highlights the importance of selecting suitable data structures and validating program performance through actual execution.

---

## Technologies Used

- **Programming Language:** C++
- **Standard:** C++17
- **Compiler:** GCC / G++
- **Concepts:** Functions, References, Pointers, Classes, Arrays, Static Members, Friend Functions, Inline Functions, Vectors, Circular Buffers
- **Benchmarking:** `std::chrono`

---

## Repository Note

The benchmark timing tables are intentionally left for actual measurements. Replace the placeholder values with the results obtained after compiling and running the programs on your system.
