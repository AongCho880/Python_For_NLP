# **Working with Strings**

---

## 1) `str` vs `bytes`, UTF-8, safe file I/O

```python
# No imports needed here
text = "café ☕"
b = text.encode("utf-8")   # str -> bytes
back = b.decode("utf-8")   # bytes -> str
print(text)                # café ☕
print(type(b))             # <class 'bytes'>

# Safe file I/O: always set encoding
with open("tweet.txt", "w", encoding="utf-8", newline="\n") as f:
    f.write(text)
with open("tweet.txt", "r", encoding="utf-8") as f:
    loaded = f.read()
print(loaded)              # café ☕
```

**Explanation:** `str` is text; `bytes` is raw data. `encode/decode` converts between them. `encoding="utf-8"` avoids garbled characters.

---

## 2) Immutability, slicing, indexing, `in`, `len`, `ord/chr`

```python
# No imports needed here
s = "hi👋!"
print(s[2])      # 👋
print(s[:2])     # hi
print("👋" in s)  # True
print(len(s))    # 4
print(ord("A"))  # 65  (char -> code)
print(chr(65))   # A   (code -> char)
```

**Explanation:** Strings can’t change in place. Use slices/indices to read parts. `ord/chr` map between characters and their code points.

---

## 3) Whitespace & lines: `strip/split/splitlines`

```python
# No imports needed here
review = "  Great  value! \nFast\tshipping "
clean = " ".join(review.split())
print(clean)                     # Great value! Fast shipping
print("hi\nthere\r\n🙂".splitlines())  # ['hi', 'there', '🙂']
```

**Explanation:** `split()` breaks on any whitespace; joining with `" ".join(...)` collapses repeated spaces. `splitlines()` removes the newline characters.

**Tricky syntax note:** `" ".join(list)` builds a string from pieces efficiently.

---

## 4) Case handling: `lower()` vs `casefold()`

```python
# No imports needed here
q = "Straße"
print(q.lower())                 # straße
print(q.casefold())              # strasse
print("STRASSE".casefold() == q.casefold())  # True
```

**Explanation:** `casefold()` is a stronger lowercase for Unicode text. Prefer it for case-insensitive matching.

---

## 5) Unicode normalization

```python
import unicodedata as ud  # unicodedata: Unicode database (names, categories, normalization)

s1 = "naïve"
s2 = "nai\u0308ve"               # 'i' + combining diaeresis
print(s1 == s2)                  # False
print(ud.normalize("NFC", s1) == ud.normalize("NFC", s2))  # True
print(ud.normalize("NFKC", "１/２"))  # 1/2
```

**Explanation:** Visually identical strings can be encoded differently. Normalize with `NFC` (compose accents) or `NFKC` (also converts compatibility forms like full-width digits).

**Tricky syntax note:** `"\u0308"` inserts a combining accent by code point.

---

## 6) Punctuation, quotes, emojis

```python
import string                    # string: handy constants like ascii letters/punctuation
import unicodedata as ud         # unicodedata: Unicode categories like 'So', 'Pi', 'Pf'

txt = '“Great!” 😂 #deal'

print({c for c in txt if c in string.punctuation})  # {'#'}  (ASCII only)
print([(c, ud.category(c)) for c in txt])           # show general categories
emojis = [c for c in txt if ud.category(c) == "So"]
print(emojis)                                       # ['😂']
```

**Explanation:** `string.punctuation` only knows ASCII. Use `unicodedata.category` to detect smart quotes (`Pi`/`Pf`) and emojis (`So`).

**Tricky syntax notes:**

* `{c for c in txt ...}` is a set comprehension (unique items).
* `ud.category(c)` returns a two-letter code (e.g., `So` = Symbol, other).

---

## 7) Token-ish splits with strings (and limits)

```python
# No imports needed here
tweet = "Deal alert: 2-for-1 pizza #yum #Dhaka"
print(tweet.split())            # rough tokens

left, _, right = tweet.partition(":")
print(left)                     # Deal alert
print(right)                    #  2-for-1 pizza #yum #Dhaka
```

