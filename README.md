# feednotes

> Notes on data feeds, versioning and orchestration.

## Background

A common, repetitive task in data processing systems is to implement data feeds
and task orchestration. A number of frameworks exist to solve this problem and
these notes try to gather some experiences with them.

## spotify/luigi

Early framework with a nice functional approach. Model follows map-reduce style
processing, where processing happens in stages and each stage does its own
reading and writing of the data, which may be expensive.

One of the best things about luigi is its complete independence of any third
party component, files is all you need and that alone improve maintainability a
lot. At the same time, it provides enough atomicity to be reliable.

Notions:

* task
* dependencies
* outputs (files, S3, anything, really)

It is important to keep orchestration code separate from business logic, e.g.
in form of additional libraries or external tools.

The overall notation could be reduced, e.g. from

```python
class Task:
    date = Parameter(...)
    def requires(self):
        return ...
    def run(self):
        with open(...) as f:
            for line in f:
                data = parse(line)
    def output(self):
        return ...
```

To a more lightweight, function-only approach.

```python
@param(date, default=datetime.date.today())
def process_file(othertask):
    """ othertask being a dependency """
    with open(othertask.output) ... as f:
        for line in f:
            data = parse(line)

    return ...
```

It's possible to work with GB or TB sized data, as the data passed between tasks is just the location of the artifact.
