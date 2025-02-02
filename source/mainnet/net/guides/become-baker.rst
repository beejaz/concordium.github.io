.. _node-dashboard: http://localhost:8099
.. _Discord: https://discord.com/invite/xWmQ5tp

.. _become-a-baker:

==========================================
Become a baker using the Concordium Client
==========================================

This guide takes you through the steps involved in upgrading your node to a baker node and managing the node using the Concordium Client.

The process of becoming a baker involves the following:

#. Create an account in the Mobile Wallet and acquire CCD.
#. Create a set of baker keys.
#. Register the baker keys with the account.
#. Start the node with the baker keys.

After completing these steps, the baker node will bake blocks. If a baked block
is added to the chain, the baker of the node will receive a reward.

.. note::

   In this section the name ``bakerAccount`` indicates the name of the
   account that will be used to register and manage a baker.

Prerequisites
=============
-  Run a node
-  Create an account
-  Export the JSON file with the account information
-  Install the Concordium Client

For general information about baking concepts, see :ref:`baker-concept`.

Start baking
============

.. _import-baker-account:

Import the account
------------------

This section provides a brief description of how to import an account using the Concordium Client.

.. Note::

   You can only import accounts created in the Mobile Wallet into the Concordium Client. That is, you cannot import accounts created in the Desktop Wallet because they are created using a Ledger device. You get the account information by exporting a JSON file with the account information from the Mobile Wallet. For more information, see :ref:`Make a backup of identities and accounts in the Mobile Wallet <export-import>`.

To import an account run:

.. code-block:: console

   $concordium-client config account import <path/to/exported/file> --name bakerAccount

``concordium-client`` asks for a password to decrypt the exported file and
import all accounts. The same password will be used for encrypting the
transaction signing keys and the encrypted transfers key.

.. _create-register-baker-keys:

Create and register baker keys
------------------------------

Each account has a unique baker ID that is used when registering its baker. This ID has to be provided by the network and currently cannot be precomputed. This ID must be given inside the baker keys file to the node so that it can use the baker keys to create blocks. The ``concordium-client`` will automatically fill this field when performing the following operations.

To create a fresh set of keys run:

.. code-block:: console

   $concordium-client baker generate-keys <keys-file>.json

You can choose an arbitrary name for the ``<keys file>``. To register the keys in the network you need to be :ref:`running a node <running-a-node>`
and send a ``baker add`` transaction to the network:

.. code-block:: console

   $concordium-client baker add MyBakerKeys.json --sender bakerAccount --stake <amount-to-stake> --open-delegation-for all --delegation-transaction-fee-commission 0.1 --delegation-baking-commission 0.1 --delegation-finalization-commission 1.0 --baker-url https://example.com/baker --keys-in MyNewBakerKeys.json --keys-out <concordium-data-dir>/baker-credentials.json


where you replace

- ``MyBakerKeys.json`` with the name of baker keys file you generated
- ``<amount-to-stake>`` with the CCD amount for the baker's initial stake
- ``MyBakerURL`` with the URL to the json file containing information for your baker pool URL (optional)
- ``<concordium-data-dir>`` with any path of your choice.

   .. Note::

      For a node on Docker, the ``<concordium-data-dir>`` must use the following paths:

      * on Linux and MacOS: ``~/.local/share/concordium``
      * on Windows: ``%LOCALAPPDATA%\\concordium``.

      Remember to configure your node to bake using this path:

      * using Docker: See :ref:`configure Linux node<baking-docker>`.
      * on MacOS: See :ref:`configure MacOS node<baker-macos>`.
      * on Windows: See :ref:`configure Windows node<configure-baker-windows>`.
      * on Ubuntu: See :ref:`configure Ubuntu node<baker-ubuntu>`.

Keep the output file name as ``baker-credentials.json``.

The following arguments are also required for the ``baker add`` transaction:

- ``--open-delegation-for`` sets whether the baker's pool is open for delegators. Options are: ``none`` (no delegators will be allowed), ``all`` (any account can delegate), ``existing`` (only existing delegators can delegate).
- ``--baker-url`` is the URL for baker information. The URL should resolve to (JSON-formatted) metadata about the baker.
- ``--keys-in`` specifies the name of the file containing the baker keys.
- ``--keys-out`` can be used to write a baker credential file containing the baker ID (and the supplied keys) to use when starting a baker node.
- ``--delegation-transaction-fee-commission`` specifies the transaction fee commission for the baker pool.
- ``--delegation-baking-commission`` specifies the baking commission for the baker pool.
- ``--delegation-finalization-commission`` specifies the finalization commission for the baker pool.

