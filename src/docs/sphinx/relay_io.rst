.. ############################################################################
.. # Copyright (c) 2014-2019, Lawrence Livermore National Security, LLC.
.. # 
.. # Produced at the Lawrence Livermore National Laboratory
.. # 
.. # LLNL-CODE-666778
.. # 
.. # All rights reserved.
.. # 
.. # This file is part of Conduit. 
.. # 
.. # For details, see: http://software.llnl.gov/conduit/.
.. # 
.. # Please also read conduit/LICENSE
.. # 
.. # Redistribution and use in source and binary forms, with or without 
.. # modification, are permitted provided that the following conditions are met:
.. # 
.. # * Redistributions of source code must retain the above copyright notice, 
.. #   this list of conditions and the disclaimer below.
.. # 
.. # * Redistributions in binary form must reproduce the above copyright notice,
.. #   this list of conditions and the disclaimer (as noted below) in the
.. #   documentation and/or other materials provided with the distribution.
.. # 
.. # * Neither the name of the LLNS/LLNL nor the names of its contributors may
.. #   be used to endorse or promote products derived from this software without
.. #   specific prior written permission.
.. # 
.. # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
.. # AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
.. # IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE
.. # ARE DISCLAIMED. IN NO EVENT SHALL LAWRENCE LIVERMORE NATIONAL SECURITY,
.. # LLC, THE U.S. DEPARTMENT OF ENERGY OR CONTRIBUTORS BE LIABLE FOR ANY
.. # DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL 
.. # DAMAGES  (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS
.. # OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
.. # HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, 
.. # STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING
.. # IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE 
.. # POSSIBILITY OF SUCH DAMAGE.
.. # 
.. ############################################################################

===================
Relay I/O
===================

Conduit Relay I/O provides optional Silo, HDF5, and ADIOS I/O interfaces. 

These interfaces can be accessed through a generic path-based API, generic handle class, or through APIs specific to each underlying I/O interface.  The specific APIs provide lower level control and allow reuse of handles, which is more efficient for most non-trivial use cases. The generic handle class strikes a balance between usability and efficiency. 


.. _relay_io_generic_interface:

Relay I/O Path-based Interface
--------------------------------

The path-based Relay I/O interface allows you to read and write conduit::Nodes using any enabled I/O interface through a simple path-based (string) API. The underlying I/O interface is selected using the extension of the destination path or an explicit protocol argument.


The ``conduit_relay`` library provides the following methods in the ``conduit::relay::io`` namespace:

 * ``relay::io::save`` 
 
   * Saves the contents of the passed Node to a file. Works like a ``Node::set`` to the file: if the file exists, it is overwritten to reflect contents of the passed Node. 
 
 * ``relay::io::save_merged`` 
 
   * Merges the contents of the passed Node to a file. Works like a ``Node::update`` to the file: if the file exists, new data paths are appended, common paths are overwritten, and other existing paths are not changed. 


 * ``relay::io::load``  
    
   * Loads the contents of a file into the passed Node. Works like a ``Node::set`` from the contents of the file: if the Node has existing data, it is overwritten to reflect contents of the file.
 
 * ``relay::io::load_merged`` 
   
   * Merges the contents of a file into the passed Node. Works like a ``Node::update`` rom the contents of the file: if the Node has existing data, new data paths are appended, common paths are overwritten, and other existing paths are not changed. 

                             
The ``conduit_relay_mpi_io`` library provides the ``conduit::relay::mpi::io`` namespace which includes variants of these methods which take a MPI Communicator. These variants pass the communicator to the underlying I/O interface to enable collective I/O. Relay currently only supports collective I/O for ADIOS.


Relay I/O Path-based Interface Examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Save and Load
+++++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_generic_examples.cpp
   :lines: 93-107
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_generic_examples_out.txt
   :lines: 41-65


Save Merged 
+++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_generic_examples.cpp
   :lines: 119-140
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_generic_examples_out.txt
   :lines: 73-110


