sckit-glpk
----------

Proof of concept Python wrappers for GLPK.

Installation
------------

Should be an easy pip installation:

.. code-block::

   pip install scikit-glpk

A Python-compatible C compiler is required to build GLPK from source.  By default a precompiled
wheel is used during pip installation, so you don't have to compile if you don't want to.
Wheels are available for Linux, Mac, and Windows for supported versions of Python.

Usage
-----

There are a few things in this package:

- `glpk()` : the wrappers over the solvers (basically acts like Python-friendly `glpsol`)
- `mpsread()` : convert an MPS file to some matrices
- `mpswrite()` : convert matrices to MPS file
- `lpwrite()` : convert matrices to CPLEX LP file

.. code-block::

   from glpk import glpk, GLPK

   res = glpk(
       c, A_ub, b_ub, A_eq, b_eq, bounds, solver, sense, maxit, timeout,
       basis_fac, message_level, disp, simplex_options, ip_options,
       mip_options)

   from glpk import mpsread, mpswrite, lpwrite

   c, A_ub, b_ub, A_eq, b_eq, bounds = mpsread(
       filename, fmt=GLPK.GLP_MPS_FILE, ret_glp_prob=False)

   success = mpswrite(c, A_ub, b_ub, A_eq, b_eq, bounds, sense, filename, fmt)

   success = lpwrite(c, A_ub, b_ub, A_eq, b_eq, bounds, sense, filename)

There's lots of information in the docstrings for these functions, please check there for a complete listing and explanation.

Notice that `glpk` is the wrapper and `GLPK` acts as a namespace that holds constants.

`bounds` behaves the same as the `scipy.optimize.linprog`  `bounds` argument.  They are converted to GLPK-style bounds first thing.


GLPK stuffs
-----------

GLPK is installed with the module and a `linprog`-like wrapper is provided with a ctypes backend.  A pared-down version of glpk-4.65 is vendored from `here <http://ftp.gnu.org/gnu/glpk/>`_ and compile instructions are scraped from the makefiles.  I'll try my best to support cross-platform pip installations.  Wheels are current being built for Linux/Mac/Windows.


Background
----------

The `GNU Linear Programming Kit (GLPK) <https://www.gnu.org/software/glpk/>`_ has simplex, interior-point, and MIP solvers all callable from a C library.  We would like to be able to use these from within Python and be potentially included as a backend for scipy's `linprog` function.

Note that there are several projects that aim for something like this, but which don't match up for what I'm looking for:

- `python-glpk <https://www.dcc.fc.up.pt/~jpp/code/python-glpk/>`_ : no longer maintained (dead as of ~2013)
- `PyGLPK <http://tfinley.net/software/pyglpk/>`_ : GPL licensed
- `PyMathProg <https://pypi.org/project/pymprog/>`_ : GPL licensed, uses different conventions than that of `linprog`
- `Pyomo <https://github.com/Pyomo/pyomo>`_ : Big, uses different conventions than that of `linprog`
- `CVXOPT <https://cvxopt.org/>`_ : Big, GPL licensed
- `Sage <https://git.sagemath.org/sage.git/tree/README.md>`_ : Big, GPL licensed
- `pulp <https://launchpad.net/pulp-or>`_ : Calls `glpsol` from command line (writes problems, solutions to file instead of shared memory model -- this is actually easy to do)
- `yaposib <https://github.com/coin-or/yaposib>`_ : seems dead? OSI-centric
- `ecyglpki <https://github.com/equaeghe/ecyglpki/tree/0.1.0>`_ : GPL licensed, dead?
- `swiglpk <https://github.com/biosustain/swiglpk>`_ : GPL licensed, low level
- `optlang <https://github.com/biosustain/optlang>`_ : sympy-like, cool project otherwise

Why do we want this?
--------------------

GLPK has a lot of options that the current scipy solvers lack as well as robust MIP support (only basic in HiGHS).  It is also a standard, well known solver in the optimization community.  The only thing that I want that it lacks on an API level is robust support for column generation.  Easy access to GLPK as a backend to `linprog` would be very welcome (to me at least).  I also find access to a linprog LP description (c, A_ub, etc.) to MPS/LP file format convienent for interacting with other solvers such as `HiGHS <https://github.com/ERGO-Code/HiGHS>`_.

Approach
--------

Since the underlying API is quite simple and written in C and only C, `ctypes` is a good fit for this.

GLPK is packaged but I may want to make it so the user can optionally specify where the installation is on a user's computer (i.e., path to the shared library) so GLPK is not packaged with `scikit-glpk` and/or scipy.  `linprog` could then presumably route the problem to the GLPK backend instead of HiGHS or the existing native python solvers.

The `ctypes` wrapper is required for integrating GLPK into the Python runtime.  Instead of using MPS files to communicate problems and reading solutions from files, `scipy.sparse.coo_matrix` and `numpy` arrays can be passed directly to the library.  More information can be extracted from GLPK this way as well (For example, there is no way to get iteration count except by reading directly from the underlying structs.  It is only ever printed to stdout, no other way to get it).