**Explanation:** `.split()` is quick and rough. `.partition(":")` splits once into `left`, the separator, and `right`.

**Tricky syntax note:** `_` is a common throwaway variable name (we don’t use the separator).

---

## 8) Regex for NLP: URLs, mentions, hashtags, word boundaries

```python
import re                        # re: regular expressions (search, match, capture)

tweet = "Check https://ex.am/ple @user 😀 #NLP"
urls     = re.findall(r'https?://\S+', tweet)   # http or https, then non-spaces
mentions = re.findall(r'@(\w+)', tweet)         # capture username without '@'
tags     = re.findall(r'#(\w+)', tweet)         # capture text after '#'

print(urls)     # ['https://ex.am/ple']
print(mentions) # ['user']
print(tags)     # ['NLP']

text = "Refund please. REFUND policy?"
print(re.findall(r'\brefund\b', text, flags=re.I))  # whole word, ignore case
```

**Explanation:** `findall` returns all matches. `\b` = word boundary; `re.I` = case-insensitive.

**Tricky syntax notes:**

* Raw string `r'...'` keeps backslashes literal in Python (nice for regex).
* `?` in `https?` means “optional s”.

---

## 9) Search/replace/translate (smart quotes → ASCII)

```python
# string is already imported above; no new imports here
smart = "“quote”—‘test’"
m = str.maketrans({"“": '"', "”": '"', "‘": "'", "’": "'", "—": "-"})
print(smart.translate(m))       # "quote"-'test'

text = "Sooo coool!!!"
print(text.replace("ooo", "oo"))  # Soo coool!!!
```

**Explanation:** `replace` swaps literal substrings. `translate` applies a character-to-character mapping fast.

**Tricky syntax note:** `str.maketrans({...})` builds the translation table that `translate()` uses.

---

## 10) Pattern extraction & counting (n-grams)

```python
from collections import Counter   # Counter: quick frequency counts of items

review = "I want a refund or return, refund now!"
tokens = [w.casefold().strip(".,!?:;") for w in review.split()]
bigrams = [" ".join(tokens[i:i+2]) for i in range(len(tokens)-1)]

print(Counter(tokens).most_common(2))            # top words
print([b for b in bigrams if "refund" in b])     # bigrams touching 'refund'
```

**Explanation:** Clean → split → count with `Counter`. N-grams are sliding windows over tokens.

**Tricky syntax notes:**

* List comprehension `[...]` builds lists compactly.
* `tokens[i:i+2]` is a 2-item slice; `" ".join(...)` merges words.

---

## 11) Reusable `clean_text()` with toggles

```python
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

**Explanation:** Normalizes Unicode and digits, removes URLs/handles, optional hashtag keep, compresses repeats, preserves chosen punctuation/emojis, trims spaces, and optionally case-folds.

**Tricky syntax notes:**

* `re.compile(...)` builds a reusable regex object (better when called often).
* `'(.)\1{2,}'` uses a **capturing group** `(.)` and a **backreference** `\1` to detect the same char repeated 3+ times; `r"\1\1"` replaces with two copies.
* `cat.startswith(("P","C"))` checks “any punctuation” or “control” categories.
* Generator expression `"".join(c for c in t if ...)` builds strings efficiently.

---

## 12) Performance & quick checks

```python
# No new imports needed
parts = ["ha"] * 10000
s = "".join(parts)              # fast build

raw = "Refund plz!!!  https://t.co/a @me"
print(len(raw), "->", len(clean_text(raw)))  # quick before/after check
```

**Explanation:** Prefer `"".join(pieces)` over `+` in loops. A simple length check helps spot over-aggressive cleaning.

---

### Final quick usage

```python
print(clean_text('Need REFUND!!! visit http://x @me #order'))  # 'refund #order'
print(clean_text('pizza-party—2-for-1'))                       # 'pizza-party-2-for-1'
```
