# Use of GenAI to validate the ChefScript BNF

> Availability note: This repository contains the grammar and worked examples.
> The `chefscript_parser.py` and `validate_chefscript.py` files mentioned below were
> not present in the source folder. The executable results in this historical
> report have not been independently reproduced during publication.

## 0. Setup and honesty note

- **Tool used:** Claude (Anthropic), in the Claude chat interface.
- **Material validated:** `ChefScript_BNF_v1.0.txt` (Lab 1 grammar) and the updated `ChefScript_BNF.txt`, plus the two programs `Program1_GreekSalad.md` and `Program2_SaltySoup.md`.
- **Method:** seven prompts in sequence. Prompts 1-6 are analytical (the model reads the grammar and reasons about it). Prompt 7 is executable: the model builds a real LALR(1) parser from the grammar and runs the programs and tests, so its result does not depend on reasoning alone. Each prompt pastes the grammar (shown below as `[BNF v1.0]` to save space) and asks for one kind of check.
- **Important:** the prompts below were prepared in the same chat session that produced the outputs, and the outputs are condensed from the model's answers. If your course requires the raw transcript of your own session, re-run the prompts in your tool, paste the real answers into section 2, and compare them with the findings in section 3. Findings were checked by hand against the grammar before being accepted (section 4).

## 1. Summary of the sequence

| # | Goal of the prompt | Result |
|---|---|---|
| 1 | Is the BNF well formed (notation, undefined / unused non-terminals)? | 5 findings |
| 2 | Ambiguity, left recursion, parsability (LL / LR) | 3 findings |
| 3 | Can every statement of the two programs be derived? | Yes, no failures |
| 4 | Do invalid programs get rejected? | 3 invalid programs wrongly accepted or only semantically rejected |
| 5 | Is the grammar enough for the domain? Recommendations | 8 recommendations, 5 accepted, 3 rejected |
| 6 | Re-validation of the updated grammar v1.1 | All previous findings resolved |
| 7 | Executable validation: real LALR(1) parser, programs run, negative tests | All checks pass; 2 scanner notes added (grammar v1.2) |

## 2. Prompts and outputs

---

### Prompt 1 - Well-formedness of the BNF

**Input**

```
You are a compiler-construction expert. Below is the BNF of a small
domain-specific language called ChefScript (recipe computation).
Check ONLY the notation and well-formedness of the BNF:
- non-terminals used but never defined
- non-terminals defined but never used
- informal constructs that are not valid BNF (such as "...", or prose)
- inconsistencies between the lexical and syntactic parts
List each problem with the rule it appears in, and suggest a fix.

[BNF v1.0]
```

**Output (condensed)**

1. **Undefined non-terminals:** `<tab>` and `<newline>` are used in `<whitespace>` (A.6) but never defined. Fix: define `<space>`, `<tab>`, `<newline>`.
2. **Unused non-terminals:** `<arith_op>`, `<separator>` and `<assign_op>` are defined in A.5 but the syntax rules write `"+"`, `"("`, `"="` directly. Fix: either use them or remove them. I suggest using `<assign_op>` in `<assign_stmt>` and replacing the other two by a plain token list, since arithmetic operators appear at different precedence levels and the separators appear individually.
3. **Non-BNF construct:** `<letter> ::= "a" | "b" | ... | "z"` uses an ellipsis. Fix: list all letters explicitly.
4. **Escape inside a terminal:** `"\""` mixes the grammar notation with a programming-language escape. Fix: introduce `<dquote> ::= '"'` and use it.
5. **Prose definitions:** `<string_char>` ("any printable character except the double quote") and `<comment>` are described informally. The comment rule is acceptable as a lexical convention, but should say what ends it (newline, end of file). The string rule should also exclude newline.

---

### Prompt 2 - Ambiguity, left recursion, parsability

**Input**

```
Using the same BNF, analyse the grammar for:
1. ambiguity (two different parse trees for one input)
2. left recursion
3. whether it is LL(1) and whether it is LR(1)/LALR(1)
4. the dangling-else problem
Give a concrete example for each problem found and say whether it must
be fixed or only documented.

[BNF v1.0]
```

**Output (condensed)**

