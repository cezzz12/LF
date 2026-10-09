# Program 1 - GreekSalad (medium size, simplified)

**Language:** ChefScript, grammar version 1.2 (`ChefScript_BNF.txt`)

## 1. Problem

A cook prepares a Greek salad for a number of guests. The program must:

1. ask for the number of servings and re-ask until it is at least 1,
2. read the grams per serving of three ingredients (cucumber, feta, olives),
3. compute calories and salt per serving,
4. report the totals and warn if the salad is **salty** (more than 700 mg salt per serving),
5. announce a large party (more than 10 guests) when the salad is not salty.

## 2. Source code

```
recipe GreekSalad {
    let servings  : number;
    let i         : number;
    let kcal_serv : number;
    let salt_serv : number;
    let grams     : list of number[3];
    let kcal100   : list of number[3];
    let salt100   : list of number[3];

    # nutrition per 100 g: 0 = cucumber, 1 = feta, 2 = olives
    kcal100[0] = 15;   salt100[0] = 2;
    kcal100[1] = 264;  salt100[1] = 1100;
    kcal100[2] = 115;  salt100[2] = 1500;

    show "How many servings?";
    read servings;
    while servings < 1 {
        show "At least 1 serving, try again:";
        read servings;
    }

    show "Grams per serving (cucumber, feta, olives):";
    for i from 0 to 2 {
        read grams[i];
    }

    kcal_serv = 0;
    salt_serv = 0;
    for i from 0 to 2 {
        kcal_serv = kcal_serv + grams[i] * kcal100[i] / 100;
        salt_serv = salt_serv + grams[i] * salt100[i] / 100;
    }

    show "Per serving:", kcal_serv, "kcal,", salt_serv, "mg salt";
    show "Whole salad:", kcal_serv * servings, "kcal";

    if salt_serv > 700 {
        show "Warning: salty salad.";
    } else {
        show "Salt level is fine.";
    }

    if servings > 10 and salt_serv <= 700 {
        show "Large party: use a big bowl.";
    }
}
```

## 3. Size

The program has **7 declarations** and **26 executable statements** (assignments, `read`, `show`, `if`, `for`, `while`, counting nested ones). This satisfies the "at least 20 statements" requirement.

## 4. Coverage of the language statements

| Statement / construct | Where it is used |
|---|---|
| Declaration, simple type `number` | `servings`, `i`, `kcal_serv`, `salt_serv` |
| Declaration, user type `list of number[N]` | `grams`, `kcal100`, `salt100` |
| Assignment to a variable | `kcal_serv = 0;` |
| Assignment to an array element | `kcal100[0] = 15;` |
| `read` into a variable | `read servings;` |
| `read` into an array element | `read grams[i];` |
| `show` (text only) | `show "How many servings?";` |
| `show` (text + expressions) | `show "Per serving:", kcal_serv, ...;` |
| `if ... else` | the salt warning |
| `if` without `else` | the large party message |
| `for` loop | reading and summing the 3 ingredients |
| `while` loop | input validation |
| `and` in a condition | `servings > 10 and salt_serv <= 700` |
| Relational operators | `<`, `>`, `<=` |
| Arithmetic operators | `+`, `*`, `/` |

## 5. Why each part is there

- **`while servings < 1`**: the number of retries is unknown, so a `for` loop cannot do it.
- **`for ... read grams[i]`**: the same operation is repeated for every ingredient.
- **Second `for`**: adds calories and salt of all ingredients with one body instead of three copies.
- **`if ... else`**: exactly one of two messages must be printed.
- **`if` without `else`**: the large party message is optional; nothing happens otherwise.

## 6. Sample run

Input values (typed in this order): `0`, `4`, `100`, `60`, `30`

```
How many servings?
At least 1 serving, try again:
Grams per serving (cucumber, feta, olives):
Per serving: 207 kcal, 1112 mg salt
Whole salad: 828 kcal
Warning: salty salad.
```

## 7. Hand trace of the computations

Per serving with integer division:

| Ingredient | grams | kcal | salt (mg) |
|---|---|---|---|
| cucumber | 100 | 100*15/100 = 15 | 100*2/100 = 2 |
| feta | 60 | 60*264/100 = 158 | 60*1100/100 = 660 |
| olives | 30 | 30*115/100 = 34 | 30*1500/100 = 450 |
| **sum** | | **207** | **1112** |

- Whole salad: 207 * 4 = 828 kcal.
- 1112 > 700, so "Warning: salty salad." is printed.
- `servings > 10` is false (4), so the last `if` prints nothing.

## 8. Other test inputs

| Inputs | Expected output after the prompts |
|---|---|
| `12`, `100`, `0`, `0` | `Per serving: 15 kcal, 2 mg salt`, `Whole salad: 180 kcal`, `Salt level is fine.`, `Large party: use a big bowl.` |
| `15`, `100`, `60`, `30` | `Per serving: 207 kcal, 1112 mg salt`, `Whole salad: 3105 kcal`, `Warning: salty salad.` (no large party message, because the salad is salty) |