.. Note::

   To find the range for the commissions, see the Concordium client :ref:`Show Chain Parameters command<consensus show-chain-parameters>`.

The following arguments are optional. If no selection is made, earnings are restaked automatically.

- ``--restake`` sets that earnings are restaked.
- ``--no-restake`` sets that earnings are not restaked. This flag means that rewards to the staked amount are not added on the baker automatically. Read more about this behavior in the section :ref:`Restake earnings<restake-earnings>`.

.. warning::

   ``concordium-client`` will offer to encrypt the generated ``baker-credentials.json`` file.
   Choose **not** to encrypt it since Concordium does not support easily starting a baker with encrypted baker credentials.
   If this is a firm requirement for you, then you need to run the :ref:`debian package<run-node-ubuntu>` and configure it appropriately.

.. Warning::
   Do not stake all of your funds or you will not have enough funds to cover transaction fees.

To start the node with these baker keys and bake blocks, configure the node to
use the baker keys, and **restart** it. The node will automatically start baking
when the baker is included in the bakers for the current epoch.

This change is executed immediately, and it will take effect when finishing the epoch after the one in which the transaction for adding the baker was included in a block.

.. table:: Timeline: adding a baker

   +-------------------------------------------+-----------------------------------------+-----------------+
   |                                           | When transaction is included in a block | After 2 epochs  |
   +===========================================+=========================================+=================+
   | Change is visible by querying the node    |  ✓                                      |                 |
   +-------------------------------------------+-----------------------------------------+-----------------+
   | Baker is included in the baking committee |                                         | ✓               |
   +-------------------------------------------+-----------------------------------------+-----------------+

.. note::

   If the transaction for adding the baker was included in a block during epoch `E`, the baker will be considered as part of the baking committee when epoch `E+2` starts.

Manage the baker
==================

Check the status of the baker and its lottery power
------------------------------------------------------

To see if the node is baking, you can check various sources that
offer different degrees of precision in the information displayed.

- In the `network dashboard <https://dashboard.mainnet.concordium.software/>`_, the baker ID of the node is shown in the ``Baker`` column.
- Using the ``concordium-client`` you can check the list of current bakers
  and the relative staked amount that they hold, that is, their lottery power.  The lottery power determines how likely it is that a given baker will win the lottery and bake a block.

  .. code-block:: console

     $concordium-client consensus show-parameters --include-bakers
     Election nonce:      07fe0e6c73d1fff4ec8ea910ffd42eb58d5a8ecd58d9f871d8f7c71e60faf0b0
     Election difficulty: 4.0e-2
     Bakers:
                                  Account                       Lottery power
             ----------------------------------------------------------------
         ...
         34: 4p2n8QQn5akq3XqAAJt2a5CsnGhDvUon6HExd2szrfkZCTD4FX   <0.0001
         ...

- Using the ``concordium-client`` you can check that the account has
  registered a baker and the current amount that is staked by that baker.

  .. code-block:: console

     $./concordium-client account show bakerAccount
     ...

     Baker: #22
      - Staked amount: 10.000000 CCD
      - Restake earnings: yes
     ...

- If the staked amount is high enough, and there is a node running with the baker keys loaded, the baker should eventually produce blocks. When this happens, you can see in your wallet that the account is receiving baking rewards.

Update the staked amount
------------------------

To update the baker stake run

.. code-block:: console

   $concordium-client baker update-stake --stake <newAmount> --sender bakerAccount

When the staked amount is modified, the probability that a baker gets elected
to bake blocks is also modified.

When a baker adds a stake for the first time or increase the stake, that
change is executed on the chain and becomes visible as soon as the transaction
is included in a block (can be seen through ``concordium-client account show
bakerAccount``) and takes effect 2 epochs after that.

.. table:: Timeline: increasing the stake

   +----------------------------------------+-----------------------------------------+----------------+
   |                                        | When transaction is included in a block | After 2 epochs |
   +========================================+=========================================+================+
   | Change is visible by querying the node | ✓                                       |                |
   +----------------------------------------+-----------------------------------------+----------------+
   | Baker uses the new stake               |                                         | ✓              |
   +----------------------------------------+-----------------------------------------+----------------+

