# Mutt Programming Language

</img src="/assets/img/logo.png" width="1rem">

Why select just one when you can cross-breed multiple languages? Mutt is a metamorphic programming language that inherits from a blend of Python, yaml, JavaScript, and other languages. However unlike any one of these, Mutt is not confined to a single cage. He gets to switch between languages on demand. Mutt is able to enjoy this freedom because he gives his master (you) direct control of the interpretter and since he performs just-in-time lexing, parsing, semantic analysis, and execution, that means you can:

- specify custom lexers and parsers
- add your own (not just context-free) patterns to the grammar
- directly manipulate concrete syntax trees
- directly manipulate abstract syntax trees (yes, you can have more than one AST per program)

Just think about the mutts you can breed and tricks you can teach with this power. Mutt is excited to meet you and wants to play!

## Commands and Tricks

- `myblock [NAME]: BLOCK`: your own custom block type. Declare a block using `CustomBlock(name, action_routine)`. In most cases, you'll want `CustomClassLikeBlock(superclass)`, eg, if you're defining a bunch of subclasses of the same superclass and don't want to write the full subclass declaration signature `class name (superclass): ...` every time.
- OOP:
  - `module [NAME]`: inline module. Has a clean variable scope
  - `scope [NAME]`: declare a nested scope. Objects outside the scope are still accessible from inside, but they can temporarily be overshadowed. 
  - `class [NAME] ([extends SUPERCLASS] [implements comma_separated[INTERFACE]])|tuple_dict|e: BLOCK`: regular class. A class is a scope that can be instantiated and used as a type.
  - `extend [class]: BLOCK`: updates a class by updating it with the contents of the following block.
  - `static [class]: BLOCK`
  - `sub|subclass`
  - `abstract [class]`
  - `interface [class]`
  - `private [CLASS_LIKE_BLOCK]` (e.g. `private class`, `private static class`, `private abstract class`, `private interface`)
  - `data`: coconut data class
  - `relation`: a data class but where non-assignment expressions are converted to `__anonymous__.append(expression)`. This mixes named and anonymous attributes, which is useful for defining complex type constraints like so:
    ```python mutt
    relation HydroxylGroup:
      Oxygen, Hydrogen
      Bond(Oxygen, Hydrogen)
    
    hydroxyl_containing_molecule = ... where hydroxyl_containing_molecule matches HydroxylGroup 
    ```
- other languages:
  - `python: BLOCK`, `coconut: BLOCK`, `cython: BLOCK`: restricts appropriate syntax and semantic patterns to interpret in the target language
  - `yaml: BLOCK`, `json { JSON } `, `toml: BLOCK`, `<xml> XML </xml>`, `<html> HTML </html>`, `<svg> SVG </svg>`, `\begin{latex} LATEX \end{latex}`: parses each in their respective language. Values can be Mutt expressions enclosed in braces `{}`, but this can be disabled by writing `pure [language]` in the block declaration to, eg, make single-item sets in json. yaml, json, and toml are converted to native python structures. The remaining are converted to domain-specific objects.
  - `javascript|js { JAVASCRIPT_CODE }`, `java { JAVA_CODE }`, `c { C_CODE }`, `c++ { CPP_CODE }`, `[common] lisp ( COMMON_LISP_CODE )`: performs limited interpretation of the code in the appropriate language. For now, I'm thinking these languages will be executed using an appropriate REPL. Serializable scope values that were already declared before the block are initialized in the REPL before executing the block. Then the REPL's scope is dumped, serialized, and deserialized back into the Mutt scope. Language identifiers can also prepend their respective function or class identifier for one-line custom-language class or function definitions, eg, `javascript function foo() { ... }` or `c int bar(char* s) { ... }`.
  - `copilot`
  - `comment`, `note`
