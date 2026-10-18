# DECODE-DLE — Write-up

**Category:** Crypto  
**Author:** @G2Gh0st  
**Challenge:** DECODE-DLE  
**Flag:** `deadface{716}`

## Challenge Description

We are given an image containing a Mastermind-style number puzzle. Each row provides a three-digit guess along with a hint describing how many digits are correct and whether they are in the correct positions.

Our objective is to analyze the clues and determine the correct three-digit lock code.

## Given Clues

| # | Guess | Hint |
|---|---|---|
| 1 | `863` | One number is correct but in the wrong place |
| 2 | `250` | Nothing is correct |
| 3 | `168` | Two numbers are correct but in the wrong places |
| 4 | `735` | One number is correct and in the right place |
| 5 | `687` | Two numbers are correct but in the wrong places |

## Solution

### Step 1: Eliminate Incorrect Digits

According to Clue 2 (`250`), none of the digits are present in the code.

Therefore:

- `2` is eliminated.
- `5` is eliminated.
- `0` is eliminated.

### Step 2: Identify the Correct Digits

From Clue 1 (`863`), exactly one of the digits `8`, `6`, and `3` is present in the code.

Clue 3 (`168`) tells us that exactly two digits among `1`, `6`, and `8` are present.

Since Clue 1 allows only one of `6` and `8`, the code must contain:

- `1`
- Either `6` or `8`

Next, Clue 5 (`687`) tells us that exactly two digits among `6`, `8`, and `7` are present.

Since only one of `6` and `8` can be present, `7` must also be part of the code.

Finally, Clue 1 confirms that `3` is not present.

At this point, the three digits must be:

**`1`, `7`, and either `6` or `8`.**

### Step 3: Determine the Positions

Clue 4 (`735`) states that exactly one digit is correct and in the correct position.

Since `3` and `5` have already been eliminated, `7` must be the correct digit.

Therefore, the first digit is:

**`7 _ _`**

The remaining digits are `1` and either `6` or `8`.

This gives us two possible candidates:

- `716`
- `718`

### Step 4: Verify the Candidates

We can eliminate `718` using Clue 3 (`168`).

In `718`, the digit `8` appears in the third position, which is also its position in the guess `168`.

However, Clue 3 explicitly states that both correct digits must be in the wrong positions.

Therefore, `718` is invalid.

The remaining candidate is:

**`716`**

Let's verify it against all five clues.

| Clue | Guess | Verification with `716` | Result |
|---|---|---|---|
| 1 | `863` | `6` is present but in the wrong position | ✓ |
| 2 | `250` | None of these digits are present | ✓ |
| 3 | `168` | `1` and `6` are present, both in wrong positions | ✓ |
| 4 | `735` | `7` is present in the correct position | ✓ |
| 5 | `687` | `6` and `7` are present, both in wrong positions | ✓ |

All five clues are satisfied.

## Flag

```text
deadface{716}
```

## Conclusion

By eliminating impossible digits, identifying the correct digit set, and checking the positional constraints, we successfully solved the Mastermind-style puzzle.

**Final Flag:** `deadface{716}`
