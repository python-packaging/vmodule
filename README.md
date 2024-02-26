# vmodule

This is a tiny project with no deps to add extra levels to stdlib logging, and
also something you can run from your main that processes logger-specific
verbosity.

This is inspired by glog, which considers VLOG(1) to be a more restrictive
version of LOG(INFO).  For simplicity we just map that onto numeric severities.

Here's a combined demo of how you might use this library -- we don't depend on
`click` but you certainly can:

```py
from logging import getLogger
from typing import Optional

import click

from vmodule import VLOG_1, VLOG_2, vmodule_init

LOG = getLogger(__name__)


@click.command()
@click.option("-v", type=int)
@click.option("--vmodule")
def main(v: Optional[int], vmodule: Optional[str]) -> None:
    vmodule_init(v, vmodule)
    LOG.warning("Warn")
    LOG.info("Starting up")
    LOG.log(VLOG_1, "Verbose %d", 1)
    LOG.log(VLOG_2, "Verbose %d", 2)
    LOG.debug("foo")


if __name__ == "__main__":
    main()
```

If you're writing a library, it's even easier:

```py
from logging import getLogger
from vmodule import VLOG_1

LOG = getLogger(__name__)

def foo(...):
    LOG.info("Starting up")
    LOG.log(VLOG_1, "Verbose %d", 1)
```


# Command line parsing

If you use the example above,

```
(unset)  -> WARNING
-v 0     -> INFO
-v 1     -> VLOG_1
-v 2     -> VLOG_2
-v 3     -> VLOG_3
-v 10    -> DEBUG (by accident)
```

You can also specify those same numbers with `--vmodule` although there are two
gotchas:

1. You CLI's logger is quite possibly called `"__main__"`.
2. This doesn't take wildcards or use hierarchy.  You need to specify actual
   logger names.


```
--vmodule filelock=10,concurrent.futures=0

sets filelock to DEBUG
sets concurrent.futures to INFO
```

# Version Compat

Usage of this library should work back to 3.7, but development (and mypy
compatibility) only on 3.10-3.12.  Linting requires 3.12 for full fidelity.

# License

vmodule is copyright [Tim Hatch](https://timhatch.com/), and licensed under
the MIT license.  I am providing code in this repository to you under an open
source license.  This is my personal repository; the license you receive to
my code is from me and not from my employer. See the `LICENSE` file for details.