- Control flow:
  - `def [NAME] (SIGNATURE) BLOCK`: function definition.
  - `with`
  - `if CONDITION: BLOCK`, `else if CONDITION: BLOCK` / `elif CONDITION: BLOCK`, `else: BLOCK`, `unless CONDITION: BLOCK`: Standard if-else-elif-else control flow. Unless checks are like if/elif checks except they take semi-higher precedence: for each section of consequtive if/elif/else checks followed by unless checks, the interpretter first evaluates all the unless checks from top-to-bottom before going through the standard if/elif/else checks, from top-to-bottom; then the interpretter moves on to the next continuous set of unless checks; then covers any if/elif/else's passed over before the last executed unless; and so on. For example:
    ```python
    if checked_second: ...
    unless checked_first: ...
    elif checked_fifth: ...
    elif checked_sixth: ...
    unless checked_third: ...
    unless checked_forth: ...
    else: ... # evaluated if all previous conditions were false
    ```
  - `when [NAME] (CONDITION): BLOCK`, `whenever [NAME] (CONDITION): BLOCK`: Reactive programming primitives. Don't ask how. `when` only runs once; `whenever` can be triggered an indefinite number of times. Named when/ever blocks can be disabled/enabled and passed around. Can be auto-disabled declared along a `with` statement.
  - `while CONDITION: BLOCK`, `until CONDITION: BLOCK`, `do BLOCK while CONDITION`, `do BLOCK until CONDITION`: Standard while/until loops. Until repeats until the condition is true. The do-while/do-until loops run once before checking the condition.
  - `repeat
  - `goto LABEL_NAME`, `label LABEL_NAME`, `comefrom LABEL_NAME: BLOCK`: standard goto/label statements. Comefrom is a way to execute code in a block only if LABEL_NAME was most recently jumped to. These operators only work inside a given scope (function/module/class/etc) and comefrom only tests if LABEL_NAME was most recently jumped to (as opposed to ever having been jumped to) to prevent goto abuse. 
  - `match`, `switch`, `case`, `default`
- TODO get coconut's other features
- TODO get python's other features
- Other:
  - `where` clauses: `IDENTIFIER [: TYPE] [where CONDITION] [= EXPR]` (or assignment and then `where` constaints). If a constraint is specified, appropriate changes are made to ensure it is respected.
    - In functions, the constraints are asserted before executing the function body. 
    - In class attributes and variables, the assigned field is converted to a `property`, and the setter first evaluates the constraint before assigning the value to its respective hidden variable.
    where clauses can make really cool generics like so:
      ```
      x where x is int
      
      def foo(a where a > 0, b where b >= 0 and a+b==3): ...

      class Foo:
        x: int where x > 0
        y: int where y >= 0 and x+y==3 = 3
        z where baz(z, ...)
        ...
      ``` 
  - inline assignment: `EXPR := IDENTIFIER := EXPR | ...`. Introduced in Python 3.10.
  - `lazy DECLARATION`: tells the interpretter to not throw an error immediately if an identifier is not found. Instead, unknown identifiers are temporarily converted to `UndefinedVariable` objects and re-checked when the declaration finishes. Examples are `lazy class Foo: ...` and `lazy foo(new_var, (new_var := A(), B(), C()))`.
  - custom operators (include unicode 🥳)
  - whitespace operator (`__whitespace__(self, other, spaces)`): optional low-precedence operator to perform operations on adjacent identifiers. Useful for writing CFG's directly in code without having to write out all the comma's. 
  - multi word identifiers: multi word identifiers take even lower precedence than the whitespace operator but higher precedence than a single word identifier.
  - `start to stop`: syntactic sugar for `range(start, stop)`
  - `TODO [text]`: syntactic sugar for `todo(text)`. marks a task to be done.

## Anatomy and Physiology

Mut is a superset of the coconut programming language, which is itself a superset of python. Code can be interpretted or compile down to coconut, python, and maybe even C (via cython).

Mutt code is interpretted as follows:

1. The main Mutt interpretter is created and started.
  - It is also started by writing `import mutt` in any python file. This makes the outer python interpretter stop at the import statement, and the Mutt interpretter takes over.
2. The initial file is opened as a stream and passed to the lexer
  - The lexer generates a token chain graph
  - The default lexer supports unicode 🥳
3. There is not a clean distinction between parsing and semantic analysis
  1. The Mutt interpretter looks for matches between its patterns and the token graph.
  2. When a pattern is matched, the interpretter performs the action routine associated with that pattern.
  3. The action routine interacts with the interpretter to construct an `ASTNode` starting from the root module and working recursively down.
      - The action routine may also modify the existing AST (relative to the root or its parents)
      - Sometimes, a match may signal that the language dynamics are changing
      - The pattern's action routine may ask the interpretter to lex more tokens (and the lexer may consume more of the input stream) or it may direct the interpretter to check for more matches on a specific section of the AST until it is satisfied.
        - A module `ASTNode` calls the interpretter on its contents and places its internal statements' ASTs in its `._body` attribute.
        - A class definition must obtain the `ASTNode`s for its body before returning itself. The class makes a placeholder for itself before lexing and parsing its body; that allows internal methods to use the containing class' type in their signature.
      - The result of this recursive process is a top-level module AST node containing all the ASTs of the top-level statements, and so on.
4. The AST is executed (mostly) bottom up, interleaved with parsing.
   - Node execution happens in post-order by default, but nodes can change this behavior if needed 
   - AST nodes have a `python` getter that returns the python code that would be generated by that node.
   - ASTNodes call the Mutt interpretter's interpret(python_code: str) function. Depending on the behavior of the interpretter, this code is executed or it is streamed to a file.

```yaml python
Pattern:
 # Pattern fn's: these fn's need to be implemented in the subclass or at least the instance. subclasses should implement this as low as possible to avoid unnecessary overhead (i.e.: use a DFA parser if applicable instead of a CSG parser).
