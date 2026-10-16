+++
title = "6 Python Tips That Quietly Level Up Your Code"
date = 2025-10-25
description = "A few understated tricks that separate working code from elegant code."
draft = false

[extra]
toc = true
+++

## Introduction

If you're like me at the moment, you've gotten to a point in your software development journey
where you find yourself writing code to tick off tasks from a task board or complete features for
a sprint. At such times, the focus sometimes shifts from poetic code to functional code (not
implying that functional code cannot be easily poetic, but it takes mastery and experience to
maintain such quality when working under tight deadlines). It is at this point that you're more
concerned with getting things to work than with how they feel and look.

Below are some Python tips and tricks I use and believe are not as emphasized as they should be.
They would do good to your mental health when writing code and, more importantly, when updating
that code 3 months from now.

**TL;DR:**

1. Inline single-line blocks in block expressions
2. Use underscores to delimit a large number
3. Dictionary comprehension
4. Using the `setdefault` dictionary method
5. Ignore exceptions with `contextlib.suppress`
6. Multiple statements on a single line

## 1. Inline single-line blocks in block expressions

```python
some_condition = True

if some_condition:
    print("Some Condition is True")
else:
    print("Some Condition is Falsey")
```

In the code sample above, the correct conventional 4-space indentation is used for the expression
in a block expression. But in this particular case, the block contains just a single expression.
This might be a trivial case, but many such scenarios arise a lot in functional codebases, and you
can save some real estate by inlining the single expression on the same line as the `if` block.

```python
some_condition = True

if some_condition: print("Some Condition is True")
else: print("Some Condition is False")
```

This is perfectly valid Python, and in some cases makes your code easier to read and follow. Do
note that you **can only** inline single-line expressions; if the expression is multi-line, you
have to use the conventional 4-space indentation to communicate the block scope to the Python
interpreter. If, however, always using the block format for block expressions works better for
you, by all means do use that way.

## 2. Use underscores to delimit a large number

This is more common than some of the tips on this list. Still, for the uninitiated, here goes
nothing. When dealing with literal values in your application, it can be counterintuitive or even
error-prone having to figure out the value of a large literal number at a glance, and this could
very well mean the only thing standing between you and your continued state of mental flow.
Python — and some other high-level programming languages like Rust — allows the use of `_` in
literal numbers to show their unit.

```python
obscure_large_number = 3422345365
clear_large_number = 3_422_345_365  # it is easier to tell the unit value when delimited by underscores
```

## 3. Dictionary comprehension

The more popular sibling of the comprehension family in Python is the list comprehension, where a
list is generated from an iterable in a single expression without needing a conventional for-loop
to populate the list. This same process can be used to generate a dictionary or a set.

Let's imagine we have a list of names and want to generate a dictionary whose key is a name and
whose value is the length of that name's characters. We can achieve this with a traditional
for-loop, like so:

```python
names = ["Brian", "Anthony", "Maryann", "Joseph", "Emediong"]
word_count = {}
for name in names:
    word_count[name] = len(name)

print(word_count)

# Output
# {'Brian': 5, 'Anthony': 7, 'Maryann': 7, 'Joseph': 6, 'Emediong': 8}
```

Now this same functionality can be achieved through dictionary comprehension in a more
comprehensive way (pun intended). It looks just like a list comprehension, but curly braces are
used, and a colon separates the key from the value being derived from the iterable.

```python
names = ["Brian", "Anthony", "Maryann", "Joseph", "Emediong"]
word_count = {name: len(name) for name in names}

# general dictionary comprehension syntax
# {key: derived_value for key in sequence}

print(word_count)

# Output
# {'Brian': 5, 'Anthony': 7, 'Maryann': 7, 'Joseph': 6, 'Emediong': 8}
```

## 4. Using the `setdefault` dictionary method

Still on the topic of working with dictionaries in Python: there are times you wish to update a
value in a dictionary, but you have to ensure that said key already exists in the dictionary, or
Python would raise a `KeyError`. You could write a one-liner to check for the existence of the key
before updating the key's value, like so:

```python
race_laps = {"Brian": 234, "Anthony": 525, "Maryann": 534, "Joseph": 467, "Emediong": 876}


new_lap_data = {
    "Brian": 34,
    "Anthony": 45,
    "Felix": 100,
}

for driver, distance in new_lap_data.items():
    if driver in race_laps:
        race_laps[driver] += distance
    else:
        race_laps[driver] = distance
```

But it is clear that this introduces unnecessary verbosity into our program by introducing an
`if`/`else` block. We can use the `setdefault` method of a dictionary to set a default value for a
key before trying to update it, in case it is not present in the dictionary. The same logic above
can be rewritten like so with the `setdefault` method:

```python
race_laps = {"Brian": 234, "Anthony": 525, "Maryann": 534, "Joseph": 467, "Emediong": 876}

new_lap_data = {
    "Brian": 34,
    "Anthony": 45,
    "Felix": 100,
}

for driver, distance in new_lap_data.items():
    race_laps.setdefault(driver, 0)  # this sets a value of 0 for the driver if not already in the dictionary
    race_laps[driver] += distance
```

This approach makes the code more concise and is more Pythonic than the structure with the
`if`/`else` block.

## 5. Ignore exceptions with `contextlib.suppress`

Sometimes we want to catch an exception and then do nothing with it — we're catching it to prevent
the program from breaking or stopping, and to continue with the flow. This can be done with a
`try`/`except` block, and if you have no logic to run in the `except` block, you can use the `pass`
statement to indicate nothingness.

```python
def do_some_action():
    # some action that might fail but is not critical to the whole program
    pass

while True:
    try:
        do_some_action()
    except Exception:
        pass
```

In the code sample above, we put the logic that might raise an exception in the `try` block and
then ignore the cases where it raises. This structure forces us to use a block; we could apply the
first tip from this post to further improve the structure and gain some real estate.

```python
def do_some_action():
    # some action that might fail but is not critical to the whole program
    pass

while True:
    try: do_some_action()
    except Exception: pass
```

However, we can go even further and use a standard library class that is not very well known, to
suppress exceptions and continue with your program as if they hadn't happened. The `contextlib`
standard library provides a class called `suppress` that can be used to suppress exceptions in our
code. Going back to the same example as above, the code can be rewritten with `suppress` as
follows:

```python
from contextlib import suppress


def do_some_action():
    # some action that might fail but is not critical to the whole program
    pass

while True:
    with suppress(Exception):  # the exception is suppressed silently
        do_some_action()
```

This suppresses the exception type passed to it, and just like the normal exception rule, a parent
class can suppress child exception classes. You can also suppress multiple exceptions with a
`with` block by passing more exception classes to `suppress`, separated by commas.

```python
with suppress(ValueError, ValidationError):  # ValueError and ValidationError are suppressed here
    ...
```

## 6. Multiple statements on a single line

Conventionally, a newline in Python code declares the end of a statement and the start of another.
Some other programming languages use semicolons to separate statements, but Python is a whitespace
language. Regardless, semicolons can be used in Python, and in rare situations you might need to
put more than one statement on a single line — you can do this by terminating each statement with a
semicolon.

```python
# Conventional Python statement flow
# each newline denotes a different statement
a = 1
b = 2
print(f"A = {a}")
print(f"B = {b}")

# using semicolons to separate statements
a = 1;  # this is also valid Python
b = 3;
print(f"A = {a}");

# putting multiple statements on a single line
a = 10;print(f"A = {a}")
```

Clearly, putting each statement on its own line leads to better readability for your code, but in
rare scenarios, knowing that multiple statements can be placed on a single line can give you the
ability to do certain magic.

## Conclusion

I love reading and learning about these little neat tricks and try to apply them when I see fit in
my projects. If there are similar tricks you know and would love to share, I'd love to hear them.
