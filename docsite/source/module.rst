.. _module:

.. currentmodule:: bitmath

The ``bitmath`` Module
######################

.. contents::
   :depth: 3
   :local:

Functions
*********

This section describes utility functions included in the bitmath
module.

bitmath.getsize()
=================
.. function:: bitmath.getsize(path[, bestprefix=True[, system=NIST]])

   Return a bitmath instance representing the size of a file at any
   given path.

   :param string path: The path of a file to read the size of
   :param bool bestprefix: **Default:** ``True``, the returned
                           instance will be in the best human-readable
                           prefix unit. If set to ``False`` the result
                           is a ``bitmath.Byte`` instance.
   :param system: **Default:** ``bitmath.NIST``. The preferred system
                  of units for the returned instance.
   :type system: One of ``bitmath.NIST`` or ``bitmath.SI``

   Internally :py:func:`bitmath.getsize` calls
   :py:func:`os.path.realpath` before calling
   :py:func:`os.path.getsize` on any paths.

   Here's an example of where we'll run :py:func:`bitmath.getsize` on
   the bitmath source code using the defaults for ``bestprefix`` and
   ``system``:

   .. code-block:: python

      >>> import bitmath
      >>> print bitmath.getsize('./bitmath/__init__.py')
      33.3583984375 KiB

   Let's say we want to see the results in bytes. We can do this by
   setting ``bestprefix`` to ``False``:

   .. code-block:: python

      >>> import bitmath
      >>> print bitmath.getsize('./bitmath/__init__.py', bestprefix=False)
      34159.0 Byte

   Recall, the default for representation is with the best
   human-readable prefix. We can control the prefix system used by
   setting ``system`` to either :py:data:`bitmath.NIST` (the default)
   or :py:data:`bitmath.SI`:

   .. code-block:: python
      :linenos:
      :emphasize-lines: 1-4

      >>> print bitmath.getsize('./bitmath/__init__.py')
      33.3583984375 KiB
      >>> print bitmath.getsize('./bitmath/__init__.py', system=bitmath.NIST)
      33.3583984375 KiB
      >>> print bitmath.getsize('./bitmath/__init__.py', system=bitmath.SI)
      34.159 kB

   We can see in lines **1** → **4** that the same result is returned
   when ``system`` is not set and when ``system`` is set to
   :py:data:`bitmath.NIST` (the default).

   .. versionadded:: 1.0.7

bitmath.listdir()
=================