1. **Ambiguity:** none found. Expression precedence is encoded by `<expression>` / `<term>` / `<factor>`, and condition precedence by `<condition>` / `<and_cond>` / `<not_cond>`. `a - b - c` has exactly one tree: `(a - b) - c`. Chains such as `a < b < c` are impossible because `<relation>` is not recursive.
2. **Left recursion:** `<expression>`, `<term>`, `<condition>` and `<and_cond>` are directly left-recursive. This is *correct* for LR parsers (and gives left associativity) and *not allowed* for LL / recursive-descent parsers. Document it; no change needed unless the course requires LL(1).
3. **Parenthesis conflict:** `(` can start a `<not_cond>` of the form `"(" <condition> ")"` *and* a `<relation>` whose left operand is `"(" <expression> ")"`. Example: `(a + b) > 3` vs. `(a > 3 or b > 2)`. Not an ambiguity (the full input always has one parse), but an LL(1) parser cannot decide with one token of lookahead. Document it.
4. **Dangling else:** no problem, since every block is enclosed in `{ }`.
5. **Left factoring:** the two alternatives of `<if_stmt>` share the prefix up to `}`. Needs left factoring for LL parsers only.

**Verdict:** the grammar is unambiguous and suitable for LR / LALR (yacc, bison, ANTLR with its LL(*) is also fine). Document items 2, 3 and 5.

---

### Prompt 3 - Derivation of the two test programs

**Input**

```
Check whether each statement of the two ChefScript programs below can be
derived from the grammar. For each construct that might be problematic
(multiple statements on one line, indexed variables as targets of read,
"and" in conditions, if with and without else, strings containing
punctuation, comments), say which grammar rules derive it, or report a
failure.

[BNF v1.0]
[Program 1 - GreekSalad]
[Program 2 - SaltySoup]
```

**Output (condensed)**

| Construct | Rules used | Result |
|---|---|---|
| `kcal100[0] = 15;   salt100[0] = 2;` (two statements on a line) | `<stmt_list> ::= <statement> <stmt_list>`; whitespace ignored | OK |
| `read grams[i];` | `<read_stmt>` -> `"read" <variable>`; `<variable> ::= <identifier> "[" <expression> "]"` | OK |
| `if servings > 10 and salt_serv <= 700 { ... }` | `<if_stmt>` (no else); `<and_cond> "and" <not_cond>`; `<=` is in `<rel_op>` | OK |
| `while servings < 1 { ... }` | `<while_stmt>` with a `<relation>` as condition | OK |
| `if salt_serv > 700 { ... } else { ... }` | `<if_stmt>` with `else` | OK |
| `show "Per serving:", kcal_serv, "kcal,", salt_serv, "mg salt";` | `<output_list>` with `<string_const>` and `<expression>` items; the punctuation inside the quotes is `<string_char>` | OK |
| `# comment` after a statement | scanner discards `<comment>` | OK |
| Identifiers `kcal100`, `salt_serv`, `volume_ml`, `salt_mg` | `<identifier>`; none equals a keyword | OK |
| `for i from 0 to 2 { ... }` | `<for_stmt>` | OK |
| `list of number[3]` | `<array_type>` | OK |

**Verdict:** both programs derive completely. Program 1 (26 statements) alone uses every production of `<statement>` and `<type>`; Program 2 (9 statements) is a short program using a subset. The operators `-` (binary and unary), `%`, `!=`, `or` and `not` are not used by either program but are derivable from the grammar.

**Side observation:** a first draft of Program 1 had to repeat a calorie-sum loop because the language has no procedures. The final, simplified Program 1 avoids this. It is a limitation of the language, not an error in the grammar.

---

### Prompt 4 - Negative tests (invalid programs must be rejected)

**Input**

```
For each fragment below, say whether the BNF accepts or rejects it, and
whether that is the desired behaviour for a recipe language. If the
grammar accepts something it should not, propose a grammar fix; if the
problem can only be caught semantically, say so.

 1.  let a : list of number[0];
 2.  x = ;
 3.  if a > 1 { }
 4.  for i from 1 { show i; }
 5.  let if : number;
 6.  y = a < b < c;   (as a condition: while a < b < c { ... })
 7.  x = grams;       (grams declared as a list)
 8.  show;
 9.  read 5;
10.  if a > 1 { x = 1; } else if a > 0 { x = 2; }
11.  while x { show x; }
12.  y = 007;
13.  x = (1 + 2;
14.  show "unterminated;

[BNF v1.0]
```

