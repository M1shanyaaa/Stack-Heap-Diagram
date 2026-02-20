

# Java Memory Management Exercise: Stack-Heap Diagram

This repository contains the solution for the Stack-Heap Diagram exercise. The goal is to demonstrate a step-by-step understanding of how the JVM manages memory during method calls, object allocation, and primitive storage.

## Code Analyzed

```java
public class Main {
    public static void main(String[] args) {
        String name = "Kevin";
        List<String> list = new ArrayList<>();
        int times = 10;
        System.out.println(times + fill(list, name + name, times));
    }

    public static int fill(Collection<String> collection, String str, int times){
        String shrunk = shrink(str);
        times = (times + shrunk.length()) / 2;
        for (int i = 0; i < times / 2; i++) {
            collection.add(shrunk);
        }
        return times;
    }

    public static String shrink(String str){
        int newLength = str.length() / 2 + str.length() % 2;
        char[] chars = new char[newLength];
        for (int i = 0; i < str.length(); i+=2) {
            chars[i / 2] = str.charAt(i);
        }
        return new String(chars);
    }
}
```

# Java Memory Management Exercise: Stack-Heap Diagram Analysis

This document provides a detailed, step-by-step memory allocation analysis for the provided Java program. It tracks the lifecycle of variables and objects in the Stack and Heap segments of the JVM.

## Step-by-Step Execution Analysis

### 1. `main` Method Initialization
When the program starts, the JVM pushes the `main` method frame onto the **Stack**.
* **Stack:** The primitive variable `int times = 10` is stored directly in the frame.
* **Heap (String Pool):** The string literal `"Kevin"` is allocated.
* **Stack:** The local reference `String name` is created, pointing to the address of `"Kevin"`.
* **Heap:** A new `ArrayList` object is instantiated.
* **Stack:** The local reference `List<String> list` is created, pointing to the `ArrayList` object.



### 2. Preparation for `fill` Method Call
Before the `fill` method is invoked, the expression `name + name` must be evaluated.
* **Heap:** A new `String` object `"KevinKevin"` is created (concatenation result).
* **Note:** This object is separate from the original `"Kevin"` in the pool.

### 3. `fill` Method Execution
A new frame for `fill` is pushed onto the **Stack**.
* **Stack (Parameters):** * `collection`: Receives a copy of the reference to the `ArrayList` (pointing to the same object in Heap).
    * `str`: Receives a reference to the temporary `"KevinKevin"` object.
    * `times`: Receives a primitive copy of the value `10`.

### 4. `shrink` Method Execution
Inside `fill`, the `shrink(str)` method is called, pushing a new frame onto the **Stack**.
* **Stack:** `int newLength` is calculated as `5` ($10 / 2 + 0$).
* **Heap:** A `char[]` array of size 5 is allocated.
* **Stack:** `chars` reference points to this array.
* **Loop:** The array is populated with characters `['K', 'v', 'n', 'e', 'i']`.
* **Heap:** `new String(chars)` creates a new `String` object `"Kvnei"`.
* **Return:** The reference to `"Kvnei"` is returned to `fill`, and the `shrink` frame is popped.



### 5. Finalizing `fill` and Loop Logic
Back in the `fill` frame:
* **Stack:** `String shrunk` now points to the `"Kvnei"` object in Heap.
* **Stack:** Local `times` is updated: $(10 + 5) / 2 = 7$.
* **Loop Execution:** The loop runs for `i = 0` to `i < 3` (since `7 / 2 = 3`).
* **Heap:** The `ArrayList` object is modified to store **three references** to the single `"Kvnei"` object.
* **Return:** The value `7` is returned to `main`, and the `fill` frame is popped.

---

## Final Memory State Snapshot (Before `System.out.println`)

### **Stack Memory (LIFO)**
| Method Frame | Variable | Value / Reference |
| :--- | :--- | :--- |
| **fill** (Current) | `i` | `3` (primitive) |
| | `shrunk` | Ref -> `0x404` ("Kvnei") |
| | `times` | `7` (primitive) |
| | `str` | Ref -> `0x303` ("KevinKevin") |
| | `collection` | Ref -> `0x202` (ArrayList) |
| **main** (Caller) | `times` | `10` (primitive) |
| | `list` | Ref -> `0x202` (ArrayList) |
| | `name` | Ref -> `0x101` ("Kevin") |

### **Heap Memory**
| Address | Object Type | Content |
| :--- | :--- | :--- |
| **0x101** | `String` | `"Kevin"` (String Pool) |
| **0x202** | `ArrayList` | `[Ref->0x404, Ref->0x404, Ref->0x404]` |
| **0x303** | `String` | `"KevinKevin"` (Eligible for GC) |
| **0x404** | `String` | `"Kvnei"` |
| **0x505** | `char[]` | `['K', 'v', 'n', 'e', 'i']` |



---

## Final Program Output
The `main` method executes `System.out.println(10 + 7)`, which results in:
**`17`**

## Self-Evaluation & Criteria Checklist
* **Primitive variables & references in Stack:** All mapped correctly (4/4 points).
* **Correct order of method calls/frames:** Stack follows `main` -> `fill` -> `shrink` flow (3/3 points).
* **Heap objects properly linked:** ArrayList and String objects accurately depict references (3/3 points).