Load Merged 
+++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_generic_examples.cpp
   :lines: 150-168
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_generic_examples_out.txt
   :lines: 118-155

Load from Subpath
+++++++++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_generic_examples.cpp
   :lines: 180-193
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_generic_examples_out.txt
   :lines: 163-179


Save to Subpath
+++++++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_generic_examples.cpp
   :lines: 204-217
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_generic_examples_out.txt
   :lines: 187-203


.. things not yet covered: options

Relay I/O Handle Interface
----------------------------

The ``relay::io::IOHandle`` class provides a high level interface to query, read, and modify files.

It provides a generic interface that is more efficient than the path-based interface for protocols like HDF5 which support partial I/O and querying without reading the entire contents of a file.
It also supports simpler built-in protocols (conduit_bin, json, etc) that do not support partial I/O for convenience. Its basic contract is that changes to backing (file on disk, etc) are not guaranteed to be reflected until the handle is closed. Relay I/O Handle does not yet support Silo or ADIOS. 

IOHandle has the following instance methods: 

 * ``open``
   
   * Opens a handle. The underlying I/O interface is selected using the extension of the destination path or an explicit protocol argument.

  .. DANGER::
    Note: While you can read from and write to subpaths using a handle, IOHandle *does not* yet support opening a file with a subpath (e.g. ``myhandle.open("file.hdf5:path/data")``).


 * ``read``
   
   * Merges the contents from the handle or contents from a subpath of the handle into the passed Node. Works like a ``Node::update`` from the handle: if the Node has existing data, new data paths are appended, common paths are overwritten, and other existing paths are not changed. 


 * ``write``
 
   * Writes the contents of the passed Node to the handle or to a subpath of the handle. Works like a ``Node::update`` to the handle: if the handle has existing data, new data paths are appended, common paths are overwritten, and other existing paths are not changed. 

 * ``has_path``
 
   * Checks if the handle contains a given path.

 * ``list_child_names``
 
   * Returns a list of the child names at a given path, or an empty list if the path does not exist.

 * ``remove``
 
   * Removes any data at and below a given path. With HDF5 the space may not be fully reclaimed.

 * ``close``
   
   * Closes a handle. This is when changes are realized to the backing (file on disc, etc).


Relay I/O Handle Examples
~~~~~~~~~~~~~~~~~~~~~~~~~~
* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_handle_examples.cpp
   :lines: 64-132
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_handle_examples_out.txt
   :lines: 9-45

* **Python Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_tutorial_python_relay_io_handle_examples.py
   :lines: 65-111
   :language: python
   :dedent: 8

* **Output:**

.. literalinclude:: t_conduit_docs_python_tutorial_relay_io_handle_examples_out.txt
   :lines: 28-64
   :dedent: 4

Relay I/O HDF5 Interface
---------------------------

The Relay I/O HDF5 interface provides methods to read and write Nodes using HDF5 handles.
It is also the interface used to implement the path-based and handle I/O interfaces for 
HDF5. This interface provides more control and allows more efficient reuse of I/O handles.
It is only available in C++.

Relay I/O HDF5 Interface Examples
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is a example exercising the basic parts of Relay I/O's HDF5 interface, for
more detailed documentation see the ``conduit_relay_io_hdf5_api.hpp`` header file. 

HDF5 I/O Interface Basics 
++++++++++++++++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_hdf5_examples.cpp
   :lines: 66-115
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_hdf5_examples_out.txt
   :lines: 9-41



HDF5 I/O Options 
++++++++++++++++++

* **C++ Example:**

.. literalinclude:: ../../tests/docs/t_conduit_docs_relay_io_hdf5_examples.cpp
   :lines: 128-160
   :language: cpp
   :dedent: 4

* **Output:**

.. literalinclude:: t_conduit_docs_relay_io_hdf5_examples_out.txt
   :lines: 49-107

You can verify using ``h5stat`` that the data set was written to the hdf5 file using chunking and
compression.


