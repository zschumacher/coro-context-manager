[![test](https://github.com/zschumacher/coro-context-manager/actions/workflows/test.yml/badge.svg)](https://github.com/zschumacher/coro-context-manager/actions/workflows/test.yml)
[![PyPI version](https://badge.fury.io/py/coro-context-manager.svg)](https://badge.fury.io/py/coro-context-manager)
[![codecov](https://codecov.io/gh/zschumacher/coro-context-manager/branch/main/graph/badge.svg?token=6610H3V6JE)](https://codecov.io/gh/zschumacher/coro-context-manager)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)
![PyPI - Python Version](https://img.shields.io/pypi/pyversions/coro-context-manager)

# coro-context-manager
*coro-context-manager* is a simple python package that includes an object that can wrap a coroutine to allow it to
behave as a context manager or a regular awaitable.

This class is super useful when you have a coroutine that returns an object that defines an async context manager using
`__aenter__` and `__aexit__`

## Installation

### pip
```console
pip install coro-context-manager
```

### poetry
```console
poetry add coro-context-manager
```


## Usage
*CoroContextManager* can be used to wrap a coroutine so that it can be awaited or called via an async context manager
in which case the library will try to use the underlying object's `__aenter__` and `__aexit__`, if they exist.
```python
import asyncio

from coro_context_manager import CoroContextManager


class MyObject:

    def __init__(self, initial_value):
        self.some_value = initial_value

    async def __aenter__(self):
        await asyncio.sleep(.1)
        self.some_value += 1
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        await asyncio.sleep(.1)
        self.some_value -= 1

    @classmethod
    async def an_io_intensive_constructor(cls, initial_value):
        await asyncio.sleep(10)
        return cls(initial_value)


async def main():
    """
    Using CoroContextManager, I get a coroutine I can await or use with an async context manager, which proxies to
    the context manager defined on object returned by the coroutine, if it exists.
    """

    # i can await it directly
    myobj = await CoroContextManager(MyObject.an_io_intensive_constructor(5))
    print(type(myobj))
    # <class '__main__.MyObject'>

    # or use it as an async context manager, not having to await it, with the same api!
    async with CoroContextManager(MyObject.an_io_intensive_constructor(5)) as myobj:
        print(type(myobj))
        # <class '__main__.MyObject'>
        print(myobj.some_value)
        # 6

    print(myobj.some_value)
    # 5


asyncio.run(main())
```

## Rationale
This is a common enough pattern used in several async packages all with slightly different implementation.  It would be
nice if there was a consistent pattern everyone was using; this package aims to provide that.

* [aiopg](https://github.com/aio-libs/aiopg/blob/master/aiopg/utils.py#L44)
* [aioodbc](https://github.com/aio-libs/aioodbc/blob/master/aioodbc/utils.py#L36)
* [aiohttp](https://github.com/aio-libs/aiohttp/blob/7514f220947ce078d4dd039cd0be49929b9976cc/aiohttp/client.py#L1082)
* [cx_Oracle_async](https://github.com/GoodManWEN/cx_Oracle_async/blob/main/cx_Oracle_async/context.py#L3)
* [aiomysql](https://github.com/aio-libs/aiomysql/blob/master/aiomysql/utils.py#L30)

## Latest Changes

