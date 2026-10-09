# ChefScript language specification

ChefScript is a small language for recipe calculations: scaling ingredient quantities, estimating calories and salt, and diluting soup. This repository contains the language specification, worked programs and a historical validation report.

## Files and architecture

| File | Purpose |
|---|---|
| [ChefScript_BNF.txt](ChefScript_BNF.txt) | Version 1.2 grammar: lexical rules, program syntax, semantic constraints and parser notes. |
| [Program1_GreekSalad.md](Program1_GreekSalad.md) | Array-based nutrition calculation, input validation, loops and conditional messages. |
| [Program2_SaltySoup.md](Program2_SaltySoup.md) | Soup dilution using integer arithmetic and a while loop. |
| [GenAI_Validation.md](GenAI_Validation.md) | Historical grammar review, proposed corrections and reported validation results. |

The specification separates scanning, parsing and semantic analysis. A scanner recognizes identifiers, integer constants, strings, keywords and operators. A parser applies the BNF productions. A semantic checker would enforce declaration-before-use, valid array indexes and initialized values. An interpreter would execute the resulting statements.

## How it works

A program starts with `recipe Name { ... }`. Declarations precede statements. Values are integers or fixed-size integer arrays; indexes start at zero. Statements include assignment, `read`, `show`, `if`/`else`, inclusive `for` loops and `while` loops. Integer division truncates each arithmetic result. The grammar describes arithmetic and Boolean precedence.

Open the grammar and the two program documents in a text editor. Each program document contains its source, sample inputs, expected output and calculation trace. GreekSalad's input `0, 4, 100, 60, 30` produces 207 kcal and 1112 mg salt per serving, with 828 kcal for the whole salad. SaltySoup's input `2000, 12000` requires dilution to 3000 ml.

## Execution and verification

No compiler, interpreter or parser script is supplied. The `chefscript_parser.py` and `validate_chefscript.py` mentioned in the historical report were absent from the original folder. Its parser-conflict and execution results have therefore not been independently reproduced for this publication.

During publication, the grammar's non-terminal definitions and references were checked, and the documented GreekSalad and SaltySoup sample arithmetic was verified. These checks do not establish parser correctness or execute the ChefScript programs. The grammar and report now explicitly state that the referenced validation scripts are unavailable.