**Output (condensed)**

| # | v1.0 verdict | Desired? | Comment / fix |
|---|---|---|---|
| 1 | **Accepted** | No: a list of 0 elements is useless | Grammar fix: array size must be `<positive_const>` |
| 2 | Rejected | Yes | `<expression>` is required |
| 3 | Rejected | Acceptable | `<stmt_list>` must be non-empty; keep (see Prompt 5) |
| 4 | Rejected | Yes | `"to"` part is mandatory |
| 5 | **Accepted** | No | `<identifier>` derives any letter sequence, including keywords. Grammar fix: add an explicit constraint / `<keyword>` list excluded from `<identifier>` (cannot be expressed in pure BNF, so a constraint line is the usual practice) |
| 6 | Rejected | Yes | `<relation>` is not recursive; no chained comparison |
| 7 | **Accepted** | No (type error) | Cannot be fixed in the grammar without splitting variable kinds; document as a semantic rule |
| 8 | Rejected | Yes | `<output_list>` is non-empty |
| 9 | Rejected | Yes | `read` needs a `<variable>` |
| 10 | Rejected | Acceptable | `else` must be followed by `{`. There is no `else if`; nested `if` inside the `else` block works instead |
| 11 | Rejected | Yes | A condition must be a relation (no "truthy" numbers) |
| 12 | Rejected | Yes | No leading zeros; `007` scans as `0 0 7` |
| 13 | Rejected | Yes | unbalanced parenthesis |
| 14 | Rejected by the scanner | Yes | the string has no closing `<dquote>` |

**Summary:** items 1 and 5 are real grammar defects, item 7 is a semantic rule that is missing from the documentation.

---

### Prompt 5 - Domain completeness and recommendations

**Input**

```
Given the domain (computing calories, salt and timing for recipes) and the
minimal requirements (one simple type, one user-defined type, assignment,
conditional, loop, input/output), list the recommended changes to the
grammar and language. Classify each as MUST (defect), SHOULD
(quality / clarity) or COULD (new feature) and justify. Keep the
language small.

[BNF v1.0]
```

**Output (condensed)**

| ID | Class | Recommendation |
|---|---|---|
| R1 | MUST | Array size must be at least 1 (negative-test finding 1) |
| R2 | MUST | Keywords must not be usable as identifiers (finding 5) |
| R3 | MUST | Define `<tab>` and `<newline>` (Prompt 1, finding 1) |
| R4 | SHOULD | Replace the ellipsis in `<letter>`, the `"\""` escape and the informal string / comment rules by explicit BNF (Prompt 1, findings 3-5) |
| R5 | SHOULD | Remove unused `<arith_op>` / `<separator>`, use `<assign_op>` (Prompt 1, finding 2) |
| R6 | SHOULD | Add semantic notes: array name without index is invalid; the loop variable must be declared and not assigned inside the body; loop bounds are evaluated once; variables must be assigned before use |
| R7 | SHOULD | Add a parser-notes section on left recursion and the `(` lookahead (Prompt 2) |
| R8 | COULD | Optional extras: initialisers in declarations (`let x : number = 5;`), `else if`, procedures/functions, empty blocks, a boolean type, `for` with a step or downwards |

**Reasoning given for R8:** these make the language more convenient but are not needed to meet the requirements, and each adds grammar rules (and a conflict risk for `else if` and initialisers on arrays). Recommend leaving them out of version 1.1 and listing them as future work.

---

### Prompt 6 - Re-validation of version 1.1

**Input**

```
Here is the revised grammar (version 1.1). Re-run the earlier checks:
well-formedness, ambiguity, derivation of the two programs, and the 14
negative tests. Report any remaining defect, and any NEW defect
introduced by the changes.

[BNF v1.1]
[Program 1]  [Program 2]  [14 negative fragments]
```

**Output (condensed)**

