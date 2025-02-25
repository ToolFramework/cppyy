.. _repositories:

Repositories
============

The ``cppyy`` module is a frontend that requires an intermediate (Python
interpreter dependent) layer, and a backend (see
:ref:`Package Structure <package-structure>`).
Because of this layering and because it leverages several existing packages
through reuse, the relevant codes are contained across a number of
repositories.

* Frontend, cppyy: https://github.com/wlav/cppyy
* CPython (v2/v3) intermediate: https://github.com/wlav/CPyCppyy
* PyPy intermediate (module _cppyy): https://foss.heptapod.net/pypy
* Backend, cppyy: https://github.com/wlav/cppyy-backend

The backend repo contains both the cppyy-cling (under "cling") and
cppyy-backend (under "clingwrapper") packages.


.. _building_from_source:

Building from source
--------------------

Except for cppyy-cling, the structure in the repositories follows a normal
PyPA package and they are thus ready to build with `setuptools`_: simply
clone the package and either run ``python setup.py``, or use ``pip``.

It is highly recommended to follow the dependency chain when manually
upgrading packages individually (i.e. ``cppyy-cling``, ``cppyy-backend``,
``CPyCppyy`` if on CPython, and then finally ``cppyy``), because upstream
packages expose headers that are used by the ones downstream.
Of course, if only building for a patch/point release, there is no need to
re-install the full chain (or follow the order).
Always run the local updates from the package directories (i.e. where the
``setup.py`` file is located), as some tools rely on the package structure.

The ``STDCXX`` envar can be used to control the C++ standard version; use
``MAKE`` to change the ``make`` command; and ``MAKE_NPROCS`` to control the
maximum number of parallel jobs.
Compilation of the backend, which contains a customized version of
Clang/LLVM, can take a long time, so by default the setup script will use all
cores (x2 if hyperthreading is enabled).

On MS Windows, some temporary path names may be too long, causing the build to
fail.
To resolve this issue, point the ``TMP`` and ``TEMP`` envars to an existing
directory with a short name before the build:
For example::

 > set TMP=C:\TMP
 > set TEMP=C:\TMP

Start with the ``cppyy-cling`` package (cppyy-backend repo, subdirectory
"cling"), which requires source to be pulled in from upstream, and thus takes
a few extra steps::

 $ git clone https://github.com/wlav/cppyy-backend.git
 $ cd cppyy-backend/cling
 $ python setup.py egg_info
 $ python create_src_directory.py
 $ python -m pip install . --upgrade

The ``egg_info`` setup command is needed for ``create_src_directory.py`` to
find the right version.
That script in turn downloads the proper release from `upstream`_, trims and
patches it,
and installs the result in the "src" directory.
When done, the structure of ``cppyy-cling`` looks again like a PyPA package
and can be used/installed as expected, here using ``pip``.

The ``cppyy-cling`` package, because it contains Cling/Clang/LLVM, is rather
large to build, so by default the setup script will use all cores (x2 if
hyperthreading is enabled).
You can change this behavior with the ``MAKE_NPROCS`` envar.
The wheel of ``cppyy-cling`` is reused by pip for all versions of CPython and
PyPy, thus the long compilation is needed only once for all different
versions of Python on the same machine.

Next up is ``cppyy-backend`` (cppyy-backend, subdirectory "clingwrapper"; omit
the first step if you already cloned the repo for ``cppyy-cling``)::

 $ git clone https://github.com/wlav/cppyy-backend.git
 $ cd cppyy-backend/clingwrapper
 $ python -m pip install . --upgrade --no-use-pep517 --no-deps

Note the use of ``--no-use-pep517``, which prevents ``pip`` from needlessly
going out to pypi.org and creating a local "clean" build environment from the
cached or remote wheels.
Instead, by skipping PEP 517, the local installation will be used.
This is imperative if there was a change in public headers or if the version
of ``cppyy-cling`` was locally updated and is thus not available on PyPI.

Upgrading ``CPyCppyy`` (if on CPython; it's not needed for PyPy) and ``cppyy``
is very similar::

 $ git clone https://github.com/wlav/CPyCppyy.git
 $ cd CPyCppyy
 $ python -m pip install . --upgrade --no-use-pep517 --no-deps

Finally, the top-level package ``cppyy``::

 $ git clone https://github.com/wlav/cppyy.git
 $ cd cppyy
 $ python -m pip install . --upgrade --no-use-pep517 --no-deps

Please see the `pip documentation`_ for more options, such as developer mode.

.. _`setuptools`: https://setuptools.readthedocs.io/
.. _`upstream`: https://root.cern.ch/download/
.. _`pip documentation`: https://pip.pypa.io/
