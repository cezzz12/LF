# Program 2 - SaltySoup (simple)

**Language:** ChefScript, grammar version 1.2 (`ChefScript_BNF.txt`)

## 1. Problem

A pot of soup is too salty when it contains more than 400 mg of salt per 100 ml. The program reads the soup volume and the total salt in the pot, and if the soup is too salty it adds water, 100 ml at a time, until the concentration is acceptable.

## 2. Source code

```
recipe SaltySoup {
    let volume_ml : number;
    let salt_mg   : number;

    show "Soup volume (ml):";
    read volume_ml;
    show "Total salt in the pot (mg):";
    read salt_mg;

    if salt_mg * 100 / volume_ml > 400 {
        while salt_mg * 100 / volume_ml > 400 {
            volume_ml = volume_ml + 100;
        }
        show "Add water until volume is", volume_ml, "ml";
    } else {
        show "Soup is fine.";
    }
}
```

## 3. Size

The program has **2 declarations** and **9 executable statements** (`show` x4, `read` x2, `if`, `while`, assignment). It is deliberately small, in contrast with Program 1 (26 statements).

## 4. Statements used

| Statement | Where it is used |
|---|---|
| Declaration, simple type | `volume_ml`, `salt_mg` |
| Assignment | `volume_ml = volume_ml + 100;` |
| `read` | `read volume_ml;`, `read salt_mg;` |
| `show` | text only and text + expression |
| `if ... else` | too salty / fine |
| `while` | add water until the concentration is low enough |

The statements not used here (array declaration, element assignment and `read`, `for`, `if` without `else`) are covered by Program 1. Together the two programs use every statement kind of the language.

## 5. Why each part is there

- **`read`**: the same script works for any pot size.
- **`if ... else`**: nothing must be added when the soup is already fine.
- **`while`**: the number of water additions depends on the data, so it cannot be a `for` loop.

## 6. Sample run

Input values: `2000`, `12000`

```
Soup volume (ml):
Total salt in the pot (mg):
Add water until volume is 3000 ml
```

Trace: 12000 * 100 / 2000 = 600 (> 400). Each pass adds 100 ml: the concentration becomes 571, 545, 521, 500, 480, 461, 444, 428, 413 and finally 1200000 / 3000 = 400, which is not above 400, so the loop stops after 10 passes at 3000 ml.

## 7. Other test input

| Inputs | Expected output after the prompts |
|---|---|
| `2000`, `5000` | 5000 * 100 / 2000 = 250, not above 400, so `Soup is fine.` |

Note: a volume of 0 would divide by zero (a runtime error, see semantic note 4 of the BNF).