.. function:: bitmath.listdir(search_base[, followlinks=False[, filter='*'[, relpath=False[, bestprefix=False[, system=NIST]]]]])

   This is a `generator
   <https://docs.python.org/2/tutorial/classes.html#generators>`_
   which recurses a directory tree yielding 2-tuples of:

   * The absolute/relative path to a discovered file
   * A bitmath instance representing the *apparent size* of the file

   :param string search_base: The directory to begin walking down
   :param bool followlinks: **Default:** ``False``, do not follow
                            links. Whether or not to follow symbolic
                            links to directories. Setting to ``True``
                            enables directory link following
   :param string filter: **Default:** ``*`` (everything). A glob to
                         filter results with. See `fnmatch
                         <https://docs.python.org/2/library/fnmatch.html>`_
                         for more details about *globs*
   :param bool relpath: **Default:** ``False``, returns the fully
                        qualified to each discovered file. ``True`` to
                        return the relative path from the present
                        working directory to the discovered file. If
                        ``relpath`` is ``False``, then
                        :py:func:`bitmath.listdir` internally calls
                        :py:func:`os.path.realpath` to normalize path
                        references
   :param bool bestprefix: **Default:** ``False``, returns
                           ``bitmath.Byte`` instances. Set to ``True``
                           to return the best human-readable prefix
                           unit for representation
   :param system: **Default:** ``bitmath.NIST``. Set a prefix
                  preferred unit system. Requires ``bestprefix`` is
                  ``True``
   :type system: One of ``bitmath.NIST`` or ``bitmath.SI``

   .. note::

      * This function does **not** return tuples for directory
        entities. Including directories in results is `scheduled for
        introduction <https://github.com/tbielawa/bitmath/issues/27>`_
        in the upcoming 1.1.0 release.
      * Symlinks to **files** are followed automatically


   When interpreting the results from this function it is *crucial* to
   understand exactly which items are being taken into account, what
   decisions were made to select those items, and how their sizes are
   measured.

   Results from this function may seem invalid when directly compared
   to the results from common command line utilities, such as ``du``,
   or ``tree``.

   Let's pretend we have a directory structure like the following::

      some_files/
      ├── deeper_files/
      │   └── second_file
      └── first_file

   Where ``some_files/`` is a directory, and so is
   ``some_files/deeper_files/``. There are two regular files in this
   tree:

   * ``somefiles/first_file`` - 1337 Bytes
   * ``some_files/deeper_files/second_file`` - 13370 Bytes

   The **total** size of the files in this tree is **1337 + 13370 =
   14707** bytes.

   Let's call :py:func:`bitmath.listdir` on the ``some_files/``
   directory and see what the results look like. First we'll use all
   the default parameters, then we'll set ``relpath`` to ``True``:

   .. code-block:: python
      :linenos:
      :emphasize-lines: 5-6,10-11

      >>> import bitmath
      >>> for f in bitmath.listdir('./some_files'):
      ...     print f
      ...
      ('/tmp/tmp.P5lqtyqwPh/some_files/first_file', Byte(1337.0))
      ('/tmp/tmp.P5lqtyqwPh/some_files/deeper_files/second_file', Byte(13370.0))
      >>> for f in bitmath.listdir('./some_files', relpath=True):
      ...     print f
      ...
      ('some_files/first_file', Byte(1337.0))
      ('some_files/deeper_files/second_file', Byte(13370.0))

   On lines **5** and **6** the results print the full path, whereas
   on lines **10** and **11** the path is relative to the present
   working directory.

   Let's play with the ``filter`` parameter now. Let's say we only
   want to include results for files whose name begins with "second":

   .. code-block:: python

      >>> for f in bitmath.listdir('./some_files', filter='second*'):
      ...     print f
      ...
      ('/tmp/tmp.P5lqtyqwPh/some_files/deeper_files/second_file', Byte(13370.0))


   If we wish to avoid having to write for-loops, we can collect the
   results into a list rather simply:

   .. code-block:: python

      >>> files = list(bitmath.listdir('./some_files'))
      >>> print files
      [('/tmp/tmp.P5lqtyqwPh/some_files/first_file', Byte(1337.0)), ('/tmp/tmp.P5lqtyqwPh/some_files/deeper_files/second_file', Byte(13370.0))]

   Here's a more advanced example where we will sum the size of all
   the returned results and then play around with the possible
   formatting. Recall that a bitmath instance representing the size of
   the discovered file is the second item in each returned tuple.

   .. code-block:: python

      >>> discovered_files = [f[1] for f in bitmath.listdir('./some_files')]
      >>> print discovered_files
      [Byte(1337.0), Byte(13370.0)]
      >>> print reduce(lambda x,y: x+y, discovered_files)
      14707.0 Byte
      >>> print reduce(lambda x,y: x+y, discovered_files).best_prefix()
      14.3623046875 KiB
      >>> print reduce(lambda x,y: x+y, discovered_files).best_prefix().format("{value:.3f} {unit}")
      14.362 KiB


   .. versionadded:: 1.0.7

.. _module_context_managers:

Context Managers
****************

This section describes all of the `context managers
<https://docs.python.org/2/reference/datamodel.html#context-managers>`_
provided by the bitmath class.

.. note::

   For a bit of background, a *context manager* (specifically, the
   ``with`` statement) is a feature of the Python language which is
   commonly used to:

   * Decorate, or *wrap*, an arbitrary block of code. I.e., effect a
     certain condition onto a specific body of code

   * Automatically *open* and *close* an object which is used in a
     specific context. I.e., handle set-up and tear-down of objects in
     the place they are used.

