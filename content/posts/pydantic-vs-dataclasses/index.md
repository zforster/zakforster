+++
date = "2025-06-09T21:55:47+01:00"
draft = true
title = "Pydantic vs Data Classes: When to Use Each"
categories = ['Python', 'Software Engineering']
+++
## The Issue With Dictionaries

In Python, one of the most basic methods of representing application data is via the built in type [dict.](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) Using this type, we can very easily and quickly represent data, in this case some hypothetical portfolio position data:

```python
position = {
    'ticker': 'AAPL',
    'weight': 0.5,
    'sector': 'Information Technology'
}
```

Whilst this is sufficient for a basic use case, or one off script, there are signficiant disadvantages to this method of data representation. If we are looking to create a longer lived project where system maintainability is of high importance, we should seek alternative methods of representation. 

### Maintainability
As a developer, opening a codebase to be greeted with plain dictionaries being passed everywhere is a frustrating experience. Using plain dictionaries can hinder mantainability. 

The primary issue is the lack of transparency over what the object contains; the object's properties and associated types are not documented. This makes it difficult for developers to understand the data model without inspecting the object at run time or manually tracing usage through the codebase.

```python
def annoying_function(item: dict, other_item: dict) -> dict
    ...
```

Not only is this ambiguity impacting developers, it impacts static type checkers such as [mypy,](https://mypy.readthedocs.io/en/stable/) which cannot meaningfully check the type saftey of plain dictionary usage downstream, reducing the effectiveness of type safety and increasing the risk of run time errors.

Additionally, dictionaries are limited to key lookups rather than acessing attributes. Again this requires advanced knowledge of the keys contained within the object.

```python
position_ticker = position['ticker']
```


## What About Classes?
As an alternative, we can define a Python [class](https://docs.python.org/3/tutorial/classes.html) to represent our portfolio position. An improvement? We now have our type hints, attribute access and improved static type checking, but this is overly verbose for simple data representation.

```python
class Position:
    def __init__(self, ticker: str, weight: float, sector: str)
        self.ticker = ticker
        self.weight = weight
        self.sector = sector

p = Position(ticker="AAPL", weight=0.5, sector="Information Technology")

position_ticker = p.ticker
```

## Enter Data Classes
Back in Python 3.7, the [dataclass](https://docs.python.org/3/library/dataclasses.html) decorator was introduced, significantly reducing the volume of boilerplate that needed to be written to represent our data as classes. 

Under the hood they are still regular Python classes. The decorator adds the same `__init__()` function seen above.

```python
from dataclasses import dataclass

@dataclass
class Position:
    ticker: str
    weight: float
    sector: str
```

The `dataclass` provides us with a lean way to represent data as classes, providing type hints, attribute access, improved static type checking and custom domain logic. We can even provide some custom validation logic utilising `__post_init__`.

```python
from dataclasses import dataclass

class WeightException(Exception):
    pass

@dataclass
class Position:
    ticker: str
    weight: float
    sector: str

    def __post_init__(self):
        if self.weight > 1:
            raise WeightException("Invalid Position Weight")
```

## When Data Classes Fail
Whilst the Data Class provides type hints, we have no guarentee as to the type saftey of our fields at run time. In the below example, we naively parse some data returned via API into a `Position` instance. Notice we expect the `weight` attribute to be a `float`, yet the Data Class has no problem accepting a `string`.

```python
from dataclasses import dataclass

@dataclass
class Position:
    ticker: str
    weight: float
    sector: str

p = Position(
    **{
        "ticker": "AAPL",
        "weight": "0.5",
        "sector": "Information Technology",
    }
)
print(type(p.weight))

>> <class 'int'>
```

We do have the option to guard against this behaviour with custom validation logic inside of `__post_init__`, although this again introduces verbosity.

## Enter Pydantic
In Pydantic, users define models that inherit from BaseModel. Pydantic models also provide type hints and attribute access, however unlike Data Classes, Pydantic parses and validates the input data to ensure the resulting object conforms to the expected type definitions, raising `ValidationError` in the case the data does not conform, or cannot be cast to the expected type. 

```python
from pydantic import BaseModel

class Position(BaseModel):
    ticker: str
    weight: float
    sector: str
```

---

{{< newsletter >}}

---
