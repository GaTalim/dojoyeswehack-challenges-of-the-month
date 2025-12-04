# CTF Write-up: Halloween Special - Ghost Whisper

## Challenge Overview

**Challenge Name:** Ghost Whisper  
**Category:** Web Security / Command Injection  
**Difficulty:** Easy-Medium  
**URL:** https://dojo-yeswehack.com/challenge-of-the-month/dojo-46  
**Key Concept:** Unicode Normalization Bypass

## Challenge Code

```python
import os, unicodedata
from urllib.parse import unquote
from jinja2 import Environment, FileSystemLoader

template = Environment(
    autoescape=True,
    loader=FileSystemLoader('/tmp/templates'),
).get_template('index.html')

os.chdir('/tmp')

def main():
    whisperMsg = unquote("INPUT")

    # Normalize dangerous characters
    whisperMsg = unicodedata.normalize("NFKC", whisperMsg.replace("'", "_"))

    # Run a command and capture its output
    with os.popen(f"echo -n '{whisperMsg}' | hexdump") as stream:
        hextext = f"{stream.read()} | {whisperMsg}"
        print(template.render(msg=whisperMsg, hextext=hextext))

main()
```

## Initial Analysis

### Step 1: Identifying the Vulnerability

The first thing that catches the eye is the user input being directly inserted into a shell command:

```python
os.popen(f"echo -n '{whisperMsg}' | hexdump")
```

This is a classic **command injection** vulnerability. The input is wrapped in single quotes within the shell command, which means we need to escape those quotes to break out and execute arbitrary commands.

### Step 2: The First Obstacle

The code attempts to prevent command injection by replacing single quotes with underscores:

```python
whisperMsg = whisperMsg.replace("'", "_")
```

In a shell environment, everything inside single quotes `'...'` is treated as a literal string, so no commands can execute. To inject commands, we need to:
1. Close the opening single quote
2. Inject our command
3. Open a new single quote to maintain syntax

However, the replacement of `'` with `_` prevents this straightforward approach.

### Step 3: Understanding the Input Processing

The input goes through several transformations:

1. **URL Decoding**: `unquote("INPUT")` - Standard URL decoding, nothing exploitable here
2. **Quote Replacement**: `replace("'", "_")` - Blocks direct single quote injection
3. **Unicode Normalization**: `unicodedata.normalize("NFKC", ...)` - This is where the magic happens!

## The Key Insight: Unicode Normalization

### What is NFKC Normalization?

According to the [Python documentation](https://docs.python.org/3/library/unicodedata.html), `unicodedata.normalize("NFKC", ...)` performs **Compatibility Composition** normalization. This form is used to compare Unicode characters and determine if they are equivalent.

The key insight: **Different Unicode representations of the same character normalize to the same character**.

For example:
- `á` can be represented as `a + ´` (combining character) or as a single precomposed character
- These are different byte sequences but represent the same character
- NFKC normalization converts them to a canonical form

### The Exploitation Vector

The critical observation: **Can we find a Unicode character that looks like a single quote but isn't actually `'` (U+0027), and that normalizes to a regular single quote?**

Yes! The answer is **U+FF07** - the **Fullwidth Apostrophe** (`＇`).

When `unicodedata.normalize("NFKC", ...)` processes U+FF07, it normalizes it to a regular single quote `'` (U+0027). This happens **after** the `replace("'", "_")` call, meaning:

1. Our payload contains `＇` (U+FF07) - not a regular single quote
2. The `replace("'", "_")` doesn't match it, so it passes through unchanged
3. `normalize("NFKC", ...)` converts `＇` to `'`
4. The shell command now receives a regular single quote, allowing us to escape!

## The Solution

### Payload Construction

To exploit this, we need to:
1. Use `＇` (U+FF07) instead of `'` in our payload
2. Break out of the echo command
3. Execute arbitrary commands

Example payload:
```
＇; env; echo ＇
```

When URL encoded and processed:
- Input: `＇; env; echo ＇`
- After `replace("'", "_")`: `＇; env; echo ＇` (unchanged, no regular quotes)
- After `normalize("NFKC", ...)`: `'; env; echo '` (normalized to regular quotes)
- Final command: `echo -n '; env; echo ' | hexdump`

This successfully breaks out of the single quotes and executes `env`.

### URL Encoding Consideration

One challenge: when passing the payload via URL encoding, we need to ensure the Unicode character is properly encoded. The fullwidth apostrophe `＇` needs to be URL encoded as `%EF%BC%87`.

## Complete Exploit

**Payload:** `test＇;env;echo＇test`

**URL Encoded:** `test%EF%BC%87%3Benv%3Becho%EF%BC%87test`

**Processing Flow:**
1. `unquote()` decodes to: `test＇;env;echo＇test`
2. `replace("'", "_")` does nothing (no regular quotes)
3. `normalize("NFKC", ...)` converts to: `test';env;echo'test`
4. Shell executes: `echo -n 'test';env;echo'test' | hexdump`
5. Command injection successful!

## References

- [Python unicodedata documentation](https://docs.python.org/3/library/unicodedata.html)
- [Unicode Normalization Forms](https://unicode.org/reports/tr15/)
