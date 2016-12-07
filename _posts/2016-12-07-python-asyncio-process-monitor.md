---
layout: post
comments: true
title:  "Monitor processes with Python asyncio"
date:   2016-12-07 19:00:02 +0100
categories: python
tags: python asyncio
---

## Chalenge Definition
Use the new [Python 3.5 asyncio][python-asyncio] to asynchronously spawn processes and monitor their status

## Prerequisites
Python 3.5

## Code
```python
import asyncio
import sys

procs=dict()

@asyncio.coroutine
def start_process():
    script = """
import sys
import random
import time
import multiprocessing

proc = multiprocessing.current_process()
wait_range = random.randint(1, 15)
print('proc with id {pid} has a wait_range of {w_range}'.format(pid=str(proc.pid), w_range=str(wait_range)))
for i in range(wait_range):
    print('proc with id {pid} and iteration {it}'.format(pid=str(proc.pid), it=str(i)))
    time.sleep(1)

sys.exit(random.choice([0, 1]))
    """

    # Create the subprocess, redirect the standard output into a pipe
    create = asyncio.create_subprocess_exec(sys.executable, '-c', script,
                                            stdout=asyncio.subprocess.PIPE,
                                            bufsize=0)
    proc = yield from create
    return proc

@asyncio.coroutine
def readline(proc):
    data = yield from proc.stdout.readline()
    line = data.decode('ascii').rstrip()
    return line

@asyncio.coroutine
def monitor_procs(procs):
    while True:
        yield from asyncio.sleep(1)
        for pid, proc in list(procs.items()):
            if proc.returncode is None:
                print('Proc with id: {pid} still running'.format(pid=pid))
            elif proc.returncode is 0:
                print('Proc with id: {pid} exited with code 0, removing...'.format(pid=pid))
                procs.pop(pid, None)
            else:
                print('Proc with id: {pid} exited with code {exit_code}, removing...'.format(pid=pid, exit_code=proc.returncode))
                procs.pop(pid, None)

@asyncio.coroutine
def generate_process(procs):
    while True:
        yield from asyncio.sleep(5)
        print('Starting new Process...')
        proc = yield from start_process()
        procs[proc.pid] = proc


@asyncio.coroutine
def follow_procs(procs):
    while True:
        yield from asyncio.sleep(0.01)
        for pid, proc in list(procs.items()):
            line = yield from readline(proc)
            if line:
                print(line)

if sys.platform == "win32":
    loop = asyncio.ProactorEventLoop()
    asyncio.set_event_loop(loop)

tasks = asyncio.gather(
    asyncio.async(monitor_procs(procs)),
    asyncio.async(generate_process(procs)),
    #asyncio.async(follow_procs(procs))
)
    
asyncio.get_event_loop().run_forever()
```

[python-asyncio]: https://docs.python.org/3/library/asyncio.html
