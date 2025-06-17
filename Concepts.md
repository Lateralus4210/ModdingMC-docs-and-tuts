# Concepts
### Data Types
    - Int, float, string, etc.

### Object Oriented Programming
    - Classes
    - Functions/Methods
    - Instantiation
    - State

### Objectives
    - Make a class in Java (or Python)
    - Give it a function
    - Define state (define parameters (variables) and give them values)
    - Instantiate it (create a new object)

>Disclaimer: these ideas are pretty abstract even once you understand them, and will make more and more sense the more exposure you have to Object-Oriented Programming (OOP) languages.


###### misc concepts and notes:
# 🔧 Essentials for Junior Programmers

## Encapsulation (Private/Public)

- Introduce access modifiers (`private`, `public`, `protected` in Java).
- Explain **why** state is often protected and accessed via getter/setter methods.
- Helps maintain control and prevent unexpected changes to internal state.

## Constructors

- Special methods for setting the initial state of an object.
- Java: `public ClassName(...)`, Python: `def __init__(...)`.
- Emphasize their role in proper object creation.

## The `self` (Python) / `this` (Java) Keyword

- Crucial for understanding object context.
- Refers to the *current instance* of the class.
- Used to access attributes and methods inside class definitions.

## Simple Inheritance

- One class extends another: `class Child extends Parent` (Java), `class Child(Parent)` (Python).
- Shows reuse of functionality.
- Introduce method overriding conceptually.

## IDE & Debugging Basics

- Navigating an IDE (e.g., IntelliJ, VS Code, PyCharm).
- Setting breakpoints, running in debug mode, inspecting variables.
- Reading and interpreting error messages.
- Builds confidence and self-sufficiency.

## Simple Testing / Output

- Use `print()` (Python) or `System.out.println()` (Java) to check variable values and behavior.
- Introduce the mindset of testing code as it's written.
- Optional: brief mention of unit testing (JUnit, `unittest`).

---

# 🧠 Mental Models

- **Objects** are "things" that contain:
  - **Data** (state/attributes)
  - **Actions** (methods/functions)

- A **class** is a **blueprint**; an **object** is a **thing built from that blueprint**.

- Every time you use `new` (Java) or call a class like a function (Python), you're making a new object.

---

> The more you build, the more these concepts will click. Keep coding!