When a baker **decreases the staked amount**, the change will need *2 +
bakerCooldownEpochs* epochs to take effect. The change becomes visible on the
chain as soon as the transaction is included in a block, it can be consulted through
``concordium-client account show bakerAccount``:

.. code-block:: console

   $concordium-client account show bakerAccount
   ...

   Baker: #22
    - Staked amount: 50.000000 CCD to be updated to 20.000000 CCD at epoch 261  (2020-12-24 12:56:26 UTC)
    - Restake earnings: yes

   ...

.. table:: Timeline: decreasing the stake

   +----------------------------------------+-----------------------------------------+----------------------------------------+
   |                                        | When transaction is included in a block | After *2 + bakerCooldownEpochs* epochs |
   +========================================+=========================================+========================================+
   | Change is visible by querying the node | ✓                                       |                                        |
   +----------------------------------------+-----------------------------------------+----------------------------------------+
   | Baker uses the new stake               |                                         | ✓                                      |
   +----------------------------------------+-----------------------------------------+----------------------------------------+
   | Stake can be decreased again or        | ✗                                       | ✓                                      |
   | baker can be removed                   |                                         |                                        |
   +----------------------------------------+-----------------------------------------+----------------------------------------+

.. note::

   In the Mainnet, ``bakerCooldownEpochs`` is set initially to 168 epochs. This
   value can be checked as follows:

   .. code-block:: console

      $concordium-client raw GetBlockSummary
      ...
              "bakerCooldownEpochs": 168
      ...

.. warning::

   The staked amount is *locked*. That is, you can't transfer it or use it for payment. You should take this into account and consider staking an amount that will not be needed in the short term. In particular, to deregister a baker or to modify the staked amount you need to own some non-staked CCD to cover the transaction costs.

   .. _restake-earnings:

Restake the earnings
----------------------

When participating as a baker in the network and baking blocks, the account
receives rewards on each baked block. These rewards are automatically added to
the staked amount by default.

You can choose to modify this behavior and instead receive the rewards in
the account balance without staking them automatically. You can change this switch through ``concordium-client``:

.. code-block:: console

   $concordium-client baker update-restake False --sender bakerAccount
   $concordium-client baker update-restake True --sender bakerAccount

Changes to the restake flag will take effect immediately; however, the changes
start affecting baking and finalizing power in the epoch after next. The current value of the switch can be seen in the account information which you can query using ``concordium-client``:

.. code-block:: console

   $concordium-client account show bakerAccount
   ...

   Baker: #22
    - Staked amount: 50.000000 CCD
    - Restake earnings: yes

   ...

.. table:: Timeline: updating restake

   +-----------------------------------------------+-----------------------------------------+-------------------------------+
   |                                               | When transaction is included in a block | 2 epochs after being rewarded |
   +===============================================+=========================================+===============================+
   | Change is visible by querying the node        | ✓                                       |                               |
   +-----------------------------------------------+-----------------------------------------+-------------------------------+
   | Earnings will [not] be restaked automatically | ✓                                       |                               |
   +-----------------------------------------------+-----------------------------------------+-------------------------------+
   | If restaking automatically, the gained        |                                         | ✓                             |
   | stake affects the lottery power               |                                         |                               |
   +-----------------------------------------------+-----------------------------------------+-------------------------------+

When the baker is registered, it will automatically restake the earnings, but you can change this by providing the ``--no-restake`` flag to
the ``baker add`` command as shown in the following:

.. code-block:: console

   $concordium-client baker add baker-keys.json --sender bakerAccount --stake <amount-to-stake> --out baker-credentials.json --no-restake

Finalization
------------

Finalization is the voting process performed by nodes in the *finalization
committee* that *finalizes* a block when a sufficiently big number of members of the committee have received the block and agree on its outcome. Newer blocks
must have the finalized block as an ancestor to ensure the integrity of the
chain. For more information about this process, see :ref:`finalization<glossary-finalization>`.

The finalization committee is formed by the bakers that have a certain staked
amount. This specifically implies that in order to participate in the
finalization committee you will probably have to modify the staked amount
to reach said threshold. In the Mainnet, the staked amount needed to participate
in the finalization committee is **0.1% of the total amount of existing CCD**.

