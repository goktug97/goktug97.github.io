---
layout: post
title:  "Using Python as a Configuration File"
date:   2020-01-07 21:17:23 +0300
categories: Python
---

While there are bunch of other configuration systems, I like to use
python file directly as a configuration file. I use below snippet to
import configuration files from anywhere in the system.

```python
config_file = '~/.config/AWindowManager/config'
config = types.ModuleType('config', 'Config')
with open(config_file) as f:
    code = f.read()
    code = compile(code, "config", "exec")
    exec(code, config.__dict__)
```

I thought about packaging it but before that I wanted to add some
security features so that malicious code can not be insterted into the
configuration file. It is not necessary because people should check
configuration files before using them but I wanted to experiment with
[AST](https://docs.python.org/3/library/ast.html). It is a built-in
python package that enables users to interact with Python abstract
syntax tree inside of the process. You can use above snippet as it is
to import python files from anywhere.

I've decided to use the function signature shown below.

```python
config = read_config(config_file = '~/.config/AWindowManager/config',
        allowed_modules={'numpy': ['fft', 'zeros', 'arange'],
            'fft': ['fft'],
            'dataclasses': ['dataclass'],
            'os': ['path'],
            'path': ['join']},
        allowed_functions=['set', 'list'],
        verbose=True)
```

Using AST module, the function traverses syntax tree and checks each
node whether it is an allowed action. I think I have covered all
tricks to bypass the security checks (Of course don't use it in a
critical application, this is just an experiment with the AST module).
For example, you enabled `bad_module` with `good_action` function and
all functions from the `numpy` module (All functions from a module can
be enabled using `'numpy': ''`). The problem is, malicious
configuration can assign `bad_module` to `numpy` module. Now,
`bad_module` has numpy prefix and it can use everything from the
`bad_module` because every action in `numpy` module is allowed. To
prevent this I disabled assigning modules.  The code responsible to
disable that is shown below.

```python
def check(node_or_string, allowed_modules={}, allowed_functions=[],
        verbose=True):

    ...

    def _check(node):

        ...

        elif isinstance(node, ast.Assign):
            for target in node.targets:
                _check(target)
            _check(node.value)
            return
        elif isinstance(node, ast.Name):
            if node.id in aliases.keys():
                raise ValueError('Assigning modules is not allowed')
            return

        ...

        raise ValueError(f'malformed node or string: {node}')
    return [_check(node) for node in node_or_string]
```

This piece of code prevents doing these in a configuration file;
```python
np = os
a = os
np = a
a, os = b, np
```

There some other checks such as importing fake modules. For example a
file named numpy.py might contain malicious code.

inside of the numpy.py
```python
def zeros(*args, **kwargs):
    bad_function()
```

To prevent importing this, it makes sure only installed packages are imported.

I have named the package `mpcspy` which stands for my personal
configuration system for python.

Install: `pip3 install mpcspy`

Source Code: [Github MPCSPY](https://github.com/goktug97/mpcspy)

# An Example
## config file
```python
#!/usr/bin/env python3

import dataclasses
import numpy as np

@dataclasses.dataclass
class Robot(object):
    width: float = 1.2 # [m]
    height: float = 0.5 # [m]
    max_angular_velocity: float = np.radians(40.0) # [rad/s]
```
## importing
```python
import mpcspy
config = mpcspy.read_config(config_file = 'config',
        allowed_modules={'numpy': ['radians'],
            'dataclasses': ['dataclass']},
        allowed_functions=[],
        verbose=True)
print(config.Robot.width)
print(config.Robot.height)
print(config.Robot.max_angular_velocity)
```
