# SailPoint IdentityNow Transforms Documentation

## Table of Contents

- [SailPoint IdentityNow Transforms Documentation](#sailpoint-identitynow-transforms-documentation)
  - [Table of Contents](#table-of-contents)
  - [1. Substring Transform](#1-substring-transform)
    - [Purpose](#purpose)
    - [Attributes](#attributes)
    - [How it Works (Step-by-Step)](#how-it-works-step-by-step)
    - [Examples](#examples)
      - [Example 1: Fixed Position Substring](#example-1-fixed-position-substring)
      - [Example 2: Extracting Data After a Delimiter](#example-2-extracting-data-after-a-delimiter)
  - [2. Lookup Transform](#2-lookup-transform)
    - [Purpose](#purpose-1)
    - [Attributes](#attributes-1)
    - [How it Works (Step-by-Step)](#how-it-works-step-by-step-1)
    - [Examples](#examples-1)
      - [Example 1: Country Code to Timezone Mapping](#example-1-country-code-to-timezone-mapping)
  - [3. ISO3166 Transform](#3-iso3166-transform)
    - [Purpose](#purpose-2)
    - [Attributes](#attributes-2)
    - [How it Works (Step-by-Step)](#how-it-works-step-by-step-2)
    - [Examples](#examples-2)
      - [Example 1: Standardizing Country Codes](#example-1-standardizing-country-codes)
  - [4. Within25days](#4-within25days)
    - [Purpose](#purpose-3)
    - [Attributes](#attributes-3)
    - [How it Works (Step-by-Step)](#how-it-works-step-by-step-3)
    - [Examples](#examples-3)
      - [Example 1: `end_date` is within 25 days from now](#example-1-end_date-is-within-25-days-from-now)
      - [Example 2: `end_date` is exactly 25 days from now (considering the date part)](#example-2-end_date-is-exactly-25-days-from-now-considering-the-date-part)
      - [Example 3: `end_date` is beyond 25 days from now](#example-3-end_date-is-beyond-25-days-from-now)

---

## 1. Substring Transform

The `substring` transform is used to extract a portion of a string (text) based on specified starting and ending positions or markers. It's extremely useful for parsing data where you need only a specific segment of a larger text field.

### Purpose

To extract a part of a larger string, such as a state from an address, initials from a name, or a specific code from an identifier.

### Attributes

- **`input`**: (Optional) The string to operate on. If not specified at the top level, it might be inferred from the context or expected to be passed as the transform's direct input. Can be a static string or reference an `accountAttribute`.
- **`begin`**: (Optional) The starting index for the substring.
  - Can be a numeric index (0-based).
  - Can be a nested `indexOf` transform to find a character/substring.
- **`end`**: (Optional) The ending index for the substring (exclusive).
  - Can be a numeric index.
  - Can be a negative number (e.g., `-1.0` to indicate the end of the string).
  - Can be a nested `indexOf` transform.
- **`beginOffset`**: (Optional) An integer offset added to the `begin` index. Useful for skipping characters *after* a found starting point (e.g., skipping a comma and space).
- **`endOffset`**: (Optional) An integer offset added to the `end` index. Useful for skipping characters *before* a found ending point.

### How it Works (Step-by-Step)

1. **Identify Input:** The transform first determines the source string it will operate on. This can be a hardcoded string or a dynamic value from an account attribute.
2. **Calculate Start Position:** It calculates the beginning index of the desired substring. This can be a fixed number, or it can dynamically find the position of a specific character or sequence of characters using an `indexOf` operation.
3. **Apply Start Offset:** If `beginOffset` is provided, it adjusts the calculated start position by adding the offset.
4. **Calculate End Position:** It calculates the ending index of the desired substring. This can be a fixed number, or it can dynamically find a position, or it can be set to the end of the input string using a negative value like `-1.0`.
5. **Apply End Offset:** If `endOffset` is provided, it adjusts the calculated end position by adding the offset.
6. **Extract:** Finally, it extracts all characters from the (adjusted) start position up to, but not including, the (adjusted) end position.

### Examples

#### Example 1: Fixed Position Substring

This example extracts a substring from a known, fixed range within an input string.

**Transform Configuration (from `transform4.json`):**

```json
{
    "id": "484e717d-2841-4bab-9bbf-6f48d8096965",
    "name": "Substring Transform example",
    "type": "substring",
    "attributes": {
        "begin": 1,
        "end": 3,
        "beginOffset": 1,
        "endOffset": 2
    }
}
```

**Step-by-Step with Example Input:**

Let's assume the input string is `"abcdefg"`.

1. **Input:** `"abcdefg"`
2. **`begin: 1`**: The starting index is 1 (the second character, 'b').
3. **`end: 3`**: The ending index is 3 (meaning characters up to, but not including, index 3). So, characters from index 1 and 2 ('b', 'c').
4. **`beginOffset: 1`**: Adds 1 to the `begin` index. Original `begin` was 1, now it's `1 + 1 = 2`. So, the substring *now* starts at index 2 ('c').
5. **`endOffset: 2`**: Adds 2 to the `end` index. Original `end` was 3, now it's `3 + 2 = 5`. So, the substring *now* ends before index 5 (meaning characters up to index 4).
6. **Extract:** From index 2 ('c') up to index 4 ('e').

**Input:** `"abcdefg"`
**Output:** `"cde"`

#### Example 2: Extracting Data After a Delimiter

This example shows how to dynamically extract a part of a string by finding a specific character as a delimiter.

**Transform Configuration (from `transform7.json`):**

```json
{
    "id": "484e717d-2841-4bab-9bbf-6f48d8096965",
    "name": "Calculate Partners State",
    "type": "substring",
    "attributes": {
        "input": {
            "attributes": {
                "attributeName": "Location",
                "sourceName": "Partner Accounts"
            },
            "type": "accountAttribute"
        },
        "end": -1.0,
        "begin": {
            "attributes": {
                "substring": ","
            },
            "type": "indexOf"
        },
        "beginOffset": 2.0
    },
    "internal": false
}
```

**Step-by-Step with Example Input:**

Let's assume the `Location` attribute for a user is `"123 Main St, Anytown, CA 90210"`.

1. **Input:** Value of `accountAttribute` named `Location` from `Partner Accounts` source: `"123 Main St, Anytown, CA 90210"`.
2. **Calculate Start Position (`begin` with `indexOf`):** The `indexOf` transform searches for the first occurrence of `","` (comma).
    - The comma is found at index 13. So, `begin` is 13.
3. **Apply Start Offset (`beginOffset: 2.0`):** Adds 2 to the `begin` index.
    - New `begin` is `13 + 2 = 15`. This skips the comma and the space immediately after it.
4. **Calculate End Position (`end: -1.0`):** `-1.0` indicates the substring should extend to the very end of the input string.
5. **Extract:** From index 15 ('A' in "Anytown") to the end of the string.

**Input (`Location` attribute):** `"123 Main St, Anytown, CA 90210"`
**Output:** `"Anytown, CA 90210"`

---

## 2. Lookup Transform

The `lookup` transform maps an input value to a corresponding output value based on a predefined table. It's ideal for translating codes or standardizing values.

### Purpose

To translate one set of values into another, such as converting country codes to timezones, or internal IDs to display names.

### Attributes

- **`table`**: A JSON object (key-value map) where keys are the input values and values are the corresponding output values.
- **`default`**: (Optional) A value to return if the input value is not found as a key in the `table`. If `default` is not specified and the input is not found, the transform typically returns `null` or the original input, depending on context.

### How it Works (Step-by-Step)

1. **Get Input Value:** The transform receives an input value (e.g., a country code).
2. **Search Table:** It checks if the input value exists as a key in its internal `table`.
3. **Return Mapped Value:**
    - If the input value is found as a key, the transform returns the associated value from the table.
    - If the input value is *not* found and a `default` attribute is specified, the transform returns the `default` value.
    - If the input value is *not* found and no `default` attribute is specified, the behavior can vary (often returns `null` or the original input).

### Examples

#### Example 1: Country Code to Timezone Mapping

This example translates a country code (like "EN-US") into a timezone string (like "CST").

**Transform Configuration (from `transform6.json`):**

```json
{
        "id": "b23788a0-41a2-453b-89ae-0d670fa0cb6a",
        "name": "Country Code To Timezone",
        "type": "lookup",
        "attributes": {
            "table": {
                "EN-US": "CST",
                "ES-MX": "CST",
                "EN-GB": "GMT",
                "default": "GMT"
            }
        },
        "internal": false
    }
```

**Step-by-Step with Example Inputs:**

1. **Input:** Assume the input value to this transform is `"EN-US"`.
    - The transform looks for `"EN-US"` in the `table`. It finds it.
    - **Output:** `"CST"`

2. **Input:** Assume the input value is `"EN-GB"`.
    - The transform looks for `"EN-GB"` in the `table`. It finds it.
    - **Output:** `"GMT"`

3. **Input:** Assume the input value is `"FR-FR"` (France).
    - The transform looks for `"FR-FR"` in the `table`. It does not find it.
    - Since a `"default": "GMT"` is specified, it returns the default.
    - **Output:** `"GMT"`

---

## 3. ISO3166 Transform

The `iso3166` transform is used to standardize country codes according to the ISO 3166 standard. This is crucial for applications that require consistent country representation.

### Purpose

To ensure that country codes are formatted correctly and consistently, typically converting between different ISO 3166 formats (e.g., Alpha-2, Alpha-3, Numeric) or validating an input against the standard.

### Attributes

- **`attributes: null`**: In the provided context, the `attributes` field is `null`, suggesting this transform operates on a standard input (which would be a country code) and applies a default or pre-configured ISO 3166 formatting logic without requiring additional configuration parameters for the transform itself. The specific target format (e.g., Alpha-2, Alpha-3) would be inherent to the transform's implementation.

### How it Works (Step-by-Step)

1. **Get Input Country Code:** The transform receives a country code as its input. This could be from an account attribute, a static value, or the output of another transform.
2. **Process and Standardize:** It processes the input country code against the ISO 3166 standard. This typically involves:
    - Validating if the input is a recognized country.
    - Converting the input to a specific ISO 3166 format (e.g., always outputting the Alpha-2 code like "US" or "GB").
3. **Return Standardized Code:** It returns the standardized country code. If the input is invalid or cannot be standardized, it might return `null` or an error, depending on its internal logic.

### Examples

#### Example 1: Standardizing Country Codes

While the JSON provided for `ISO3166 Country Format` doesn't show specific input/output examples within its `attributes`, its purpose is clear.

**Transform Configuration (from `transform5.json`):**

```json
{
        "id": "2b5191bb-051f-4edf-8283-3962b4a0f7a5",
        "name": "ISO3166 Country Format",
        "type": "iso3166",
        "attributes": null,
        "internal": true
    }
```

**Step-by-Step with Example Inputs:**

Let's assume this transform is used to ensure all country codes are in the ISO 3166 Alpha-2 format.

1. **Input:** Assume the input value to this transform is `"United States"`.
    - The transform recognizes "United States" and converts it to its Alpha-2 code.
    - **Output:** `"US"`

2. **Input:** Assume the input value is `"GBR"`.
    - The transform recognizes "GBR" (Alpha-3 for United Kingdom) and converts it to its Alpha-2 code.
    - **Output:** `"GB"`

3. **Input:** Assume the input value is `"Deutschland"`.
    - The transform recognizes "Deutschland" and converts it to its Alpha-2 code.
    - **Output:** `"DE"`

4. **Input:** Assume the input value is `"Atlantis"`.
    - The transform does not recognize "Atlantis" as a valid country.
    - **Output:** `null` (or an empty string, or an error, depending on implementation).

Here's the developer documentation and input/output example for the "Within25days" transform, structured as requested:

---

## 4. Within25days

### Purpose

The "Within25days" transform is designed to evaluate whether a specified `end_date` attribute, retrieved from an "Example CSV" account source, falls within 25 days from the current date. It returns a boolean string (`"true"` or `"false"`) indicating if the `end_date` is less than or equal to the current date plus 25 days.

### Attributes

The transform's configuration is defined by the following attributes:

- **`attributes`**:
  - **`Within25Days`**: This is an internal attribute that encapsulates the core logic of the transform, specifically a date comparison.
    - **`type`**: `dateCompare` - Indicates that this attribute performs a comparison between two dates.
    - **`attributes`**:
      - **`firstDate`**: Defines the left-hand side of the date comparison.
        - **`type`**: `dateFormat` - Formats a date from a specified input.
        - **`attributes`**:
          - **`input`**: Specifies the source of the date to be formatted.
            - **`type`**: `accountAttribute` - Retrieves a value from an account attribute.
            - **`attributes`**:
              - **`sourceName`**: `"Example CSV"` - The name of the source system (e.g., a connector) from which the attribute is fetched.
              - **`attributeName`**: `"end_date"` - The specific attribute name on the account to be used.
          - **`inputFormat`**: `"yyyy-MM-dd"` - The expected format of the `end_date` attribute from the source.
          - **`outputFormat`**: `"ISO8601"` - The standard format to which the `end_date` will be converted for accurate comparison.
      - **`secondDate`**: Defines the right-hand side of the date comparison.
        - **`type`**: `dateFormat` - Formats a date from a specified input.
        - **`attributes`**:
          - **`input`**: Specifies the source of the date to be formatted.
            - **`type`**: `dateMath` - Performs date arithmetic.
            - **`attributes`**:
              - **`expression`**: `"now+25d"` - An expression that calculates a date 25 days from the current timestamp (`now`).
          - **`inputFormat`**: `"yyyy-MM-dd'T'HH:mm"` - The expected format of the date string produced by the `dateMath` operation.
          - **`outputFormat`**: `"ISO8601"` - The standard format to which the calculated date will be converted for comparison.
      - **`operator`**: `"lte"` - The comparison operator, meaning "Less Than or Equal To" (`firstDate <= secondDate`).
      - **`positiveCondition`**: `"true"` - The string value returned if the `dateCompare` condition evaluates to true.
      - **`negativeCondition`**: `"false"` - The string value returned if the `dateCompare` condition evaluates to false.
  - **`value`**: `"$Within25Days"` - This defines the final output of the entire `static` transform, which is the boolean string result from the `Within25Days` comparison.

### How it Works (Step-by-Step)

The "Within25days" transform operates through the following steps:

1. **Retrieve `end_date`**: It first accesses an account attribute named `end_date` from the `Example CSV` source.
2. **Format `end_date`**: The retrieved `end_date` (expected in `yyyy-MM-dd` format) is then converted into `ISO8601` format. This becomes the `firstDate` for comparison.
3. **Calculate Target Date**: Concurrently, it calculates a target date by taking the current timestamp (`now`) and adding 25 days to it (`now+25d`).
4. **Format Target Date**: This calculated date (expected in `yyyy-MM-dd'T'HH:mm` format from `dateMath`) is also converted into `ISO8601` format. This becomes the `secondDate` for comparison.
5. **Perform Date Comparison**: A comparison is made between the `firstDate` (formatted `end_date`) and the `secondDate` (formatted `now+25d`). The `operator` is "less than or equal to" (`lte`).
6. **Return Result**:
    - If `firstDate <= secondDate` is true, the transform returns the string `"true"`.
    - If `firstDate <= secondDate` is false, the transform returns the string `"false"`.

### Examples

**Contextual Input:**
Assume the current date and time (the `now` reference point) is **`2025-10-10T10:00:00Z`**.

#### Example 1: `end_date` is within 25 days from now

- **Account Attribute Input (`Example CSV` -> `end_date`):** `"2025-10-25"`
- **Internal Processing:**
  - `firstDate` (from `end_date`): `2025-10-25T00:00:00Z`
  - `secondDate` (`now+25d` from `2025-10-10T10:00:00Z`): `2025-11-04T10:00:00Z`
  - Comparison: `2025-10-25T00:00:00Z <= 2025-11-04T10:00:00Z` (This evaluates to `true`)
- **Transform Output:** `"true"`

#### Example 2: `end_date` is exactly 25 days from now (considering the date part)

- **Account Attribute Input (`Example CSV` -> `end_date`):** `"2025-11-04"`
- **Internal Processing:**
  - `firstDate` (from `end_date`): `2025-11-04T00:00:00Z`
  - `secondDate` (`now+25d` from `2025-10-10T10:00:00Z`): `2025-11-04T10:00:00Z`
  - Comparison: `2025-11-04T00:00:00Z <= 2025-11-04T10:00:00Z` (This evaluates to `true`)
- **Transform Output:** `"true"`

#### Example 3: `end_date` is beyond 25 days from now

- **Account Attribute Input (`Example CSV` -> `end_date`):** `"2025-11-15"`
- **Internal Processing:**
  - `firstDate` (from `end_date`): `2025-11-15T00:00:00Z`
  - `secondDate` (`now+25d` from `2025-10-10T10:00:00Z`): `2025-11-04T10:00:00Z`
  - Comparison: `2025-11-15T00:00:00Z <= 2025-11-04T10:00:00Z` (This evaluates to `false`)
- **Transform Output:** `"false"`
