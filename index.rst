:tocdepth: 1
:numbered:

This document describes coding guidelines for using pybind11 within LSST.

  .. warning::
     Use of pybind11 within LSST code is not yet allowed (use Swig instead)!
     This document only serves to collect guidelines as part of DM-6168

  .. note::
     This document uses nomenclature defined
     `here <https://developer.lsst.io/coding/intro.html#stringency-levels>`_.

Modules and source files
========================

Wrapper modules SHALL be named after the header they wrap with an underscore prefix
-----------------------------------------------------------------------------------

If a module wraps ``header.h`` the wrapper implementation shall be in
``header.cc`` and the module name shall be ``_header``.

  .. note::
     Unlike what we currently have with Swig there is no Lib suffix.

.. _module-name:

Pure Python code SHALL go in a module named after the header without a pre or suffix
------------------------------------------------------------------------------------

So the pure Python code that extends functionality from ``_header`` goes in ``header.py``.

Pure Python code MAY go into a differently named file to avoid circular imports
-------------------------------------------------------------------------------

If circular imports would otherwise happen with regular :ref:`module naming <module-name>`
the pure Python code may go into another file.

There SHALL be one module for each wrapped header file
------------------------------------------------------

By wrapping into separate modules (to be combined in ``__init__``) we reduce
compilation and linking time.

Naming conventions
==================

Module object names SHALL be mod or camel case prefixed with mod
----------------------------------------------------------------

If a wrapper only contains one module the name of the object shall be
``mod``. Otherwise it shall be camel case prefixed with ``mod`` as in
``modExample``.

Class object names SHALL be cls or camel case prefixed with cls
---------------------------------------------------------------

If a wrapper only contains one class the name of the object shall be
``cls``. Otherwise it shall be camel case prefixed with ``cls`` as in
``clsExample``.

Lambda arguments refferring to the current object SHALL be named self
---------------------------------------------------------------------

For example::

        clsExample.def("f", [](Example const & self, ... ) { ... });

Wrapping templates
==================

Functions declaring template wrappers SHALL be prefixed with declare
--------------------------------------------------------------------

The wrapper for the templated type ``Example<T>`` shall be added by
a function with the following signature::

        template<typename T>
        static void declareExample(py::module & mod, std::string const & suffix)

The return type may be non-void in case more functionality needs to be
added later. The suffix may be ommitted when not needed.

Use of pybind11 features
========================

The default unique_ptr holder type SHALL be used where possible
---------------------------------------------------------------

By not specifying a holder type explicitly it becomes ``unique_ptr``.
This is to be preferred (over ``shared_ptr``).

.. _rule-3-2:

A shared_ptr holder type MAY be used where nessesary
----------------------------------------------------

Only to be used where required.

Litterals SHALL be used for all named arguments
-----------------------------------------------

The `_a` argument litteral, from `pybind11::litterals` shall be used
for all named arguments (e.g. ``mod.def("f", f, "arg1"_a, "arg2"_a);``).
The ``py::arg()`` construct SHALL NOT be used.

Default automatic conversions SHALL be used for all STL containers
------------------------------------------------------------------

The pybind11 header ``pybind11/stl.h`` provides automatic conversion
support (to standard Python ``list``, ``set``, ``tuple`` and ``dict`` types)
for most STL containers (i.e. ``std::vector``, ``std::set``, ``std::unordered_set``,
``std::pair``, ``std::tuple``, ``std::list``, ``std::map`` and ``std::unordered_map``).
These conversions shall always be used instead of manual wrapping.

Where copying of STL containers is undesirable an ndarray type SHALL be used instead
------------------------------------------------------------------------------------

The ``ndarray`` C++ types can share storage with NumPy arrays.

Default operator support SHALL NOT be used
------------------------------------------

Support from the ``pybind11/operators.h`` header cannot be applied consistently and
shall not be used.

Instead all operators are to be wrapped as lambda functions (the extra function call
is optimized away so there is no performance penalty).

For example::

        clsExample.def("__eq__", [](Example const & self, Example const & other) { return self == other; },
                py::is_operator());

Note that ``py::is_operator()`` is nesessary to get the correct ``NotImplemented`` return
when called with unsupported types.

Enums that require binary comparisons SHALL use py::arithmetic
--------------------------------------------------------------

If enums exposed to Python are used with bitwise comparison the desired operators shall not be added
manually. Instead ``py::enum_<ExampleEnum>(..., py::arithmetic())`` shall be used.

