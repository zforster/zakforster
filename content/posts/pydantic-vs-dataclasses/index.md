+++
date = "2025-06-22T20:00:47+01:00"
draft = false
title = "Pydantic vs Data Classes: Which Should You Use?"
categories = ['Python', 'Software Engineering', 'Data Modelling']
description = "Compare Pydantic models and Python data classes. Learn when to use each for validation, type safety, serialization, and maintainable data modelling."
+++

## Approaches to Data Modelling Using Python

The modern software engineer must master the skill of representing real-world entities as objects in code. This translation process, known as data modelling, defines the structure, rules and relationships of the domain objects within an application.

In Python, data can be modelled in various ways, using built-in approaches such as dictionaries, classes and dataclasses, or by using external packages such as Pydantic. In this article, we explore these methods to help you decide which is most suitable to your project's needs.

## Why You Shouldn't Use Dictionaries

Python's built-in type [dictionary](https://docs.python.org/3/tutorial/datastructures.html#dictionaries) provides software engineers with a basic method of data representation. As we gain more experience with Python, we come to realise the limitations associated with the dictionary type and the impact it has on the maintainability of the systems we create.

If we are looking to create professional and maintainable projects, we should leave the dictionary behind and seek alternate methods of data modelling. The dictionary suffers from these key issues:

### Meaningless Type Hints

Over reliance on the vanilla dictionary type can hinder maintainability. The lack of transparency over the object's properties and associated types makes it difficult for developers to understand the data model without inspecting the object at runtime or manually tracing usage through the codebase.

```python
def annoying_function(item: dict, other_item: dict) -> dict:
    ...

"""
This function signature isn't useful. It is too vague.
- We don't know what keys the arguments should include.
- We have no context as to the structure of the result.
- To understand the objects, we would need to inspect at runtime.
"""
```

### Impaired Static Type Analysis

