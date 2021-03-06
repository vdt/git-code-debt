[![Build Status](https://travis-ci.org/asottile/git-code-debt.svg?branch=master)](https://travis-ci.org/asottile/git-code-debt)
[![Coverage Status](https://img.shields.io/coveralls/asottile/git-code-debt.svg)](https://coveralls.io/r/asottile/git-code-debt?branch=master)

git-code-debt
=============

A dashboard for monitoring code debt in a git repository.


## Installation

`pip install git-code-debt`


## Usage


### Basic / tl;dr Usage

```
# Create tables
$ git-code-debt-create-tables database.db
# Generate code metric data (substitute your own repo path)
$ git-code-debt-generate git://github.com/asottile/git-code-debt database.db
# Start the server
$ git-code-debt-server database.db
```

### Updating data on an existing database

Adding data to the database is as simple as running generate again.
`git-code-debt` will pick up in the git history from where data was generated
previously.

```
# (substitute your own repo path)
$ git-code-debt-generate git://github.com/asottile/git-code-debt database.db
```

### Creating your own metrics

1. Create a python project which adds `git-code-debt` as a dependency.
2. Create a package where you'll write your metrics
3. Pass your package(s) as positional argument(s) to
    `git-code-debt-create-tables` as well as `git-code-debt-generate`.


The simplest way to write your own custom metrics is to extend
`git_code_debt.metrics.base.SimpleLineCounterBase`


Here's what the base class looks like

```python

class SimpleLineCounterBase(DiffParserBase):
    # ...

    def should_include_file(self, file_diff_stat):
        """Implement me to return whether a filename should be included.
        By default, this returns True.

        :param FileDiffStat file_diff_stat:
        """
        return True

    def line_matches_metric(self, line, file_diff_stat):
        """Implement me to return whether a line matches the metric.

        :param bytes line: Line in the file
        :param FileDiffStat file_diff_stat:
        """
        raise NotImplementedError
```

Here's an example metric

```python
from git_code_debt.metrics.base import SimpleLineCounterBase


class Python__init__LineCount(SimpleLineCounterBase):
    def should_include_file(self, file_diff_stat):
        return file_diff_stat.filename == b'__init__.py'

    def line_matches_metric(self, line, file_diff_stat):
        # All lines in __init__.py match
        return True
```

More complex metrics can extend `DiffParserBase`

```python
class DiffParserBase(object):
    """Generates metrics from git show"""
    # Specify __metric__ = False to not be included (useful for base classes)
    __metric__ = False

    def get_metrics_from_stat(self, commit, file_diff_stats):
        """Implement me to yield Metric objects from the input list of
        FileStat objects.

        Args:
            commit - Commit object
            file_diff_stats - list of FileDiffStat objects

        Returns:
           generator of Metric objects
        """
        raise NotImplementedError

    def get_possible_metric_ids(self):
        raise NotImplementedError
```


## Some screenshots

### Index
![Example screen index](https://raw.githubusercontent.com/asottile/git-code-debt/master/img/debt_screen_1.png)

### Graph
![Example screen graph](https://raw.githubusercontent.com/asottile/git-code-debt/master/img/debt_screen_2.png)