.. seealso::

   :pep:`343`
      *The "with" Statement*

   :pep:`318`
      *Decorators for Functions and Methods*


.. _module_bitmath_format:

bitmath.format()
================

.. function:: bitmath.format([fmt_str=None[, plural=False[, bestprefix=False]]])

   The :py:func:`bitmath.format` context manager allows you to specify
   the string representation of all bitmath instances within a
   specific block of code.

   This is effectively equivalent to applying the
   :ref:`format()<instances_format>` method to an entire region of
   code.

   :param str fmt_str: a formatting mini-language compat formatting
                       string. See the :ref:`instances attributes
                       <instance_attributes>` for a list of available
                       items.
   :param bool plural: ``True`` enables printing instances with
                       trailing **s**'s if they're plural. ``False``
                       (default) prints them as singular (no trailing
                       's')
   :param bool bestprefix: ``True`` enables printing instances in
                           their best human-readable
                           representation. ``False``, the default,
                           prints instances using their current prefix
                           unit.


   .. note:: The ``bestprefix`` parameter is not yet implemented!

   Let's look at an example of toggling pluralization on and
   off. First we'll look over a demonstration script (below), and then
   we'll review the output.

   .. code-block:: python
      :linenos:
      :emphasize-lines: 33-34

      import bitmath

      a_single_bit = bitmath.Bit(1)
      technically_plural_bytes = bitmath.Byte(0)
      always_plural_kbs = bitmath.kb(42)

      formatting_args = {
          'not_plural': a_single_bit,
          'technically_plural': technically_plural_bytes,
          'always_plural': always_plural_kbs
      }

      print """None of the following will be pluralized, because that feature is turned off
      """

      test_string = """   One unit of 'Bit': {not_plural}

         0 of a unit is typically said pluralized in US English: {technically_plural}

         several items of a unit will always be pluralized in normal US English
         speech: {always_plural}"""

      print test_string.format(**formatting_args)

      print """
      ----------------------------------------------------------------------
      """

      print """Now, we'll use the bitmath.format() context manager
      to print the same test string, but with pluralization enabled.
      """

      with bitmath.format(plural=True):
          print test_string.format(**formatting_args)

   The context manager is demonstrated in lines **33** → **34**. In
   these lines we use the :py:func:`bitmath.format` context manager,
   setting ``plural`` to ``True``, to print the original string
   again. By doing this we have enabled pluralized string
   representations (where appropriate). Running this script would have
   the following output::


      None of the following will be pluralized, because that feature is turned off

         One unit of 'Bit': 1.0 Bit

         0 of a unit is typically said pluralized in US English: 0.0 Byte

         several items of a unit will always be pluralized in normal US English
         speech: 42.0 kb

      ----------------------------------------------------------------------

      Now, we'll use the bitmath.format() context manager
      to print the same test string, but with pluralization enabled.

         One unit of 'Bit': 1.0 Bit

         0 of a unit is typically said pluralized in US English: 0.0 Bytes

         several items of a unit will always be pluralized in normal US English
         speech: 42.0 kbs

   Here's a shorter example, where we'll:

   * Print a string containing bitmath instances using the default
     formatting (lines **2** → **3**)
   * Use the context manager to print the instances in scientific
     notation (lines **4** → **7**)
   * Print the string one last time to demonstrate how the formatting
     automatically returns to the default format (lines **8** → **9**)

   .. code-block:: python
      :linenos:

      >>> import bitmath
      >>> print "Some instances: %s, %s" % (bitmath.KiB(1 / 3.0), bitmath.Bit(512))
      Some instances: 0.333333333333 KiB, 512.0 Bit
      >>> with bitmath.format("{value:e}-{unit}"):
      ...     print "Some instances: %s, %s" % (bitmath.KiB(1 / 3.0), bitmath.Bit(512))
      ...
      Some instances: 3.333333e-01-KiB, 5.120000e+02-Bit
      >>> print "Some instances: %s, %s" % (bitmath.KiB(1 / 3.0), bitmath.Bit(512))
      Some instances: 0.333333333333 KiB, 512.0 Bit


   .. versionadded:: 1.0.8


