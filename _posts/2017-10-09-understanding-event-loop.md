---
layout: post
comments: true
title:  "Understanding Event Loops by building a simple one!"
date:   2017-10-09 18:00:02 +0100
categories: python
tags: python
---

## Challenge
So, what is the magic behind Event Loops? We see them everywhere nowadays? [Asyncio][asyncio-web], [Tornado][tornado-web], [Nodejs][nodejs-web], etc. etc.!

## Prerequisites
* Use Python 3 

## Description
All in all, an EventLoop is just a loop :) A while loop!
And what about the Events? Well, events span from regular network IO, user interactions, to message passing etc.
In Unix systems, most of the events are handled by ["file-like"][std-files] structures. For each process, for example, the OS creates 3 files. One for stdin, one for stdout and one for stderr. Same happens for sockets, pipes etc.

Handling these types of files normally happens through system calls (which differ from OS to OS) like select, poll, epoll, kqueue etc. The aforementioned calls provide us with the status of the file. If it is open for read/write and how many bits are ready to be read/written. Check [here][io-models] for more. 

The important part for our study is the [arguments][select-args] of the select method. And most importantly the `timeout`. Why? Because is makes the call non blocking. That practically means that our while loop will keep running forever. And that gives us the ability to squeeze little more work in there :)

Check the code below

## Code
```python
from functools import partial, wraps
import selectors
import sys
import types
import logging

FORMAT = '[%(levelname)s] %(asctime)-15s - %(message)s'
logging.basicConfig(format=FORMAT, level=logging.DEBUG)

logger = logging.getLogger('event_loop')

def coroutine(f):
    @wraps(f)
    def wrapper(*args, **kwargs):
        g = f(*args, **kwargs)
        g.__next__()
        return g
    return wrapper


fib = lambda n: n if n < 2 else fib(n-1) + fib(n-2)

class IOEventLoop(object):

    def __init__(self):
        self._running = False
        self._selector = selectors.DefaultSelector()

        # FIFO List of tasks scheduled to run
        self._tasks = []

        # Register polling on stdin (which is a fd) for read events
        self._selector.register(sys.stdin, selectors.EVENT_READ)

        # (coroutine, stack) pair of tasks waiting for stdin event
        self._tasks_waiting_on_stdin_event = []

    def resume_task(self, coroutine, value=None):
        result = coroutine.send(value)
        if isinstance(result, types.GeneratorType):
            logger.debug("Schedules generator task")
            self.schedule(result, None, coroutine)
        elif result is sys.stdin:
            logger.debug("Schedules stdin task")
            self._tasks_waiting_on_stdin_event.append(coroutine)

    def schedule(self, coroutine, value=None):
        logger.debug("Schedules {} with value {}".format(coroutine.__name__, value))
        task = partial(self.resume_task, coroutine, value)
        self._tasks.append(task)

    def run_forever(self):
        self._running = True
        # That is the main Loop (reads io events and executes tasks from the task list)
        while self._running:
            try:
                # Check for IO Events
                for key, _ in self._selector.select(0):
                    line = key.fileobj.readline().strip()
                    for task in self._tasks_waiting_on_stdin_event:
                        self.schedule(task, line)
                    self._tasks_waiting_on_stdin_event.clear()

                # Run the next task
                if self._tasks:
                    task = self._tasks.pop(0)
                    task()
            except KeyboardInterrupt:
                self._running = False


@coroutine
def stdin_line_handler():
    while True:
        logger.debug("yielding stdin")
        line = yield sys.stdin
        if line:
            n = int(line)
            t = fib(n)
            logger.info("fib({}) = {}".format(n, t))


def main():
    loop = IOEventLoop()
    loop.schedule(stdin_line_handler())
    loop.run_forever()


if __name__ == '__main__':
    main()
```

The whole idea of how Event Loops work resides in the `run_forever` method. A `while loop` keeps reading from the input file is a non-blocking manner and executes tasks that are already in the task list, like our nice fibonacci method.

What is the whole idea of the snippet above? A selector reads always from the stdin file descriptor for a number. Then uses that number as input to the fibonacci method. Replace the sys.stdin with a socket and the `sdtin_line_handler` with an `http_request_handler` and you have just built the simplest single threaded IO Loop!

It worths mentioning a really important aspect when using event loops. And that is the implementation in the `fib` method. If you try executing the snippet above with input 10, it will return really fast and you will not notice the loop blocking. Try it with input number 100 and you will see that the loop is now blocked, since it tries to calculate the Fibonacci number. The CPU is fully occupied!

In that case we need to refactor our `fib` method to something like:

```python
def fib(n):
    def fib_list_gen():
        a, b = 0, 1
        for _ in range(n+1):
            yield a
            a, b = b, a + b
        return b

    for index, fib_num in enumerate(fib_list_gen()):
        if index == n: return fib_num
```

Give it a try. You will notice that the calculation is much faster and the loop doesn't block anymore. That is because we have transformed the Fibonacci calculation to asynchronous by using the yield operator.

That is something to keep in mind when developing using eventloops. Try to make everything async all the way to avoid blocking the loop for long periods! 

[asyncio-web]: https://docs.python.org/3/library/asyncio.html
[tornado-web]: https://github.com/tornadoweb/tornado
[nodejs-web]: https://github.com/nodejs/node
[std-files]: http://www.learnlinux.org.za/courses/build/shell-scripting/ch01s04.html
[io-models]: https://acemood.github.io/2016/02/01/event-loop-in-javascript/
[select-args]: https://docs.python.org/3/library/select.html#select.select
