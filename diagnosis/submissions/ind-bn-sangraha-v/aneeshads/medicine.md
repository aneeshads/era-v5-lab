# Medicine sheet: ind-bn-sangraha-v (aneeshads)

Counts are from my label sheet: 161 documents read, 16 codes found. Eight codes are found more than five times, so
this sheet has eight medicines. A count is the number of times the code is written in the `diseases` column (a
document with three doubled vowel signs counts three times), followed by the number of documents.

**Harm test.** The gold set of Bengali (owned by the Data A and Data B teams) was not yet available to me. The
numbers below come from a stand-in: the 106 documents of this pack that I marked `clean`. On 106 documents, 1% is
1.06 documents, so one document changed is 0.9% (pass) and two are 1.9% (fail). The code of all eight medicines is
in one script; with the gold set, the test is `python3 medicines.py --gold <gold file>`. "Pack" gives what the
medicine does to all 500 documents of the pack.

| # | Code | Count (documents) | Action | Gold changed | Gold dropped | Share | Result | Pack changed / dropped |
|---|---|---|---|---|---|---|---|---|
| 1 | A3 | 63 (6) | fix, drop if still damaged | 0 | 0 | 0.0% | pass | 8 / 3 |
| 2 | A6 | 58 (11) | fix | 1 | 0 | 0.9% | pass | 38 / 0 |
| 3 | D3 | 51 (36) | none (candidate tested) | 14 | 0 | 13.2% | **fail** | 51 / 0 |
| 4 | G1 | 28 (10) | fix | 0 | 0 | 0.0% | pass | 13 / 0 |
| 5 | C2 | 27 (13) | fix | 0 | 0 | 0.0% | pass | 10 / 0 |
| 6 | C1 | 15 (15) | fix | 2 | 0 | 1.9% | **fail** | 23 / 0 |
| 7 | B4 | 8 (1) | fix | 0 | 0 | 0.0% | pass | 7 / 0 |
| 8 | C4 | 6 (6) | fix | 1 | 0 | 0.9% | pass | 9 / 0 |

Three of the stand-in hits (A6, C1, C4) are documents that I marked `clean` but that do have the disease. They are
errors in my labels, not harm by the medicine. See the harm test of each medicine.

The code below shares these definitions:

```python
import re
L = "[\u0985-\u09B9\u09BC-\u09D7\u09DC-\u09E3\u09F0\u09F1]"   # a Bengali letter or sign (not a digit)
SIGN = "[\u09BE-\u09CC\u09D7]"                                # a dependent vowel sign
```

## Medicine 1

1. **Disease:** A3, mojibake from a legacy Bijoy (ANSI font) conversion. A Latin-1 or symbol character stands
   inside a Bengali word in place of a conjunct or a vowel sign. Count 63, in 6 documents.
   - `bf521322a90491aa`, offset 155: `বিদ্রæপ`, U+00E6 `æ` in place of the vowel sign U (should be `বিদ্রুপ`).
   - `87590e5df7527a67`, offset 2000: `দ্বন্দ¦টা`, U+00A6 `¦` in place of `্ব` (should be `দ্বন্দ্বটা`).
   - `e4e559c63731c7fd`, offset 10147: `ঝা-াগুলোতেও`, U+002D `-` before the vowel sign AA U+09BE (should be
     `ঝাণ্ডাগুলোতেও`). This document has 49 such places.
2. **Medicine:** map the four leftovers whose meaning is certain in this pack back to Bengali: `¦` and `^` to `্ব`,
   `Ð` to `ণ্ড`, `æ` to `ু`. Only replace them when they stand between two Bengali letters or signs. Then count
   the leftovers that remain (a Latin-1 or symbol character inside a Bengali word, or any non-Bengali character
   before a vowel sign). If three or more remain, the text cannot be repaired: drop the document.
   ```python
   BIJOY = {"\u00A6": "\u09CD\u09AC", "^": "\u09CD\u09AC", "\u00D0": "\u09A3\u09CD\u09A1", "\u00E6": "\u09C1"}
   BIJOY_FIX = re.compile(f"(?<={L})[{''.join(map(re.escape, BIJOY))}](?={L})")
   LEFTOVER = re.compile(f"(?<={L})[\u00A1-\u00FF\u0153\u0304\u2022\u203A^](?={L})"
                         f"|[\u00A1-\u00FF\u0153\u0304\u2022\u203A^-](?={SIGN})")

   def a3(text):
       fixed = BIJOY_FIX.sub(lambda m: BIJOY[m.group()], text)
       return fixed, len(LEFTOVER.findall(fixed)) >= 3      # (new text, drop?)
   ```
