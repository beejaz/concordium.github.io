.. _list of types implementing the SchemaType: https://docs.rs/concordium-contracts-common/latest/concordium_contracts_common/schema/trait.SchemaType.html#foreign-impls
.. _build-schema-v0:

=======================
Build a contract schema
=======================

This guide will show you how to build a smart contract schema, how to export it
to a file, and/or embed the schema into the smart contract module, all using
``cargo-concordium``.

Preparation
===========

First, ensure you have ``cargo-concordium`` installed and if not the guide
:ref:`setup-tools-v0` will help you.

We also need the Rust source code of the smart contract you wish to build a
schema for.

Setup the contract for a schema
===============================

In order to build a contract schema, we first have to prepare our smart
contract for building the schema.

We can choose which parts of our smart contract to included in the schema.
The options are to include a schema for the contract state, and/or for each of
the parameters of init and receive functions.

Every type we want to include in the schema must implement the ``SchemaType``
trait. This is already done for all base types and some other types (see `list of types implementing the SchemaType`_).
For most other cases, it can also be achieved automatically, using
``#[derive(SchemaType)]``::

   #[derive(SchemaType)]
   struct SomeType {
       ...
   }

Implementing the ``SchemaType`` trait manually only requires specifying one
function, which is a getter for a ``schema::Type``, which essentially describes
how this type is represented as bytes and how to represent it.

.. todo::

   Create an example showing how to manually implement ``SchemaType`` and link
   to it from here.

Including contract state
------------------------

To generate and include the schema for the contract state, we annotate the type
with the ``#[contract_state(contract = ...)]`` macro::

   #[contract_state(contract = "my_contract")]
   #[derive(SchemaType)]
   struct MyState {
       ...
   }

Or even simpler if the contract state is of a type that already implements ``SchemaType``, e.g., u32::

   #[contract_state(contract = "my_contract")]
   type State = u32;

Including function parameters
-----------------------------

To generate and include the schema for parameters for init  and
receive functions, we set the optional ``parameter`` attribute for the
``#[init(..)]``- and ``#[receive(..)]``-macro::

   #[derive(SchemaType)]
   enum InitParameter { ... }

   #[derive(SchemaType)]
   enum ReceiveParameter { ... }

   #[init(contract = "my_contract", parameter = "InitParameter")]
   fn contract_init<...> (...){ ... }

   #[receive(contract = "my_contract", name = "my_receive", parameter = "ReceiveParameter")]
   fn contract_receive<...> (...){ ... }

Building the schema
===================

Now, we are ready to build the actual schema using ``cargo-concordium``, and we
have the options to embed the schema and/or write the schema to a file.

.. seealso::

   For more on which to choose see
   :ref:`here<contract-schema-which-to-choose-v0>`.

Embedding the schema
--------------------

In order to embed the schema into the smart contract module, we add
``--schema-embed`` to the build command

.. code-block:: console

   $cargo concordium build --schema-embed

If successful the output of the command will tell you the total size of the
schema in bytes.

Outputting a schema file
------------------------

To output the schema into a file, we can use the ``--schema-out=FILE``
where ``FILE`` is a path of the file to create:

.. code-block:: console

   $cargo concordium build --schema-out="/some/path/schema.bin"
