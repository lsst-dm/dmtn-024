:tocdepth: 1

This document describes coding guidelines for using pybind11 within LSST.

  .. warning::
     Use of pybind11 within LSST code is not yet allowed (use Swig instead)!
     This document only serves to collect guidelines as part of DM-6168

  .. note::
     This document uses nomenclature defined `here <https://developer.lsst.io/coding/intro.html#stringency-levels>`_.

1. Modules and source files
===========================

.. _rule-1-1:

1-1. Wrapper modules SHOULD be named after the header they wrap with an underscore prefix
-----------------------------------------------------------------------------------------

If a module wraps ``header.h`` the wrapper implementation should be in
``header.cc`` and the module name should be ``_header``.

  .. note::
     Unlike what we currently have with Swig there is no Lib suffix.

.. _rule-1-2:

1-2. Pure Python code SHOULD go in a module named after the header without a pre or suffix
------------------------------------------------------------------------------------------

So the pure Python code that extends functionality from ``_header`` goes in ``header.py``.

.. _rule-1-3:

1-3. Pure Python code MAY go into a differently named file to avoid circular imports
------------------------------------------------------------------------------------

If circular imports would otherwise happen when applying :ref:`1-2 <rule-1-2>` the pure Python code may go
into another file.

.. _rule-1-4:

1-4. There SHOULD be one module for each wrapped header file
------------------------------------------------------------

By wrapping into separate modules (to be combined in ``__init__``) we reduce
compilation and linking time.

2. Naming conventions
=====================

.. _rule-2-1:

2-1. Module object names SHOULD be mod or camel case prefixed with mod
----------------------------------------------------------------------

If a wrapper only contains one module the name of the object should be
``mod``. Otherwise it should be camel case prefixed with ``mod`` as in
``modExample``.

.. _rule-2-2:

2-2. Class object names SHOULD be cls or camel case prefixed with cls
---------------------------------------------------------------------

If a wrapper only contains one class the name of the object should be
``cls``. Otherwise it should be camel case prefixed with ``cls`` as in
``clsExample``.

3. Holder types
===============

.. _rule-3-1:

3-1. The default unique_ptr holder type SHOULD be used where possible
---------------------------------------------------------------------

By not specifying a holder type explicitly it becomes ``unique_ptr``.
This is to be preferred (over ``shared_ptr``).

.. _rule-3-2:

3-2. A shared_ptr holder type MAY be used where nessesary
---------------------------------------------------------

Only to be used where required.