.. _module_class_variables:

Module Variables
****************

This section describes the module-level variables. Some of which are
constants and are used for reference. Some of which effect output or
behavior.

.. versionchanged:: 1.0.7 The formatting strings were not available
   for manupulate/inspection in earlier versions

.. note:: Modifying these variables will change the default
          representation indefinitely. Use the
          :py:func:`bitmath.format` context manager to limit
          changes to a specific block of code.

.. _module_format_string:

.. py:data:: bitmath.format_string

   This is the default string representation of all bitmath
   instances. The default value is ``{value} {unit}`` which, when
   evaluated, formats an instance as a floating point number with at
   least one digit of precision, followed by a character of
   whitespace, followed by the prefix unit of the instance.

   For example, given bitmath instances representing the following
   values: **1337 MiB**, **0.1234567 kb**, and **0 B**, their printed
   output would look like the following:

   .. code-block:: python

      >>> from bitmath import *
      >>> print MiB(1337), kb(0.1234567), Byte(0)
      1337.0 MiB 0.1234567 kb 0.0 Byte

   We can make these instances print however we want to. Let's wrap
   each one in square brackets (``[``, ``]``), replace the separating
   space character with a hyphen (``-``), and limit the precision to
   just 2 digits:

   .. code-block:: python

      >>> import bitmath
      >>> bitmath.format_string = "[{value:.2f}-{unit}]"
      >>> print bitmath.MiB(1337), bitmath.kb(0.1234567), bitmath.Byte(0)
      [1337.00-MiB] [0.12-kb] [0.00-Byte]

.. py:data:: bitmath.format_plural

   A boolean which controls the pluralization of instances in string
   representation. The default is ``False``.

   If we wanted to enable pluralization we could set the
   :py:data:`format_plural` variable to ``True``. First, let's look at
   some output using the default singular formatting.

   .. code-block:: python

      >>> import bitmath
      >>> print bitmath.MiB(1337)
      1337.0 MiB

   And now we'll enable pluralization (line **2**):

   .. code-block:: python
      :linenos:
      :emphasize-lines: 2,5

      >>> import bitmath
      >>> bitmath.format_plural = True
      >>> print bitmath.MiB(1337)
      1337.0 MiBs
      >>> bitmath.format_plural = False
      >>> print bitmath.MiB(1337)
      1337.0 MiB

   On line **5** we disable pluralization again and then see that the
   output has no trailing "s" character.

.. py:data:: bitmath.NIST

   Constant used as an argument to some functions to specify the
   **NIST** system.

.. py:data:: bitmath.SI

   Constant used as an argument to some functions to specify the
   **SI** system.

.. py:data:: bitmath.SI_PREFIXES

   An array of all of the SI unit prefixes (e.g., ``k``, ``M``, or
   ``E``)

.. py:data:: bitmath.SI_STEPS

   .. code-block:: python

      SI_STEPS = {
          'Bit': 1 / 8.0,
          'Byte': 1,
          'k': 1000,
          'M': 1000000,
          'G': 1000000000,
          'T': 1000000000000,
          'P': 1000000000000000,
          'E': 1000000000000000000
      }


.. py:data:: bitmath.NIST_PREFIXES

   An array of all of the NIST unit prefixes (e.g., ``Ki``, ``Mi``, or
   ``Ei``)


.. py:data:: bitmath.NIST_STEPS

   .. code-block:: python

      NIST_STEPS = {
          'Bit': 1 / 8.0,
          'Byte': 1,
          'Ki': 1024,
          'Mi': 1048576,
          'Gi': 1073741824,
          'Ti': 1099511627776,
          'Pi': 1125899906842624,
          'Ei': 1152921504606846976
      }
