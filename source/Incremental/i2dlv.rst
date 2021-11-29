.. toctree::
   :maxdepth: 3
   :caption: Incremental Version 

   i2dlv

About
++++++++++++

* I²DLV is intended as the incremental version of the intelligent grounder I-DLV: as its standard version, I²DLV is an Answer Set Programming (ASP) instantiator that natively supports the `ASP-Core-2`_ standard language.

* The I²DLV instantiation mechanism allows repeated "shots" in which previous instantiations are reused and updated according to the so-called "overgrounding technique".

Download
=============

* You can download a beta release of I²DLV, `here <https://www.mat.unical.it/pacenza/storage/incremental-idlv/idlv-incremental.zip>`_.

How to execute
===================

It is possible to use the I²DLV system in different ways:

* Run the system without any command-line options. The system will wait for user input. The default incremental grounding policy is the one described in the TPLP2019 paper.

::

	./i2dlv

* Forces output to be printed in textual format (see all the available options below).

::
	
	./i2dlv -t

* Run the system enabling simplification and desimplification steps as described in the ICLP2020 submission. The system will wait for user input.

::

	./i2dlv --isd 

* Run the system with an associate XML file containing a set of instructions that has to be executed. The output will be printed in textual format.

::

	./i2dlv -t < file.xml

How to Cite I²DLV
=========================

* Francesco Calimeri, Giovambattista Ianni, Francesco Pacenza, Simona Perri, Jessica Zangari: `Incremental Answer Set Programming with Overgrounding. TPLP 2019: 957-973 <https://arxiv.org/abs/1907.09212>`_

* Giovambattista Ianni and Francesco Pacenza and Jessica Zangari: `Incremental maintenance of overgrounded logic programs with tailored simplifications. TPLP 2020: 719-734 <https://arxiv.org/abs/2008.04108>`_

.. _IncrementalSyntax:

Syntax
+++++++++++

Command
====================

XML Command Scripts
------------------------

* I²DLV is intended to be executed as a process which is kept alive in memory. Users can communicate with the process by simply piping XML commands specifying the desired tasks. A permanent in-memory instantiation can be created, updated and piped to a given ASP solver.

Currently, the following XML commands are available:

Load Command
_________________

* The tag ``load`` can be used to form an XML element that requests to load input programs or data; the admitted attribute is:

* ``path`` a file path pointing to the data to be loaded, in `ASP-Core-2`_ format.

	Example of load tag::
	
		<load path="./recursion.0.asp"/>

Ground Command
_________________

* The tag ``ground`` can be used to form an XML statement that instructs I²DLV to ground/update the current logic program and, possibly, send the current instantiation to a solver; the admitted attributes are:

* ``run_mode``, which can be set to the following 3 values:

	#. ``print``: the current ground program is updated and the result is printed on stdout (default);

	#. ``updateonly``: the current ground program is updated in memory and no output is provided;

	#. ``solve``: the current ground program is updated in memory and, then, piped to a solver. By default, the ground program is sent to the ``wasp`` solver (it is assumed that ``wasp`` is reachable in the default user ``PATH``). The solver output will be printed on stdout.

		* ``solve_with``: the default solver and its command-line options can be overriden when this value is specified.

Example of ground tag:

* I²DLV updates the ground program and prints the result on stdout::

	<ground/>

* I²DLV produces the ground program without printing the result::

	<ground run_mode="updateonly"/>

* I²DLV produces the ground and pipes the result to a solver (``wasp`` in this example).

.. note:: Note that ``--mode=wasp`` and ``--printonlyoptimum`` are options of the ``dlv2`` system.

::

	<ground run_mode="solve" solve_with="./dlv2 --mode=wasp --printonlyoptimum"/>

Reset Command
_________________

* The tag ``reset`` can be used to form an XML element that requests to reset all data loaded so far. No attribute is admitted.

Example of reset tag::

	<reset/>

Exit command
_________________


* The tag ``exit`` can be used to form an XML element that requests to close the process. No attribute is admitted.

Example of exit tag::

	<exit/>

Command-line Options
-------------------------

Grounding Options
______________________

* ``--isd``, ``--incremental-simpl-desimpl`` enable simplification and desimplification steps during grounding.

Output Options
______________________

* ``-t``, ``--textual`` prints in textual mode (same behaviour of ``--output=1``), instead of the default numeric.

* ``--filter`` filters the specified predicates with the specified arity. 
	
	Example::

		--filter=p1/2,p2/3.

* ``--print-rewriting`` prints in STDERR the rewritten program as preprocessed by IDLV.

Statistics Options
______________________

* ``--time`` the system prints the grounding time of each rule.

* ``--istats`` the system displays incremental grounding statistics.

General Options
______________________


* ``--help`` the system prints this guide and exit.

* ``--stdin`` the system reads input from standard input (default).

* ``--mode`` set the execution mode:

	* ``0 = Console``: the user provides commands via standard input.

	* ``1 = Server``: the user provides commands via a connection over the IP and port numbers at which the system is reachable.

	.. note:: By default ``--mode`` is set to ``0``.

* ``--port`` set the port number in server mode (by default, it is set to ``4790``).

.. _IncrementalExample:

Example
+++++++++++++++

* A zip archive with a full working example can be downloaded, `here <https://www.mat.unical.it/pacenza/storage/incremental-idlv/idlv-incremental.zip>`_. The File included in the zip archive are listed in the following.

* The test folder containing::

	recursion/recursion.*.asp: ASP files given in input to I2DLV.
	disjunction/disjunction.*.asp: ASP files given in input to I-DLV Incremental.
	template.xml: it is an XML file containing all the instructions that have to be executed by I2DLV.

* solver folder containing::

	dlv2: binaries of systems that integrate an ASP solver.

Disjunction
=================

How to execute
----------------

::

	./idlv-incremental/i2dlv < test/disjunction/template.xml


Recursion
=============

How to execute
----------------

::

	./idlv-incremental/i2dlv < test/recursion/template.xml


.. _ASP-Core-2: https://arxiv.org/abs/1911.04326 
.. _DLV: https://dlv.demacs.unical.it   