- **Well-formedness:** every non-terminal used is defined (`<space>`, `<tab>`, `<newline>`, `<dquote>`, `<keyword>`, `<positive_const>`, `<assign_op>`, `<name>`, ...), and every defined non-terminal is used. `<digit>`, `<nonzero_digit>` and `<positive_const>` are consistent: `<digit> ::= "0" | <nonzero_digit>`.
- **Ambiguity:** still none. `<number_const> ::= "0" | <positive_const>` introduces no overlap: the array type uses only `<positive_const>`, and expressions use `<number_const>`.
- **Programs:** both still derive completely.
- **Negative tests:** #1 now rejected (size 0). #5 now rejected by the keyword constraint. #7 is still accepted by the grammar and is now documented as semantic rule 3 in Part C. All other results unchanged.
- **New defects:** none. One remark: the keyword constraint is a side condition, not a BNF rule. This is normal in language definitions (the scanner checks identifiers against the keyword table).

---

### Prompt 7 - Executable validation with a real parser

**Input**

```
Do not only reason: build a real parser from the ChefScript BNF (LALR(1),
for example with Python Lark) and use it to validate the language.
1. Report any shift/reduce or reduce/reduce conflict.
2. Parse every program in the lab documents (Program 1, Program 2, the
   Lab 1 use-case program, the BNF sample program, and the code fragments
   of the use-case document).
3. Write a small interpreter and check that each program prints exactly
   the output claimed in its document, for the documented inputs.
4. Count the executable statements of the two programs and check which
   statement kinds they cover.
5. Run the 14 negative tests (plus a few more) and a list of inputs that
   must be ACCEPTED (identifiers that start with a keyword, unary minus
   chain, parenthesised expressions and conditions, empty string, ...).
6. Report every failure and say whether the grammar, the program or the
   test is wrong. Fix and repeat until everything passes.

[BNF v1.1]  [Program 1]  [Program 2]  [use-case document]
```

**Output (actual run; the code is in `chefscript_parser.py` and `validate_chefscript.py`)**

| Check | Result |
|---|---|
| Parser conflicts (LALR(1)) | **0** shift/reduce, **0** reduce/reduce. This also confirms that the `(` situation of Prompt 2 is not a conflict for an LR parser. |
| Programs that parse | Program 1, Program 2, Lab 1 Bruschetta program, BNF sample program (CaesarSalad): all parse |
| Code fragments of the use-case document | all 8 parse (declared variables added around them) |
| Execution against the documented output | Program 1 (3 input sets), Program 2 (2 input sets), Bruschetta, CaesarSalad: **all outputs identical** to the documents (e.g. `Per serving: 207 kcal, 1112 mg salt`, `Add water until volume is 3000 ml`) |
| Statement count | Program 1: **26** (medium, at least 20); Program 2: **9** (simple) |
| Statement kinds covered by the two programs | simple declaration, array declaration, assignment, indexed assignment / read, `read`, `show`, `if ... else`, `if` without `else`, `for`, `while`: **all covered** |
| Negative tests | all 16 invalid programs rejected (list size 0, keyword as identifier, empty `if`, `else if`, chained comparison, `007`, missing `;`, declaration after a statement, ...) |
| Positive tests | accepted: `iffy` as identifier, `- - 3`, `(a + 1) * 2 > 3`, `not (a > 1 or a < 0) and a == 0`, comment after statements, `show "";` |

**Failures found in the first run, and what was wrong**

| Failure | Cause | Action |
|---|---|---|
| "P1 test 12,100,0,0 full" | Wrong output index in the *test script* (no retry line is printed for a valid first input) | Test fixed; document was correct |
| "comment + two statements per line" | The *test* put `# comment` before the closing `}` on the same line, so the comment swallowed the brace | Test fixed; grammar was correct |
| Strict lexer: collision between `<positive_const>` and `<number_const>` | Real observation about the grammar: the text `5` is both. It is harmless for a context-sensitive scanner but must be stated | Scanner note added to the BNF (version 1.2) |
| (Review of the test results) | Keywords must match whole words, as shown by the accepted identifier `iffy` | Whole-word rule added to the lexical rules (version 1.2) |

Two lazy checks in my first version of the test script (`... or True`) were removed, and the expected value of the CaesarSalad test was corrected by hand (13 + 160 + 190 = 363 kcal, times 2 servings = 726, not 566) before the final run.

