---
layout: page
title: CLike - A Programming Language
description: An interpreter for a C-like language built from scratch in Python, demonstrating the complete pipeline from text to execution.
# img: assets/img/12.jpg
importance: 4
category: personal
related_publications: false
---

## Overview

We use programming languages daily, but how do they *actually* work? What is the magic that happens between typing `int x = 10;` and a value being stored in memory? My challenge was to deconstruct this "black box" by building one from first principles.

The goal of CLIKE was not to create the next production language, but to build a complete, educational interpreter from the ground up. First the designing language's syntax and *I was too lazy to come up with a new syntax so I just stole the syntax from C (I think now you can guess where I got the name of this language), all hail the great lord [Dennis Ritchie](https://en.wikipedia.org/wiki/Dennis_Ritchie)*, then comes the architecture, and runtime model. The project's central purpose was to master the classic multi-pass interpreter pipeline, taking raw source code text and turning it into a running program.

## Research

Building an interpreter requires a deep understanding of compiler theory. My research centered on key questions that shaped the system's design, outlining the "journey" of the source code from text to execution.

1. **How do we convert raw text into meaningful units?**  
   This involves **Lexical Analysis**. The solution is a **Lexer (Tokenizer)** that scans the input and produces tokens (e.g., `KEYWORD:int`, `IDENTIFIER:x`, `OPERATOR:=`). Key considerations included handling multi-character operators (e.g., `==` vs. `=`) and ignoring whitespace or comments.

2. **How do we ensure these units form valid structures?**  
   This is **Parsing (Syntax Analysis)**. I selected a **Recursive Descent Parser**, where grammar rules (e.g., for statements or expressions) are implemented as recursive functions. The output is an **Abstract Syntax Tree (AST)**, a tree-based representation ideal for further analysis.

3. **How do we verify the structures are logically sound?**  
   **Semantic Analysis** addresses this, catching errors like type mismatches (e.g., assigning a string to an integer). Using a **Scoped Symbol Table**—a stack of dictionaries—I tracked variables and functions across scopes to detect undeclared identifiers or argument mismatches.

4. **How do we execute the validated program?**  
   **Interpretation** handles runtime. I designed a model with a **Call Stack** for function management and **Activation Records** for local variables, enabling recursion and scoped execution.

## Implementation

CLIKE is a multi-pass interpreter written in Python, processing `.clike` files through four interconnected components:

- **`Lexer`:** Iterates character-by-character, using a `peek()` method to differentiate tokens. It skips irrelevant content like comments (`//`) and supports line/column tracking for error reporting.

- **`Parser`:** Constructs the AST from tokens via recursive descent. The `eat()` function enforces grammar by consuming expected tokens. A notable feature is `#include` handling: it recursively parses included files but only imports function declarations, preventing cycles and enabling modular code.

- **`SemanticAnalyzer`:** Traverses the AST using the Visitor pattern. The `ScopedSymbolTable` manages scopes—pushing a new table for functions and popping on exit. It flags errors like undeclared variables (`ID_NOT_FOUND`) or parameter mismatches.

- **`Interpreter`:** Executes the AST as another Visitor. The `CallStack` (a list of `ActivationRecord` objects) handles function calls: pushing a new record on entry, populating parameters, and popping on return. `ReturnException` simplifies value propagation from `return` statements.

Debugging flags (`--debug`, `--scope`, `--stack`) provide insights into parsing, symbols, and runtime states.

## Results

The interpreter reliably executes CLIKE programs, supporting variables, arrays, control flow, recursion, and imports. It passes a suite of tests covering advanced features, edge cases, and robustness.

- **Success Metrics:** All examples run correctly, including recursive factorial computation and array operations. The pipeline's modularity allowed independent testing, ensuring reliability.

- **Key Lesson 1: Modularity:** Separating concerns (lexer from parser, semantics from execution) simplified development and debugging.

- **Key Lesson 2: Data Structures:** The AST (tree), Symbol Table (scoped maps), and Call Stack (runtime stack) were pivotal, reinforcing their role in language implementation.

- **Future Work:** Evolve into a compiler by generating bytecode for a virtual machine. Enhance error messages and add features like escape sequences in literals.

## Reflection

- **Core Technology: Python**  
  Python's expressive syntax and built-ins (e.g., lists for stacks, dictionaries for records) enabled rapid prototyping and clear code, letting me prioritize theory over low-level details.

This project deepened my understanding of language design, transforming abstract concepts into tangible systems. It honed my skills in algorithms, data structures, and software architecture—essential for any developer. Languages are no longer "magic"; they're engineered marvels with purposeful design choices.

> View the CLIKE source code and documentation on [GitHub](https://github.com/carb0ned0/C-Like).