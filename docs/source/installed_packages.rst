.. _installed-packages:

Using installed packages
========================

Packages installed with pip can declare that they support type
checking. For example, the `aiohttp
<https://docs.aiohttp.org/en/stable/>`_ package has built-in support
for type checking.

Packages can also provide stubs for a library. For example,
``types-requests`` is a stub-only package that provides stubs for the
`requests <https://requests.readthedocs.io/en/master/>`_ package.
Stub packages are usually published from `typeshed
<https://github.com/python/typeshed>`_, a shared repository for Python
library stubs, and have a name of form ``types-<library>``. Note that
many stub packages are not maintained by the original maintainers of
the package.

The sections below explain how mypy can use these packages, and how
you can create such packages.

.. note::

   :pep:`561` specifies how a package can declare that it supports
   type checking.

Using installed packages with mypy (PEP 561)
********************************************

Typically mypy will automatically find and use installed packages that
support type checking or provide stubs. This requires that you install
the packages in the Python environment that you use to run mypy.  As
many packages don't support type checking yet, you may also have to
install a separate stub package, usually named
``types-<library>``. (See :ref:`fix-missing-imports` for how to deal
with libraries that don't support type checking and are also missing
stubs.)

If you have installed typed packages in another Python installation or
environment, mypy won't automatically find them. One option is to
install another copy of those packages in the environment in which you
use to run mypy. Alternatively, you can use the
:option:`--python-executable <mypy --python-executable>` flag to point
to the target Python executable, and mypy will find packages installed
for that Python executable.

Note that mypy does not support some more advanced import features,
such as zip imports and custom import hooks.

If you don't want to use installed packages that provide type
information at all, use the :option:`--no-site-packages <mypy
--no-site-packages>` flag to disable searching for installed packages.

Note that stub-only packages cannot be used with ``MYPYPATH``. If you
want mypy to find the package, it must be installed. For a package
``foo``, the name of the stub-only package (``foo-stubs``) is not a
legal package name, so mypy will not find it, unless it is installed
(see :pep:`PEP 561: Stub-only Packages <561#stub-only-packages>` for
more information).

Creating PEP 561 compatible packages
************************************

.. note::

  You can generally ignore this section unless you maintain a package on
  PyPI, or want to publish type information for an existing PyPI
  package.

:pep:`561` describes three main ways to distribute type
information:

1. A package has inline type annotations in the Python implementation.

2. A package ships :ref:`stub files <stub-files>` with type
   information alongside the Python implementation.

3. A package ships type information for another package separately as
   stub files (also known as a "stub-only package").

If you want to create a stub-only package for an existing library, the
simplest way is to contribute stubs to the `typeshed
<https://github.com/python/typeshed>`_ repository, and a stub package
will automatically be uploaded to PyPI.

If you would like to publish a library package to a package repository
yourself (e.g. on PyPI) for either internal or external use in type
checking, packages that supply type information via type comments or
annotations in the code should put a ``py.typed`` file in their
package directory. For example, here is a typical directory structure:

.. code-block:: text

    setup.py
    package_a/
        __init__.py
        lib.py
        py.typed

The ``setup.py`` file could look like this:

.. code-block:: python

    from distutils.core import setup

    setup(
        name="SuperPackageA",
        author="Me",
        version="0.1",
        package_data={"package_a": ["py.typed"]},
        packages=["package_a"]
    )

.. note::

   If you use :doc:`setuptools <setuptools:index>`, you must pass the option ``zip_safe=False`` to
   ``setup()``, or mypy will not be able to find the installed package.

Some packages have a mix of stub files and runtime files. These packages also
require a ``py.typed`` file. An example can be seen below:

.. code-block:: text

    setup.py
    package_b/
        __init__.py
        lib.py
        lib.pyi
        py.typed

The ``setup.py`` file might look like this:

.. code-block:: python

    from distutils.core import setup

    setup(
        name="SuperPackageB",
        author="Me",
        version="0.1",
        package_data={"package_b": ["py.typed", "lib.pyi"]},
        packages=["package_b"]
    )

In this example, both ``lib.py`` and the ``lib.pyi`` stub file exist. At
runtime, the Python interpreter will use ``lib.py``, but mypy will use
``lib.pyi`` instead.

If the package is stub-only (not imported at runtime), the package should have
a prefix of the runtime package name and a suffix of ``-stubs``.
A ``py.typed`` file is not needed for stub-only packages. For example, if we
had stubs for ``package_c``, we might do the following:

.. code-block:: text

    setup.py
    package_c-stubs/
        __init__.pyi
        lib.pyi

The ``setup.py`` might look like this:

.. code-block:: python

    from distutils.core import setup

    setup(
        name="SuperPackageC",
        author="Me",
        version="0.1",
        package_data={"package_c-stubs": ["__init__.pyi", "lib.pyi"]},
        packages=["package_c-stubs"]
    )

If you have separate stubs for Python 2 and Python 3, you can place
the Python 2 stubs in a directory with the suffix ``-python2-stubs``.
We recommend that Python 2 and Python 3 stubs are bundled together for
simplicity, instead of distributing them separately.
