# Number forms #
This is the proposed method for stringifying numbers in a generalized manner. This is proposed as a new key under the specialized *pretty* for number types.

## comma, period, etc ##

```mprl
[pattern, ",?###"]
```

Reads right to left and repeats:

1. "###": draw up to 3 digits
2. ",?" (only if 3 digits were drawn): If there is a fourth digit, draw a ","

That is, # pops a digit and ? peeks a digit. Reading right to left, if a ?/# is matched, it draws the string up to the next ?/#, if that is unmatched then it stops processing. The symbols are thrown away but # also draws the digit. The digit is not used up in the case of ?, obviously. If you must use a literal # or ? idk just don't for now. You may specify **ltr** in *dir* if you need to process the other way.

Expanded form, if it needs to be used in conjunction with something else (like *base*), is just:

```mprl
form {
    pattern: ",?###"
}
```


## CJK ##
This should work for all digit systems...

*number#* lets you state the literal value for this specific number

*digit#* lets you state the value for this digit as:

* `2`
    - appears as "2" in every slot
* `[..., 100s, 10s, 1s]`
    - the digit used changes based on position
    - this limits the amount of possible digits
* `[..., 20: 廿, 二]` or `[二, 20: 廿, ...]`
    - specify specific points where it changes
    - the case that's not a list (concat above), is the general case
        + that is, it will appear in all non-mentioned cases
    - if there is no general case defined, this digit may only appear in the defined spaces

*after#* states what is inserted after the given digit when read ltr.

*before#* similar but inserts before.

\# may be an actual arabic number or of the form: <u>base</u>**e**<u>digit</u>

* such that 10e1 is the tens digit in a decimal system
* this is not meant to be used when the number doesn't represent a multiple of the base. That is, only one base is allowed in a form.

Note that for the above two forms, the digit must exist (both in the number being processed and as a *digit#*) to be triggered.

The base is inferred as best as possible but can be specified with *base*, inferrence is as:

1. taken from the base in a number form, if that form is ever used
2. taken from the highest *digit#*'s # + 1

One can specify a minimized CJK form with:

* "cjk" doesn't have defaults, so places not defined won't exist: `[cjk, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 100, 1000, 10e4, 10e8, ..., n-1 * 10e4]`
* Or to replace only specific forms: `[japanese, 20: 廿]`
    - Read like "Japanese...but with 20 as 'hatachi'"

### japanese:kanji ###

```mprl
form {
    number0: 零
    digit1: [一, 10:"", 100:"", 1000:""]
    digit2: 二
    digit3: 三
    digit4: 四
    digit5: 五
    digit6: 六
    digit7: 七
    digit8: 八
    digit9: 九
    after1 : 十
    after2 : 百
    after3 : 千
    after4 : 万
    after8 : 億
    after12: 兆
    after16: 京
    after20: 垓
    after24: 秭
    after28: 穣
    after32: 溝
    after36: 澗
    after40: 正
    after44: 載
    after48: 極
}
```

### japanese:arabic ###

```mprl
form {
    digit0: 0
    digit1: 1
    digit2: 2
    digit3: 3
    digit4: 4
    digit5: 5
    digit6: 6
    digit7: 7
    digit8: 8
    digit9: 9
    after4 : 万
    after8 : 億
    after12: 兆
    after16: 京
    after20: 垓
    after24: 秭
    after28: 穣
    after32: 溝
    after36: 澗
    after40: 正
    after44: 載
    after48: 極
}
```


## roman ##
This should work for all 1-dimensional counting rod systems...

*every#* divides by the number as many times as it can, writing the given value for each success. It then continues with the remainder. By default it writes ltr but this can be changed with the *dir* key.

Obviously, these are executed from the largest number to the smallest.

*replace* provides a list of replacements to make in a post-processing stage.

```mprl
form {
    every1000: M
    every500 : D
    every100 : C
    every50  : L
    every10  : X
    every5   : V
    every1   : I
    replace {
        DCCCC: CM
        CCCC : CD
        LXXXX: XC
        XXXX : XL
        VIIII: IX
        IIII : IV
    }
}
```

One may also specify the type of roman numerals to use:

* `roman:ascii` or `roman:ascii:upper`
    - ASCII-compatible version: IVXLCDM
* `roman:ascii:lower`
    - ASCII-compatible lowercase version: ivxlcdm
* `roman:modern` or `roman:unicode`
    - Using the designated unicode codepoints of the modern variant.
    - With `:upper` or without `:lower`: ⅠⅡⅢⅣⅤⅥⅦⅧⅨⅩⅪⅫⅬⅭⅮⅯↁↂↇↈ
    - With `:lower`: ⅰⅱⅲⅳⅴⅵⅶⅷⅸⅹⅺⅻⅼⅽⅾⅿↁↂↇↈ
    - `:composed` is the default shown above.
    - `:decomposed` will only use ⅠⅤⅩⅬⅭⅮⅯ or ⅰⅴⅹⅼⅽⅾⅿ
    - Order of upper/lower and de/composed doesn't matter.
* `[roman, I, V, X, L, C, D, M, 5k, 10k, 50k, 100k, ..., n-2 * 5, n-1 * 2]`
    - Every odd entry represents the previous number * 2, where 1 is the first odd entry.
    - Every even entry represents the previous number * 5
* `[roman:ascii, 1: |]`
    - Makes specific replacements, defaulting the rest.
    - The "ascii" version doesn't require replacement to be ASCII-compatible.


## bases ##
Standard bases should be available as forms, too. For the most part, these are too modern for other number systems to actually handle them so they're considered their own forms which use arabic numerals. The form mentioned in CJK does allow customization.

* **bin** or **binary**: base 2
* **oct** or **octal**: base 8
* **dec** or **decimal**: base 10
* **duodecimal**: base 12
* **hex** or **hexadecimal**: base 16
* or just specify the number or "base #"