3. **Action:** fix the text; drop the document if three or more leftovers remain after the fix.
4. **Harm test:** gold (stand-in, 106 documents): 0 changed, 0 dropped, 0.0%. **Pass.** Pack: 8 documents fixed
   (for example `জ¦ালানী` to `জ্বালানী` in `9419f580e774eb4d` and `তাÐবে` to `তাণ্ডবে` in
   `fc44be28d237bf38`) and 3 dropped (`e4e559c63731c7fd`, `fadb66be94959179`, `bf4fd3ccaf9e861d`).
5. **Side effects:**
   - The table is only right for Bijoy text. `æ` gives `ু`: `বিদ্রæপ` becomes `বিদ্রুপ`, a common spelling, while the
     dictionary spelling is `বিদ্রূপ`.
   - When Bijoy put the e-kar before the conjunct, the result has the wrong order: `বিশে^র` becomes `বিশে্বর`, not
     `বিশ্বের` (`5584556a2ea2dcfd`).
   - Leftovers outside the table (`›`, `Ñ`, `¡`, `•`, `œ`, the macron) stay. With fewer than three in a document,
     the damage stays in the text (`b5d3379fa1dd2757`). With three or more, the whole document is lost, also when
     the rest is good: in `bf4fd3ccaf9e861d`, `•` stands for `ল` three times.
   - A3 between digits is not seen: `৩°৫` in `2856497430a08269`.
   - A real `^` or `-` between Bengali letters, for example a suffix written `-ার` in a grammar text, would count as
     a leftover.

## Medicine 2

1. **Disease:** A6, wrong script. The Assamese letter RA WITH MIDDLE DIAGONAL U+09F0 `ৰ` is used in Bengali words
   in place of the Bengali RA U+09B0 `র`, mostly in old texts that were scanned and read by OCR. Count 58, in 11
   documents.
   - `b3f021daf03eebe5`, offset 21: `ধ্ৰুং` (09A7 09CD **09F0** 09C1 0982). The document has 24 of them.
   - `7bd37b13e5ceedf7`, offset 37: `পৰ্যায়` (09AA **09F0** 09CD 09AF …).
   - `87c77e18093a596b`, offset 1933: `প্ৰকাশ` (09AA 09CD **09F0** 0995 09BE 09B6).
2. **Medicine:** in a document where `র` is more common than `ৰ` (so the document is Bengali, not Assamese), replace
   every `ৰ` with `র`.
   ```python
   def a6(text):
       if text.count("\u09F0") and text.count("\u09F0") < text.count("\u09B0"):
           return text.replace("\u09F0", "\u09B0"), False
       return text, False
   ```
3. **Action:** fix the text.
4. **Harm test:** gold (stand-in): 1 changed, 0 dropped, 0.9%. **Pass.** The document changed is `80199b73dc4150b0`.
   It has `সৰ্ব্বসিদ্ধি` at offset 104, five `ৰ` in all, which I missed when I marked it `clean`. It is a true case of
   A6. Pack: 38 documents changed. In every one of them `র` is far more common than `ৰ`.
5. **Side effects:**
   - An Assamese name or quotation inside a Bengali document would be respelled in Bengali.
   - The Assamese WA U+09F1 `ৱ` (in 3 documents of the pack) is not handled.
   - An Assamese document (more `ৰ` than `র`) is left alone: it is D1, a wrong language label, and needs another
     medicine.
   - My other A6 case, the visarga `ঃ` used as a colon between numbers (`৭ঃ৪ঃ১১` in `0ca18aa03db2ef77`), is not
     treated.