Participating in the finalization committee produces rewards on each block that
is finalized. The rewards are paid to the baker account some time after the
block is finalized.

Update baker keys
-----------------

If it is necessary to update your baker keys, you need to first generate new baker keys. To create a fresh set of keys run:

.. code-block:: console

   $concordium-client baker generate-keys <keys-file>.json

You can choose an arbitrary name for the ``keys file``.

Then run the transaction:

.. code-block:: console

   $concordium-client baker set-key <keys-file>.json --sender <account> --out <concordium-data-dir>/baker-credentials.json

``--sender`` is the name or address of the transaction's sender account. The name is the one that's used when you :ref:`import the account<concordium-client-import-accounts-keys>` (assuming that this
was done). It defaults to the account name "default".

If you want to keep the ``baker-credentials.json`` output file in the same location as your other Concordium files, you can omit ``<concordium-data-dir>/``.

To start the node with these baker keys and bake blocks you need to restart the node.

Configure a baker
=================

Use ``baker configure`` to configure a baker and open a baker pool. The following is an example of how ``configure baker`` might be used:

.. code-block:: console

   $concordium-client baker configure --sender "acc1" --stake 14001 --open-delegation-for existing --delegation-transaction-fee-commission 0.1 --delegation-baking-commission 0.1 --delegation-finalization-commission 1.0 --baker-url https://example.com/baker --keys-in MyBakerKeys.json --keys-out <concordium-data-dir>/baker-credentials.json

Configure baker has the following optional arguments:

- ``--sender`` is the name or address of the baker account.
- ``--stake`` is an amount of CCD that is the intended equity capital of the baker
- ``--restake`` sets that earnings are restaked.
- ``--no-restake`` sets that earnings are not restaked.
- ``--open-delegation-for`` sets whether the baker's pool is open for delegators. Options are: ``none`` (no delegators will be allowed), ``all`` (any account can delegate), ``existing`` (only existing delegators can delegate).
- ``--baker-url`` is the URL for baker information. The URL should resolve to (JSON-formatted) metadata about the baker.
- ``--keys-in`` specifies the name of the file containing the baker keys.
- ``--keys-out`` can be used to write a baker credential file containing the baker ID (and the supplied keys) to use when starting a baker node. Replace ``<concordium-data-dir>`` with any path of your choice.
- ``--delegation-transaction-fee-commission`` specifies the transaction fee commission for the baker pool.
- ``--delegation-baking-commission`` specifies the baking commission for the baker pool.
- ``--delegation-finalization-commission`` specifies the finalization commission for the baker pool.

.. Note::

   To find the range for the commissions, see the Concordium client :ref:`Show Chain Parameters command<consensus show-chain-parameters>`.

Remove a baker
==============

The controlling account can choose to de-register its baker on the chain. To do
so you have to execute the ``concordium-client``:

.. code-block:: console

   $concordium-client baker remove --sender bakerAccount

This removes the baker from the baker list and unlocks the staked amount on
the baker so that it can be transferred or moved freely.

When removing the baker, the change has the same timeline as decreasing
the staked amount. The change will need *2 + bakerCooldownEpochs* epochs to take effect. The change becomes visible on the chain as soon as the transaction is included in a block and you can check when the change will be take effect by querying the account information with ``concordium-client``:

.. code-block:: console

   $concordium-client account show bakerAccount
   ...

   Baker #22 to be removed at epoch 275 (2020-12-24 13:56:26 UTC)
    - Staked amount: 20.000000 CCD
    - Restake earnings: yes

   ...

.. table:: Timeline: removing a baker

   +--------------------------------------------+-----------------------------------------+----------------------------------------+
   |                                            | When transaction is included in a block | After *2 + bakerCooldownEpochs* epochs |
   +============================================+=========================================+========================================+
   | Change is visible by querying the node     | ✓                                       |                                        |
   +--------------------------------------------+-----------------------------------------+----------------------------------------+
   | Baker is removed from the baking committee |                                         | ✓                                      |
   +--------------------------------------------+-----------------------------------------+----------------------------------------+

.. warning::

   Decreasing the staked amount and removing the baker can't be done
   simultaneously. During the cooldown period produced by decreasing the staked
   amount, the baker can't be removed and vice versa.
