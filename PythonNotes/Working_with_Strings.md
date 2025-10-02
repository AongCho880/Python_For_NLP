# Working with Strings & Regex in Python (NLP-friendly)

---

## Table of Contents

1. [Text vs Bytes & Safe File I/O](#1-text-vs-bytes--safe-file-io)
2. [Core String Operations](#2-core-string-operations)
3. [Whitespace & Line Handling](#3-whitespace--line-handling)
4. [Case Handling: `lower()` vs `casefold()`](#4-case-handling-lower-vs-casefold)
5. [Unicode Normalization](#5-unicode-normalization)
6. [Punctuation, Quotes, Emojis](#6-punctuation-quotes-emojis)
7. [Quick Token-like Splits](#7-quick-token-like-splits)
8. [Regex Extraction: URLs, Mentions, Hashtags](#8-regex-extraction-urls-mentions-hashtags)
9. [Search/Replace: `re.sub()` & `str.translate()`](#9-searchreplace-resub--strtranslate)
10. [Reusable Cleaner Function](#10-reusable-cleaner-function)
11. [Regex Syntax Quick Reference](#11-regex-syntax-quick-reference)
12. [Apply-It-Now Recipes](#12-apply-it-now-recipes)

---

## 1) Text vs Bytes & Safe File I/O

**What:** `str` is Unicode text; `bytes` is raw data. Use **UTF-8** and be explicit.

**Critical syntax**

* `text.encode("utf-8")` → `bytes`
* `b.decode("utf-8")` → `str`
* `open(..., encoding="utf-8")` to avoid mojibake

```python
text = "café ☕"
b = text.encode("utf-8")        # str -> bytes
back = b.decode("utf-8")        # bytes -> str

with open("tweet.txt", "w", encoding="utf-8", newline="\n") as f:
    f.write(text)
with open("tweet.txt", "r", encoding="utf-8") as f:
    loaded = f.read()
```

---

## 2) Core String Operations

**What:** Strings are immutable; use slicing/indexing/membership.

**Critical syntax**

* Index & slice: `s[i]`, `s[a:b]`
* Membership: `"x" in s`
* Length/codes: `len(s)`, `ord("A")`, `chr(65)`

```python
s = "hi👋!"
s[2]          # 👋
s[:2]         # 'hi'
"👋" in s     # True
len(s)        # 4
ord("A"), chr(65)  # (65, 'A')
```

---

## 3) Whitespace & Line Handling

**What:** Normalize irregular spacing; split lines safely.

**Critical syntax**

* Collapse whitespace: `" ".join(text.split())`
* Split lines: `text.splitlines()`

```python
review = "  Great  value! \nFast\tshipping "
clean = " ".join(review.split())           # 'Great value! Fast shipping'
"hi\nthere\r\n🙂".splitlines()             # ['hi', 'there', '🙂']
```

---

## 4) Case Handling: `lower()` vs `casefold()`

**What:** `casefold()` is stronger and Unicode-aware for case-insensitive work.

```python
q = "Straße"
q.lower()          # 'straße'
q.casefold()       # 'strasse'
"STRASSE".casefold() == q.casefold()  # True
```

---

## 5) Unicode Normalization

**What:** Visually same strings can differ in bytes. Normalize first.

**Critical syntax**

* NFC: compose accents; safe default for comparison
* NFKC: also normalizes compatibility forms (e.g., full-width digits)

```python
import unicodedata as ud

s1 = "naïve"
s2 = "nai\u0308ve"                          # i + combining diaeresis
ud.normalize("NFC", s1) == ud.normalize("NFC", s2)   # True
ud.normalize("NFKC", "１/２")               # '1/2'
```

---

## 6) Punctuation, Quotes, Emojis

**What:** ASCII punctuation is limited; use Unicode categories for robust detection.

**Critical syntax**

* `string.punctuation` → ASCII only
* `unicodedata.category(c)` → e.g., `Pi`/`Pf` (quotes), `So` (emoji-like symbols)

```python
import string, unicodedata as ud

txt = '“Great!” 😂 #deal'
{c for c in txt if c in string.punctuation}     # {'#'}
[(c, ud.category(c)) for c in txt]              # categories
[c for c in txt if ud.category(c) == "So"]      # ['😂']
```

---

## 7) Quick Token-like Splits

**What:** Simple splitting and single-split partitioning.

**Critical syntax**

* `text.split()` → whitespace tokens (rough)
* `left, _, right = text.partition(":")` → split once

```python
tweet = "Deal: 2-for-1 pizza #yum #Dhaka"
tweet.split()             # rough tokens
left, _, right = tweet.partition(":")   # 'Deal', ':', ' 2-for-1 pizza #yum #Dhaka'
```

---

## 8) Regex Extraction: URLs, Mentions, Hashtags

**What:** Pull structured bits from text.

**Critical syntax**

* Raw strings: `r"..."`
* `\b` word boundary
* `\S+` = non-whitespace run (watch trailing punctuation)

```python
import re

tweet = "Check https://ex.am/ple @user 😀 #NLP"
urls     = re.findall(r'https?://\S+', tweet)
mentions = re.findall(r'@(\w+)', tweet)      # capture without '@'
tags     = re.findall(r'#(\w+)', tweet)

re.findall(r'\brefund\b', "Refund please. REFUND policy?", flags=re.I)
```

> **Tip:** If commas/`)` appear after URLs, use a tighter tail:
> `re.compile(r"https?://\S+?(?=[\s\]\)}]|$)")`

---

## 9) Search/Replace: `re.sub()` & `str.translate()`

**What:** Powerful find/replace via patterns; fast char-to-char mapping.

**Critical syntax**

* `re.sub(pattern, repl, text)`
* Replacement with groups: `r"\2 \1"`
* Make translation table: `str.maketrans({...})`

```python
import re

re.sub(r"[^\w\s]", "", "Hi! #NLP @2025")          # remove punctuation
re.sub(r"(.)\1{2,}", r"\1\1", "soooo coooool!!!") # cap repeats (3+ → 2)
re.sub(r"\s+", " ", "a \t b\nc").strip()          # normalize spaces
re.sub(r"(\w+),\s*(\w+)", r"\2 \1", "Doe, Jane")  # 'Jane Doe'

# Smart quotes/dashes → ASCII
SMART_MAP = str.maketrans({"“": '"', "”": '"', "‘": "'", "’": "'", "—": "-", "–": "-"})
"“quote”—‘test’".translate(SMART_MAP)             # '"quote"-'test'
```

---

## 10) Reusable Cleaner Function

**What:** Modular pipeline with toggles for common chat/tweet/review cleaning.

```python
import re, unicodedata as ud

SMART_MAP = str.maketrans({"“": '"', "”": '"', "‘": "'", "’": "'", "—": "-", "–": "-"})
DIGIT_MAP = str.maketrans({ "০":"0","১":"1","২":"2","৩":"3","৪":"4","৫":"5","৬":"6","৭":"7","৮":"8","৯":"9",
                            "٠":"0","١":"1","٢":"2","٣":"3","٤":"4","٥":"5","٦":"6","٧":"7","٨":"8","٩":"9" })
URL_RE   = re.compile(r'https?://\S+')
MENT_RE  = re.compile(r'@[\w_]+')
HASH_RE  = re.compile(r'#(\w+)')
REPEAT3  = re.compile(r'(.)\1{2,}')      # any char repeated 3+ times

def clean_text_simple(text: str,
                      keep_hashtags: bool = True,
                      keep_emojis: bool = True,
                      compress_repeats: bool = True,
                      casefolding: bool = True,
                      normalize_form: str = "NFC") -> str:
    t = ud.normalize(normalize_form, text).translate(SMART_MAP).translate(DIGIT_MAP)
    t = URL_RE.sub("", t)
    t = MENT_RE.sub("", t)
    if not keep_hashtags:
        t = HASH_RE.sub("", t)
    if compress_repeats:
        t = REPEAT3.sub(r"\1\1", t)      # keep two of any long run
    if not keep_emojis:
        t = "".join(ch for ch in t if ud.category(ch) != "So")
    t = " ".join(t.split()).strip()
    return t.casefold() if casefolding else t
```

---

## 11) Regex Syntax Quick Reference

**Character types**

* `\w` word char (Unicode letters/digits/underscore)
* `\d` digit (Unicode); `\s` whitespace; **`\S` non-whitespace**
* `.` any char except newline (add `re.DOTALL` to include newline)

**Quantifiers**

* `+` one or more; `*` zero or more; `?` zero or one
* `{m,n}` between *m* and *n* times (e.g., `{2,}` means 2+)

**Grouping & classes**

* `( ... )` capture **group** (use with `\1`, `\2`, …)
* `(?: ... )` non-capturing group
* `[ ... ]` character **class** (any one of these)
* `[^ ... ]` negated class (none of these)

**Anchors & boundaries**

* `^` start of string/line; `$` end of string/line
* `\b` word boundary

**Flags (common)**

* `re.I` ignore case (Unicode aware)
* `re.M` multiline (`^`/`$` match line starts/ends)
* `re.S` (DOTALL) `.` matches newline
* `re.ASCII` restrict `\w`, `\d`, `\s` to ASCII

**Important patterns from above**

* `https?://\S+` → URL until whitespace
* `@(\w+)` → capture username (no `@`)
* `#(\w+)` → capture hashtag text
* `(.)\1{2,}` → **repeat detection** (3+ of same char)

---

## 12) Apply-It-Now Recipes

**A. Minimal review cleaner**

```python
import re, unicodedata as ud
def clean_review(p):
    p = ud.normalize("NFC", p)
    p = re.sub(r"https?://\S+", "", p)        # strip URLs
    p = re.sub(r"(.)\1{2,}", r"\1\1", p)      # cap repeats (3+ -> 2)
    p = re.sub(r"[^\w\s]", " ", p)            # drop punctuation
    return re.sub(r"\s+", " ", p).strip().casefold()
```

**B. Compile once for speed**

```python
URL      = re.compile(r"https?://\S+")
REPEAT3  = re.compile(r"(.)\1{2,}")
NONWORD  = re.compile(r"[^\w\s]+")

def fast_clean(s):
    s = URL.sub("", s)
    s = REPEAT3.sub(r"\1\1", s)
    s = NONWORD.sub(" ", s)
    return re.sub(r"\s+", " ", s).strip().casefold()
```

**C. Keep hashtags & emojis, remove @mentions/URLs**

```python
def keep_hashtags_emojis(s):
    s = URL_RE.sub("", s)
    s = MENT_RE.sub("", s)
    s = " ".join(s.split())
    return s
```

---

### Tiny sanity checks

* Prefer `"".join(pieces)` over `+` in loops.
* Normalize (`NFC`) **before** regex ops when accents/combining marks may appear.
* If a URL gets a trailing `,` or `)`, tighten the tail as shown in §8.

---


## **A Full Code Example**
```
# Imports (with one-line descriptions):
import re                    # re: regular expressions for finding/removing patterns
import unicodedata as ud     # unicodedata: Unicode helpers (normalize text)
import string                # string: basic constants like ASCII punctuation

# Small, readable building blocks:

# 1) Fix “smart” quotes/dashes to plain ASCII so text is consistent
SMART_MAP = str.maketrans({
    "“": '"', "”": '"', "‘": "'", "’": "'", "—": "-", "–": "-"
})

# 2) Convert Bengali and Arabic-Indic digits to Western 0–9
DIGIT_MAP = str.maketrans({
    "০": "0","১": "1","২": "2","৩": "3","৪": "4","৫": "5","৬": "6","৭": "7","৮": "8","৯": "9",
    "٠": "0","١": "1","٢": "2","٣": "3","٤": "4","٥": "5","٦": "6","٧": "7","٨": "8","٩": "9",
})

# 3) Common patterns we’ll remove or read
URL_RE   = re.compile(r'https?://\S+')   # find http:// or https:// then non-space
MENT_RE  = re.compile(r'@[\w_]+')        # find @username (letters/digits/_)
HASH_RE  = re.compile(r'#(\w+)')         # capture hashtag text (letters/digits/_)
REPEAT3  = re.compile(r'(.)\1{2,}')      # find any char repeated 3+ times (e.g., "soooo")

def clean_text_simple(
    text: str,
    keep_hashtags: bool = True,   # True -> keep #topic; False -> remove them
    keep_emojis: bool = True,     # True -> keep emojis; False -> drop them
    compress_repeats: bool = True,# "soooo" -> "soo"
    casefolding: bool = True,     # better-than-lower() for case-insensitive text
    normalize_form: str = "NFC"   # make composed Unicode (recommended default)
) -> str:
    """
    A small, readable cleaner for tweets/chats/reviews.
    """

    # Step 0) Normalize to a consistent Unicode form; fix quotes/dashes; unify digits
    t = ud.normalize(normalize_form, text)      # make Unicode consistent
    t = t.translate(SMART_MAP)                  # smart quotes/dashes -> ASCII
    t = t.translate(DIGIT_MAP)                  # non-Western digits -> 0-9

    # Step 1) Remove URLs and @mentions (handles)
    t = URL_RE.sub("", t)
    t = MENT_RE.sub("", t)

    # Step 2) Keep or remove hashtags
    if not keep_hashtags:
        t = HASH_RE.sub("", t)                  # strip the whole #word

    # Step 3) Compress very long character repeats
    if compress_repeats:
        # Tricky syntax note:
        #   Pattern '(.)\1{2,}' means:
        #     (.)   -> remember one character as group 1
        #     \1    -> the same character again
        #     {2,}  -> at least 2 more times (so total 3+ in a row)
        #   Replacement r'\1\1' keeps just 2 of that character.
        t = REPEAT3.sub(r"\1\1", t)

    # Step 4) Optionally remove emojis
    if not keep_emojis:
        # Keep only characters that are NOT emoji-like symbols.
        # Many emojis fall under Unicode category "So" (Symbol, other).
        kept_chars = []
        for ch in t:
            cat = ud.category(ch)
            if cat == "So":
                continue          # drop emoji
            kept_chars.append(ch)
        t = "".join(kept_chars)

    # Step 5) Light punctuation control (very simple)
    # We keep useful punctuation: hash (if kept above), apostrophe, and hyphen.
    # Everything else is allowed to stay for now; we just tidy spaces next.
    # If you want to be stricter, you can remove more punctuation here.

    # Step 6) Collapse extra spaces and trim
    t = " ".join(t.split()).strip()

    # Step 7) Casefold at the end (stronger lowercase)
    if casefolding:
        t = t.casefold()

    return t

# --------- Examples ---------

print(clean_text_simple('Sooooo cooool!!! visit https://x.co @aa #fun 😂'))
# Expected -> 'soo coool #fun 😂'  (keeps hashtag and emoji, compresses repeats, removes URL/@)

print(clean_text_simple("ফেরত ১০০৳ চান?", keep_hashtags=False))
# Expected -> 'ফেরত 100৳ চান?'   (digits mapped to Western)

print(clean_text_simple("naïve naive nai\u0308ve"))
# Expected -> 'naïve naive naïve' (Unicode forms aligned by NFC)

print(clean_text_simple("Need REFUND!!! @me http://x #order", keep_emojis=False))
# Expected -> 'refund #order'

print(clean_text_simple("Soooo goooood 😂😂", keep_emojis=False))
# Expected -> 'soo goood'

```