## Medicine 3

1. **Disease:** D3, mixed language. Count 51, in 36 documents. Most of them are `clean` documents, because DISEASES.md
   says to label D3 but not judge it.
   - `16838a9d467d9521`, offset 6: `করোনা ( Corona) মুক্ত ভারত ( India)`, an English gloss in brackets after the
     Bengali word.
   - `abff62cd526be189`, offset 35: `থ্রি-স্টার`, an English word written in Bengali letters.
   - `eb3dc547426a3951`, offset 1432: `This is the wound of love.`, an English sentence in Bengali prose.
2. **Medicine:** none. The candidate I tested removes an English gloss in brackets that follows a Bengali word:
   ```python
   GLOSS = re.compile(r"\s?\(\s*[A-Za-z][A-Za-z0-9 .,&'’-]*\)")

   def d3(text):
       return GLOSS.sub("", text), False
   ```
3. **Action:** keep the documents as they are. The candidate's action would be to fix the text.
4. **Harm test:** for the candidate, gold (stand-in): 14 changed, 0 dropped, 13.2%. **Fail.** The English gloss is
   normal style in Bengali news: `রোহিত শর্মা (Rohit Sharma)`, `বিজেপি (BJP)`. A text rule cannot tell good mixing
   from bad, so I propose no medicine for D3.
5. **Side effects:**
   - The candidate removes good text: names and terms in English that help a model link the two scripts.
   - It still misses most D3: English words in Bengali letters (`থ্রি-স্টার`, `ক্যারিয়ার`) and English sentences
     without brackets. It treats only 16 of my 36 D3 documents.

## Medicine 4

1. **Disease:** G1, cut at the vowel sign. The commonest form in this pack is a vowel sign written twice, which no
   Bengali word has. Count 28, in 10 documents.
   - `b5d3379fa1dd2757`, offset 1918: `মাহমুদুুুর`, the vowel sign U U+09C1 three times (should be `মাহমুদুর`).
   - `489141c194e52d26`, offset 574: `চাালাচ্ছ`, the vowel sign AA U+09BE twice (should be `চালাচ্ছ`).
   - `63a1a908974b93be`, offset 2608: `শত্রুুমক্ত`, U+09C1 twice.
2. **Medicine:** when the same vowel sign or virama is written two or more times in a row, keep one.
   ```python
   DOUBLED = re.compile("([\u09BE-\u09CD\u09D7])\\1+")

   def g1(text):
       return DOUBLED.sub(r"\1", text), False
   ```
3. **Action:** fix the text.
4. **Harm test:** gold (stand-in): 0 changed, 0 dropped, 0.0%. **Pass.** Pack: 13 documents changed, for example
   `ভাঙচুুর` to `ভাঙচুর` three times in `6a3345ed5d8e4c08`, and `বাংলাাদেশ` to `বাংলাদেশ` in `c03dbc4ed2f2858d`.
5. **Side effects:**
   - The other forms of G1 are not fixed, because the right repair is not certain:
     - a vowel sign after a space (`ভরে েগলে` for `ভরে গেলে`)
     - a vowel sign after an independent vowel (`অা`)
     - two different vowel signs in a row.

     So 4 of my 10 G1 documents are not treated (`c620feba93dd0986`, `42f394b1b2292a07`, `9edb9f45b9686a66`,
     `4fd6a400467e4b0b`).
   - Expressive lengthening in informal writing (`নাাা`) is shortened.
   - Inside Bijoy-damaged text (A3) the fix changes one kind of garbage into another (`িি.িৎধহ` in
     `5584556a2ea2dcfd`).

## Medicine 5