- match_fn: (tokens, mutt) -> bool = None
- claim_fn: (tokens, mutt) -> Token[] = None
- action_fn: (tokens, mutt) -> ASTNode = None
: CSGPattern:
  - csg_grammar
  - ... Pattern fn's
  : CFGPattern:
    # written using eBNF
    - cfg_grammar
    - ... Pattern fn's
    : NFAPattern:
      - state_machine: StateMachine
      - ... Pattern fn's
      : RegexPattern:
        - regex: str

Mutt:
# the interpretter is always running, but you have to "import mutt" to get access to "mutt.Mutt"
- lexer: Lexer
- patterns: Pattern[]
- modules: Module[]
- main_module: ASTNode
- mode: 'exec'|'compile'

patterns:
- ModulePattern
- ClassPattern
- DoWhilePattern
...

LOC = (line, column, total_char_offset)

ASTNode:
- source: str
- source_start: LOC
- parent: ASTNode | None # none if this is the root node
- get_python: () -> str = None # implemented by subclasses

ModuleASTNode(ASTNode):
- statements: ASTNode[] # order preserved
```

## Development

1. Make it work on pure python syntax
   1. Generate the CFGPattern's from the Python syntax file
   2. Make appropriate AST node types, and maybe use the official python AST node types
   3. Make the part of the interpreter that renders the AST back into python code
   4. Test cycle consistency on lots of programs
2. Make it work on pure coconut syntax
3. Add features incrementally
4. Integrate Hy support, which provides native lisp for python: https://docs.hylang.org/en/alpha/index.html

## Examples

```python coconut mutt
relation(x, y) where x > y # ???



```

## Strachpad


: RelationalPattern
  # Used for writing graph languages
  - relations: tuple_dict
  - ... Pattern fn's
  # assignments to a relation that uses where clauses will raise an error if the where clause is not satisfied. This can be used in the match_fn to identify a suitable match.

The relation pattern is not used in the parsing phase. Although it is used in a rule-matching system