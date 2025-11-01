This rule, named "ADUserNameGenerator," is designed to create a unique username for Active Directory accounts in SailPoint IdentityNow, following specific formatting and length constraints.

Here's a step-by-step breakdown:

---

# ADUserNameGenerator Rule Explanation

This rule generates a unique username, typically in the format of `firstname.lastname` or a variation, ensuring it's no longer than 12 characters and hasn't been used before.

## How it Works (Step-by-Step)

1.  **Get User Information:**
    *   It starts by getting the user's `firstName`, `lastName`, and an optional `otherName` (like a preferred name or middle name) from their identity profile.

2.  **Clean Up Names:**
    *   It removes any special characters (like hyphens, spaces, apostrophes) from `firstName`, `lastName`, and `otherName`, leaving only letters and numbers.
    *   If `firstName` or `lastName` are empty after cleaning, the rule stops and returns nothing, as a username cannot be generated.

3.  **Prioritize "Other Name":**
    *   **Important:** If an `otherName` is provided, it *replaces* the `firstName` for the purpose of username generation. This means `otherName` takes precedence over the official `firstName`.

4.  **Determine Username Length Strategy (Max 12 Characters):**
    *   The rule has a strict maximum username length of 12 characters (`MAX_USERNAME_LENGTH = 12`). It checks if the combination of `firstName.lastName` would exceed this limit.

    *   **Scenario A: `firstName.lastName` is 12 characters or less.**
        *   It first tries to create the username as `firstName.lastName` (e.g., `john.doe`).
        *   If this username is **unique**, it uses it.
        *   If it's **not unique** (meaning another user already has `john.doe`), it then tries variations by taking the `firstName` and adding a dot followed by *each character* of the `lastName` one by one (e.g., `john.d`, then `john.o`, then `john.e`). It stops at the first unique one it finds.

    *   **Scenario B: `firstName.lastName` is longer than 12 characters.**
        *   This means the username must be shortened.
        *   **Sub-Scenario B1: The `firstName` itself is very long (more than 10 characters).**
            *   It will shorten the `firstName` to its first 10 characters. (Because `MAX_USERNAME_LENGTH - 2` = 12 - 2 = 10 characters for the first name part, leaving 2 characters for `.L`).
            *   Then, it tries variations by taking this shortened `firstName`, adding a dot, and followed by *each character* of the `lastName` one by one (e.g., if `firstName` was "Christopher" and `lastName` was "Smith", it might try `christophe.s`, then `christophe.m`, etc.). It stops at the first unique one.
        *   **Sub-Scenario B2: The `firstName` is 10 characters or less (but `firstName.lastName` is still > 12 characters).**
            *   It uses the full (cleaned) `firstName`.
            *   Then, it tries variations by taking this `firstName`, adding a dot, and followed by *each character* of the `lastName` one by one (e.g., if `firstName` was "Alexander" and `lastName` was "Hamilton", it might try `alexander.h`, then `alexander.a`, etc.). It stops at the first unique one.

5.  **Check for Uniqueness:**
    *   For every username generated, it calls a helper function `isUnique()` which checks if an account with that exact username (treated as a `displayName` in Active Directory context) already exists on the target application (Active Directory in this case).
    *   If `isUnique()` returns `true`, the username is valid and the rule returns it.

6.  **Final Result:**
    *   The rule returns the first unique username it successfully generates.
    *   If, after trying all possible combinations, it cannot find a unique username that fits the criteria, it returns `null`.

## Example of How it Works

Let's trace an example with a user named **"Robert" "Smith"** with an `otherName` of **"Bob"**.
We'll assume `MAX_USERNAME_LENGTH = 12`.

**User Data:**
*   `identity.getFirstname()`: "Robert"
*   `identity.getLastname()`: "Smith"
*   `identity.getStringAttribute("otherName")`: "Bob"

**Step-by-Step Execution:**

1.  **Get User Information:**
    *   `firstName` = "Robert"
    *   `lastName` = "Smith"
    *   `otherName` = "Bob"

2.  **Clean Up Names:**
    *   `firstName` becomes "Robert" (no special chars)
    *   `lastName` becomes "Smith" (no special chars)
    *   `otherName` becomes "Bob" (no special chars)

3.  **Prioritize "Other Name":**
    *   Since `otherName` ("Bob") is not empty, `firstName` is *reassigned* to "Bob".
    *   Now, for username generation, `firstName` = "Bob" and `lastName` = "Smith".

4.  **Determine Username Length Strategy:**
    *   Calculate `fullName` using the (reassigned) `firstName`: `"Bob" + "." + "Smith"` = `"Bob.Smith"`.
    *   Length of `"Bob.Smith"` is 9 characters.
    *   Is 9 > `MAX_USERNAME_LENGTH` (12)? No, 9 is less than or equal to 12.
    *   So, it falls into **Scenario A**.

5.  **Generate and Check Uniqueness (Scenario A):**
    *   **Attempt 1:**
        *   Try `username = "Bob.Smith"`. Convert to lowercase: `"bob.smith"`.
        *   Call `isUnique("bob.smith")`.
        *   **Let's assume `isUnique("bob.smith")` returns `true` (meaning no other user has this username).**

6.  **Return Unique Username:**
    *   The rule finds a unique username: `"bob.smith"`.
    *   It returns `"bob.smith"`.

**Output:** `bob.smith`

---

**Another Example: Long Name, requiring truncation and iteration**

Let's assume `firstName = "Christopher"`, `lastName = "Smitherton"`, `otherName` is null.

1.  **Get User Information:** `firstName = "Christopher"`, `lastName = "Smitherton"`, `otherName = null`.
2.  **Clean Up Names:** `firstName = "Christopher"`, `lastName = "Smitherton"`.
3.  **Prioritize "Other Name":** `otherName` is null, so `firstName` remains "Christopher".
4.  **Determine Username Length Strategy:**
    *   `fullName = "Christopher.Smitherton"` (Length = 20 characters).
    *   Is 20 > `MAX_USERNAME_LENGTH` (12)? Yes.
    *   It falls into **Scenario B**.
    *   Is `firstNameLength` (11 for "Christopher") > `MAX_USERNAME_LENGTH - 2` (10)? Yes.
    *   It falls into **Sub-Scenario B1**. `firstName` will be truncated to 10 characters. So, `firstName` becomes `"Christophe"`.

5.  **Generate and Check Uniqueness (Sub-Scenario B1):**
    *   It will iterate through each character of `lastName` ("Smitherton"):
        *   **Attempt 1 (lastName char 'S'):** `username = "christophe.s"`. Call `isUnique("christophe.s")`.
            *   *Assume `isUnique("christophe.s")` returns `false` (already taken).*
        *   **Attempt 2 (lastName char 'm'):** `username = "christophe.m"`. Call `isUnique("christophe.m")`.
            *   *Assume `isUnique("christophe.m")` returns `true`.*

6.  **Return Unique Username:**
    *   The rule finds a unique username: `"christophe.m"`.
    *   It returns `"christophe.m"`.

**Output:** `christophe.m`