1. **Disease:** C2, repeated lines. Count 27, in 13 documents.
   - `b0d8affd2329469d`, offset 219: line 2 (218 characters, `ভোলায় আঞ্চলিক এমআরপি পাসপোর্ট অফিসের সেবা …`) is line 1
     again.
   - `1da19ac48860fb47`, offset 138: line 2 (`লা লিগায় জয় পেয়েছে স্প্যানিশ জায়ান্ট রিয়াল মাদ্রিদ …`) is line 1 again.
   - `4fd6a400467e4b0b`, offset 965: lines 24 to 31 (`আঁচলের একটু ছোঁয়া, রুমালের ঘ্রানে শিহরন!` …) are lines
     2 to 9 again.
2. **Medicine:** remove a line of 20 or more characters (spaces at the ends ignored) that already appeared earlier
   in the same document. Keep the first copy.
   ```python
   def c2(text, min_len=20):
       seen, out = set(), []
       for line in text.split("\n"):
           key = line.strip()
           if len(key) >= min_len and key in seen:
               continue
           seen.add(key)
           out.append(line)
       return "\n".join(out), False
   ```
3. **Action:** fix the text.
4. **Harm test:** gold (stand-in): 0 changed, 0 dropped, 0.0%. **Pass.** Pack: 10 documents changed.
5. **Side effects:**
   - A repeat inside one line, the same sentence twice in a paragraph, is not seen: 8 of my 13 C2 documents are not
     treated.
   - Lines that repeat on purpose are removed:
     - the refrain of a song or poem of 20 or more characters
     - a line of a table or scorecard (`ইফতেখার আহমেদ ১৬ (১৩)` in `7acc8a39bbf7eb53`)
     - a question asked again as a heading (`গাছের পাতা সবুজ হয় কেন?` in `d9fb73294bcea228`).

     Short refrains, such as `জয় জয় জয়` in the hymn `80199b73dc4150b0`, are kept by the 20-character limit.

## Medicine 6

1. **Disease:** C1, boilerplate of the news sites in this pack. Count 15, in 15 documents.
   - `30bca5e624fe83fc`, offset 534: `নিউজ১৮ বাংলায় সবার আগে পড়ুন ব্রেকিং নিউজ। …`, the News18 footer.
   - `14c39215a007bbe9`, offset 1162: `… অনুমতি ছাড়া ব্যবহার বেআইনি।`, a copyright footer.
   - `0f79a07b7f39f669`, offset 1321: `(দ্য রিপোর্ট/এসকে/নূরু/এপি/এনডিএস/ডিসেম্বর ১৩, ২০১৩)`, the agency slug.
2. **Medicine:**
   - Remove a line that is a footer, share prompt or breadcrumb: a copyright line (`©`, `অনুমতি ছাড়া ব্যবহার বেআইনি`,
     `সর্বস্বত্ব সংরক্ষিত`, `All rights reserved`), the News18 footer, `এই পোস্টটি শেয়ার করতে চাইলে`, `Home/…`, or
     `Get the latest world news`.
   - Remove the agency slug `(দ্য রিপোর্ট/…)` at the end of a paragraph.
   ```python
   BOILER_LINE = re.compile(r"অনুমতি ছাড়া ব্যবহার বেআইনি|সর্বস্বত্ব সংরক্ষিত|All rights reserved|^\s*©"
                            r"|সবার আগে পড়ুন ব্রেকিং নিউজ|এই পোস্টটি শেয়ার করতে চাইলে|^\s*Home/"
                            r"|^\s*Get the latest world news")
   SLUG = re.compile(r"\s*\((?:দ্য রিপোর্ট|বিডিনিউজ|বাসস|ইউএনবি)/[^()\n]*\)")

   def c1(text):
       lines = [ln for ln in text.split("\n") if not BOILER_LINE.search(ln)]
       return SLUG.sub("", "\n".join(lines)), False
   ```
3. **Action:** fix the text.
4. **Harm test:** gold (stand-in): 2 changed, 0 dropped, 1.9%. **Fail on the stand-in.** Both documents do have the
   boilerplate:
   - `017b4f09daad47e4` has the News18 footer. I marked it `clean` and missed it.
   - `c5f15f32bee8a1be` has the copyright footer. I marked it `clean` but also wrote C1 in its `diseases`, which
     does not agree.

   With those two labels corrected to `fix`, the stand-in gives 0 of 104. The test must be repeated on the gold set.
   Pack: 23 documents changed. Every removed line is a footer, slug, share prompt or breadcrumb.
