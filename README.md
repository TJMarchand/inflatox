![inflatox_banner](https://raw.githubusercontent.com/smups/inflatox/dev/logos/banner.png)
# Inflatox - multifield inflation consistency conditions in python
[![License: EUPL v1.2](https://img.shields.io/badge/License-EUPLv1.2-blue.svg)](https://joinup.ec.europa.eu/collection/eupl/eupl-text-eupl-12)
[![PyPi](https://img.shields.io/pypi/v/inflatox)](https://pypi.org/project/inflatox)
[![CI](https://github.com/smups/inflatox/actions/workflows/CI.yml/badge.svg)](https://github.com/smups/inflatox/actions/workflows/CI.yml)

Inflatox provides a framework to implement high-performance numerical consistency
conditions for multifield inflation models. As an example, an implementation of
the potential consistency condition for slow-roll rapid-turn two-field inflation
from Anguelova & Lazaroiu (2023)[^1] is built right into the package.

## Features
- symbolic solver for components of the Hesse matrix of an inflationary model
  with non-canonical kinetic terms, powered by [`sympy`](https://www.sympy.org).
- transpiler to transform `sympy` expressions into executable compiled (`C`) code
- built-in multithreaded `rust` module for high-performance calculations of
  consistency conditions that interfaces directly with `numpy` and python.
- no need to read, write or compile any `rust` or `C` code manually
  (this is all done automatically behind the scenes)
- no external dependencies, everything needed to run the package is included

## Installation and Dependencies
Inflatox requires at least python (ABI) version `3.7`. The latest version of
inflatox can be installed using pip:
```console
pip install inflatox
```
Inflatox can be updated using:
```console
pip install --upgrade inflatox
```

## Example programme
The following code example shows how `inflatox` can be used to calculate the
potential and components of the Hesse matrix for a two-field hyperinflation model.
```python
#import inflatox
import inflatox
import sympy as sp
import numpy as np
sp.init_printing()

#define model
φ, θ, L, m, φ0 = sp.symbols('φ θ L m φ0')
fields = [φ, θ]

V = (1/2*m**2*(φ-φ0)**2).nsimplify()
g = [
  [1, 0],
  [0, L**2 * sp.sinh(φ/L)**2]
]

#print metric and potential
display(g, V)

#symbolic calculation
calc = inflatox.SymbolicCalculation.new_from_list(fields, g, V)
hesse = calc.execute([[0,1]])

#run the compiler
out = inflatox.Compiler(hesse).compile()

#evaluate the compiled potential and Hesse matrix
from inflatox.consistency_conditions import AnguelovaLazaroiuCondition
anguelova = AnguelovaLazaroiuCondition(out)

args = np.array([1.0, 1.0, 1.0])
x = np.array([2.0, 2.0])
print(anguelova.calc_V(x, args))
print(anguelova.calc_H(x, args))
```

## Supported Architectures
- Intel/AMD x86/i686 (32 bit)
  - linux/gnu (glibc >= 2.17, kernel >= 3.2)
  - windows 7+ [^2]
- ARM armv7 (32 bit)
  - linux/gnu (glibc >= 2.17, kernel >= 3.2, hard float)
- Intel/AMD x86_64/amd64 (64 bit)
  - linux/gnu (glibc >= 2.17, kernel >= 3.2)
  - windows 7+ [^2]
  - macOS 10.12+ / Sierra+
- ARM aarch64 (64 bit)
  - linux/gnu (glibc >= 2.17, kernel >= 4.1)
  - macOS 11.0+ / Big Sur+
- PowerPC ppc64le (64 bit)
  - linux/gnu (glibc >= 2.17, kernel >= 3.10)
- IBM s390x (64 bit)
  - linux/gnu (glibc >= 2.17, kernel >= 3.2)
*Note: Apple silicon M-series chips are supported (aarch64)*

## License
[![License: EUPL v1.2](https://img.shields.io/badge/License-EUPLv1.2-blue.svg)](https://joinup.ec.europa.eu/collection/eupl/eupl-text-eupl-12)
>**Inflatox is explicitly not licensed under the dual
Apache/MIT license common to the Rust ecosystem. Instead it is licensed under
the terms of the [European Union Public License v1.2](https://joinup.ec.europa.eu/collection/eupl/eupl-text-eupl-12)**.

Inflatox is a science project and embraces the values of open science and free
and open software. Closed and paid scientific software suites hinder the
development of new technologies and research methods, as well as diverting much-
needed public funds away from researchers to large publishing and software
companies.

See the [LICENSE.md](../LICENSE.md) file for the EUPL text in all 22 official
languages of the EU, and [LICENSE-EN.txt](../LICENSE-EN.txt) for a plain text
English version of the license.

## References and footnotes
[^1]: Anguelova, L., & Lazaroiu, C. (2023). Dynamical consistency conditions for
  rapid-turn inflation. *Journal of Cosmology and Astroparticle Physics*,
  May 2023(20). https://doi.org/10.1088/1475-7516/2023/ 05/020
[^2]: Windows 7 is no longer considered a tier-1 target by the rust project. Usage
  of Windows 10+ is recommended.