The ambiguity of the dictionary extends to static type checkers such as [mypy,](https://mypy.readthedocs.io/en/stable/) which cannot meaningfully check the type safety of our dictionary keys downstream, reducing the effectiveness of the check and increasing the risk of runtime errors.

### Key Over Attribute Access

When accessing a value within our dictionary, we are limited to using key lookups rather than attribute access. This requires advanced knowledge as to the contents of the object. In comparison, if attribute access was available we would have autocompletion options within our IDE.

```python
# hypothetical portfolio position represented as a dictionary
position = {
    'ticker': 'AAPL',
    'weight': 0.5,
    'sector': 'Information Technology'
}

position_ticker = position['ticker']  # ✅ Requires knowledge of the structure
position_ticker = position.ticker  # ❌ Raises AttributeError
```

## Data Modelling Using Classes

As an alternative to the dictionary type, we can define a Python [class](https://docs.python.org/3/tutorial/classes.html) to represent our portfolio position.

```python
class Position:
    def __init__(self, ticker: str, weight: float, sector: str):
        self.ticker = ticker
        self.weight = weight
        self.sector = sector

p = Position(ticker="AAPL", weight=0.5, sector="Information Technology")

position_ticker = p.ticker
```

In many ways, this is an improvement. We now have our type hints, attribute access and improved static type checking effectiveness. This however is a verbose means of representing data - can we do better?

## Data Modelling Using Data Classes

In Python 3.7, the [dataclass](https://docs.python.org/3/library/dataclasses.html) decorator was introduced, significantly reducing the boilerplate needed to model our data as classes. Under the hood, the `dataclass` is a regular Python class. The decorator simply adds the `__init__` function for us, removing the need to explicitly define it and pass the arguments manually.

```python
from dataclasses import dataclass

@dataclass
class Position:
    ticker: str
    weight: float
    sector: str
```

The `dataclass` provides a lean way to represent data as classes and has the same benefits as the regular Python class without the verbosity. We can even provide custom validation logic utilising `__post_init__`.

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

### Data Classes & Run-Time Type Safety

Initialised `dataclass` objects provide type hints, but unless we were to define bespoke logic, the `dataclass` does not enforce type safety at runtime.

In the below example, we naively parse position data into a `Position` instance. Our class definition declares the `weight` attribute to be a `float`, yet the class has no problem accepting a `string`.

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
print(type(p.weight))  # <class 'str'>
```

## Data Modelling Using Pydantic

In [Pydantic,](https://docs.pydantic.dev/latest/) users define models that inherit from `BaseModel`. Unlike `dataclasses` any class inheriting from `BaseModel` will validate the input data to ensure the resulting object conforms to the expected type definitions. `ValidationError` is raised in the event the field does not conform, or cannot be cast to the expected type.

```python
from pydantic import BaseModel

class Position(BaseModel):
    ticker: str
    weight: float
    sector: str

p = Position(
    **{
        "ticker": "AAPL",
        "weight": "oops!",
        "sector": "Information Technology",
    }
)
```

The above code raises a `ValidationError` error because `weight` cannot be parsed as a `float`:

```
pydantic_core._pydantic_core.ValidationError: 1 validation error for Position
weight
  Input should be a valid number, unable to parse string as a number [type=float_parsing, input_value='oops!', input_type=str]
```

Pydantic will only guarantee the types of the resulting model, not the input data itself. This may result in the coercion of the data into the expected type. See below how `weight` is a `string`, yet Pydantic has coerced this into the `float` type according to our class definition.

```python
p = Position(
    **{
        "ticker": "AAPL",
        "weight": "0.5",
        "sector": "Information Technology",
    }
)
print(type(p.weight))  # <class 'float'>
```

This behaviour can be disabled by enabling [strict mode.](https://docs.pydantic.dev/latest/concepts/strict_mode/)

### Implementing Custom Validation Rules

With Pydantic we can easily build upon the default validation implementations. To enforce more complex constraints we can use model or field level [validators.](https://docs.pydantic.dev/latest/concepts/validators/)

```python
from pydantic import BaseModel, field_validator

class Position(BaseModel):
    ticker: str
    weight: float
    sector: str

    @field_validator("weight", mode='after')
    def check_weight_validity(cls, value: float):
        if value > 1 or value < 0:
            raise ValueError("weight must be less than or equal to 1")
        return value

p = Position(
    **{
        "ticker": "AAPL",
        "weight": 10,
        "sector": "Information Technology",
    }
)
```

```
pydantic_core._pydantic_core.ValidationError: 1 validation error for Position
weight
  Value error, weight must be less than or equal to 1 [type=value_error, input_value=10, input_type=int]
```

### Data Serialisation

In the context of Pydantic, [serialisation](https://docs.pydantic.dev/latest/concepts/serialization/) refers to the process of converting a model into a dictionary or JSON string.

Pydantic's serialisation abilities are useful in many contexts. Where they really shine is when returning data to a web front end. Python models are written in `snake_case`, web apps generally expect data in `camelCase`. Pydantic provides a clean way to handle this using [alias generators.](https://docs.pydantic.dev/latest/concepts/alias/)

```python
from pydantic import BaseModel, ConfigDict
from pydantic.alias_generators import to_camel


class CamelCaseAliasModel(BaseModel):
    model_config = ConfigDict(
        alias_generator=to_camel,
        populate_by_name=True,
    )


class Position(CamelCaseAliasModel):
    ticker: str
    weight: float
    sector: str


class Portfolio(CamelCaseAliasModel):
    portfolio_name: str
    positions: list[Position]


p = Portfolio(
    positions=[
        Position(
            **{
                "ticker": "AAPL",
                 "weight": 0.5,
                 "sector": "Information Technology"
            }
        )
    ],
    portfolio_name="My Portfolio",
)
print(p.model_dump(by_alias=True))

>> {'portfolioName': 'My Portfolio', 'positions': [{'ticker': 'AAPL', 'weight': 0.5, 'sector': 'Information Technology'}]}
```

## Data Classes or Pydantic - Which Is Best For Your Project?

As a general rule, you should not replace all of your `dataclasses` with Pydantic models. In many cases, `dataclasses` are perfectly sufficient. Pydantic should only be used once you encounter a problem that actually requires it, such as implementing complex validation requirements on untrusted data.

### Use Pydantic When

- You are receiving data from an untrusted source and need to ensure it includes all fields with the correct types. For example, when handling input from a public facing API, or fetching data from a third party vendor.
- You have complex data validation requirements.
- You want easy serialisation and deserialisation, such as converting data into JSON for storage or transmission.

### Use Data Classes When

- You can already trust the data is valid, for example when representing internal data structures where data integrity is guaranteed.
- You have performance considerations; `dataclasses` are generally lighter since they have no runtime validation overhead.
- If minimising external dependencies is a priority, `dataclasses` are preferable since they are part of the standard library.