5. **Side effects:**
   - The list knows only the sites seen in this pack (News18, The Report, somewhere in net, ABP). Other boilerplate
     stays: 6 of my 15 C1 documents are not treated, for example `www.BDeBooks. Com` in `7891ebc9ab87ad20`.
   - A line of real content that quotes a copyright notice, or starts with `©`, would be removed. So would a
     paragraph that ends with an agency slug and continues on the same line.

## Medicine 7

1. **Disease:** B4, template and code junk: a literal backslash-n or backslash-t (two characters) left by an export
   in place of a line break or a tab. Count 8, all in one document. The pack has 7 documents with it.
   - `7378fae019d08f76`, offset 97: `TRT Bangla\nHome/`, a literal `\n` (5C 6E) between the title and the menu.
   - `7378fae019d08f76`, offset 309: a literal `\t` (5C 74).
   - `a16e7291bad69a10`: `আলোচিত সংবাদ\nসোমবার …`, between the title and the body (13 times in the document; not
     labelled yet).
2. **Medicine:** replace a literal `\n` with a line break and a literal `\t` with a tab, unless the backslash is
   itself escaped (`\\n`).
   ```python
   ESCAPE = re.compile(r"(?<!\\)\\([nt])")

   def b4(text):
       return ESCAPE.sub(lambda m: "\n" if m.group(1) == "n" else "\t", text), False
   ```
3. **Action:** fix the text.
4. **Harm test:** gold (stand-in): 0 changed, 0 dropped, 0.0%. **Pass.** Pack: 7 documents changed, all of them at
   the join of a title and the body.
5. **Side effects:**
   - In a document with LaTeX or program code, `\theta`, `\nabla`, `\n` in a string, or a Windows path such as
     `C:\new` would be broken. The pack has no such document, but a code or science pack would.
   - Other escapes (`\"`, `\u0985`) are not treated.

## Medicine 8

1. **Disease:** C4, two documents joined. In this pack the visible form is the headline of another article inserted
   after `আরও পড়ুন` ("read more"). Count 6, in 6 documents.
   - `58827dcddbbf097e`, offset 1496: `আরও পড়ুন- রাত পেরোলেই ভারতে আসছে ওপ্পো রেনো ৮ সিরিজ …`, a phone headline inside an
     iPhone story.
   - `7035900113f62f27`, offset 2: `#IndiaWorld Population Day 2023: শীর্ষে ভারত …`, a headline on population above a
     story on a helicopter crash.
   - `0ca18aa03db2ef77`, from offset 0: seven unrelated comments of different readers, joined only by line breaks.
2. **Medicine:** remove a line that starts with `আরও পড়ুন` or `আরো পড়ুন` (or `Read more`, `Also read`), together with
   the headline that follows on the same line.
   ```python
   READ_MORE = re.compile(r"^\s*(?:আর[োও]\s*পড়ুন|Read more|Also read)\b.*$\n?", re.M | re.I)

   def c4(text):
       return READ_MORE.sub("", text), False
   ```
3. **Action:** fix the text.
4. **Harm test:** gold (stand-in): 1 changed, 0 dropped, 0.9%. **Pass.** The document changed is `017b4f09daad47e4`.
   It has two `আরও পড়ুনঃ` headlines (horoscope stories), which I missed when I marked it `clean`. Pack: 9 documents
   changed, among them `43c51cdac699187d`, where I labelled the same line C1.
5. **Side effects:**
   - Only the marked form is found. Texts joined with no marker, such as the comment thread `0ca18aa03db2ef77` and
     the headline on top of `7035900113f62f27`, are not treated: 5 of my 6 C4 documents. These need a reader or a
     topic model.
   - A sentence in the text that starts with `আরও পড়ুন` ("read more of …", for example in an essay on reading)
     would be removed.
   - The headline goes but nothing after it: if the other article's body follows on the next lines, it stays.
