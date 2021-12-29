---
layout: post
title: "Submit an Arbitrary Number of Coroutines in Asyncio Event Loop during Runtime"
---

## Asyncio

Python package [asyncio](https://docs.python.org/3/library/asyncio.html) allows the script to be executed concurrently. This package is suitable for IO-bound network code, e.g., elimating the busy waiting part when making an API request.

## Problem

As I adopt asyncio-related packages in some projects, a need occurred that an arbitrary number of coroutines should be executed during runtime. The coroutine will be called inside an infinite loop when certain condition is met. 

## Solution

By looking through the documentations, it is not easy to achieve that logic by using ascynio package itself. You can schedule some random coroutines before the program is run, but I do not find a way to add them during runtime. 

What I ended up with is to explicitly submit coroutines to a separate thread where the event loop is running, by calling `run_coroutine_threadsafe` method.

Below is the sample code for reference:

```
class AsyncLoopThread(Thread):
    """loop wrapper that runs in a separate thread"""
    def __init__(self):
        super().__init__(daemon=True)
        self.loop = asyncio.new_event_loop()

    def run(self):
        asyncio.set_event_loop(self.loop)
        self.loop.run_forever()

async def coroutine(num, sec):
    """example coroutine"""
    await asyncio.sleep(sec)
    print('Coro %d has finished' % num)

if __name__ == '__main__':
    # start a loop in another thread
    loop_handler = AsyncLoopThread()
    loop_handler.start()

    while True:
        # adding first 1000 coros
        for i in range(1000):
            print('Add Coro %d to the loop' % i)
            asyncio.run_coroutine_threadsafe(coroutine(i, randint(3, 5)), loop_handler.loop)

        time.sleep(3)
        print('Adding 1000 more coros')

        # adding 1000 more coros
        for i in range(1000, 2000):
            print('Add Coro %d to the loop' % i)
            asyncio.run_coroutine_threadsafe(coroutine(i, randint(3, 5)), loop_handler.loop)
```