**Verdict:** the grammar v1.1 is valid, conflict-free and unambiguous in practice. Only two documentation notes were needed (no production changed), giving version 1.2. Final run: **all checks pass**.

---

## 3. Findings collected

| ID | Finding | Source prompt | Severity |
|---|---|---|---|
| F1 | `<tab>`, `<newline>` undefined | 1 | Must |
| F2 | `<arith_op>`, `<separator>`, `<assign_op>` unused | 1 | Should |
| F3 | Ellipsis in `<letter>` is not BNF | 1 | Should |
| F4 | `"\""` escape in a BNF terminal | 1 | Should |
| F5 | Informal string / comment definitions | 1 | Should |
| F6 | Left recursion (LR ok, LL not) | 2 | Document |
| F7 | `(` lookahead conflict between condition and expression | 2 | Document |
| F8 | `<if_stmt>` needs left factoring for LL parsers | 2 | Document |
| F9 | Array size 0 accepted | 4 | Must |
| F10 | Keywords accepted as identifiers | 4 | Must |
| F11 | Array name alone accepted as a value | 4 | Semantic rule |
| F12 | No procedures: repeated code in a first draft of Program 1 (removed by simplifying it) | 3 | Language limitation, future work |
| F13 | `<positive_const>` / `<number_const>` overlap and whole-word keyword matching must be stated for the scanner | 7 | Document (version 1.2) |

## 4. Decisions: what was changed in the grammar (Requirement 3)

Each recommendation was checked by hand against the grammar before it was accepted.

| Recommendation | Decision | Change in `ChefScript_BNF.txt` (v1.1) |
|---|---|---|
| R1 array size >= 1 | **Accepted** | New `<positive_const>`; `<array_type> ::= "list" "of" "number" "[" <positive_const> "]"`; `<number_const> ::= "0" \| <positive_const>` |
| R2 keywords not identifiers | **Accepted** | `<identifier> ::= <name>` with a CONSTRAINT line, new `<keyword>` rule listing the 16 reserved words |
| R3 define tab/newline | **Accepted** | New `<space>`, `<tab>`, `<newline>` in A.6 |
| R4 explicit BNF for letters, quote, string, comment | **Accepted** | Letters listed in full; `<dquote> ::= '"'`; string chars exclude newline; `<comment>` formalised with `<comment_body>` and `<comment_char>` |
| R5 unused non-terminals | **Accepted** | `<assign_op>` now used in `<assign_stmt>`; `<arith_op>` and `<separator>` replaced by a token list |
| R6 semantic notes | **Accepted** | Part C extended with notes 3, 6, 7, 8 |
| R7 parser notes | **Accepted** | New Part E |
| R8 optional extras (initialisers, `else if`, procedures, empty blocks, boolean, `for` step) | **Rejected for now** | Not needed for the requirements; would enlarge the grammar. Kept as future work. The use of nested `if` inside `else` replaces `else if`. |
| R9 scanner notes (whole-word keywords, constant overlap, reference parser) | **Accepted** | Notes added in Part A.6 and Part E; version history entry 1.2 (no production changed) |

Everything else (statements, expressions, conditions, precedence) is **unchanged**, so Program 1, Program 2 and the Lab 1 use-case document remain valid with grammar v1.1. The version history is also recorded at the top of the BNF file.

## 5. Conclusion

GenAI was useful for four things: a systematic well-formedness check (it found undefined and unused non-terminals that are easy to overlook when reading), a list of negative tests (which exposed the array-size and keyword defects), an explanation of parser consequences (left recursion, parenthesis lookahead), and, in Prompt 7, an executable check: a parser generated from the BNF reported no conflicts, accepted all lab programs, rejected all invalid ones, and reproduced every documented program output. Its answers still had to be checked against the grammar by hand: for example, for the unused `<arith_op>` and `<separator>` the tool offered "use or remove", and the decision (use `<assign_op>`, remove the other two) was made by weighing which rules the syntactic part actually refers to. The grammar was therefore also run through a real parser generator (Prompt 7), which found no grammar defect and led to two scanner notes. The final grammar is version 1.2.
