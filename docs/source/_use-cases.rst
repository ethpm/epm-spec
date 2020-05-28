.. include:: _include.rst


The following use cases were considered during the creation of this
specification.

Stand Alone Package with an Inheritable Contract
   e.g. ``owned``

   See `full description <#stand-alone-package-with-an-inheritable-contract>`__.

Dependent Package with an Inheritable Contract
   e.g. ``transferable``

   See `full description <#dependent-package-with-an-inheritable-contract>`__.

Stand Alone Package with a Reusable Contract
   e.g. ``standard-token``

   See `full description <#stand-alone-package-with-a-reusable-contract>`__.

Stand Alone Package with a Deployed Contract
   e.g. ``safe-math-lib``

   See `full description <#stand-alone-package-with-a-deployed-contract>`__.

Dependent Package with a Reusable Contract
   e.g. ``piper-coin``

   See `full description <#dependent-package-with-a-reusable-contract>`__.

Stand Alone Package with a Deployed Contract Linked against a Deployed Library
   e.g. ``escrow``

   See `full description <#stand-alone-package-with-a-deployed-contract-linked-against-a-deployed-library>`__.

Dependent Package with a Deployed Contract Linked against a Package Dependency
   e.g. ``wallet``

   See `full description <#dependent-package-with-a-deployed-contract-linked-against-a-package-dependency>`__.

Dependent Package with a Deployed Contract Linked Against a Deep Dependency
   e.g. ``wallet-with-send``

   See `full description <#dependent-package-with-a-deployed-contract-linked-against-a-deep-dependency>`__.

*Each use case builds incrementally on the previous one.*

Compiler Metadata
    e.g. ``simple-auction``

    See `full description <#compiler-metadata>`__.

Compiler Input (generation &/or verification)
    e.g. ``simple-auction``

    See `full description <#standard-json>`__.


Keywords
--------

:Stand Alone:                
                              Package has no external dependencies (i.e. no ``buildDependencies``),
                              contains all contract data needed without reaching into another package.

:Dependent:                   Package does not contain all necessary contract data (i.e. has ``buildDependencies``),
                              **must** reach into a package dependency to retrieve data.

:Inheritable:                 Contract doesn't provide useful functionality on it's own and is meant 
                              to serve as a base contract for others to inherit from.

:Reusable:                    Contract is useful on it's own, meant to be used as-is.

:Deployed Contract/Library:   Refers to an instance of a contract/library that has already been
                              deployed to a specific address on a chain.

:Package Dependency:          External dependency directly referenced via the ``buildDependencies`` of a package.

:Deep Dependency:             External dependency referenced via the ``buildDependencies`` of a package dependency
                              (or by reaching down dependency tree as far as necessary).


.. _stand-alone-package-with-an-inheritable-contract:

Stand Alone Package with an Inheritable Contract
------------------------------------------------

For the first example we'll look at a package which only contains
contracts which serve as base contracts for other contracts to inherit
from but do not provide any real useful functionality on their own. The
common *owned* pattern is a example for this use case.

.. literalinclude:: ../../examples/owned/contracts/Owned.sol
   :language: solidity

..

   For this example we will assume this file is located in the solidity
   source file ``./contracts/owned.sol``

The ``owned`` package contains a single solidity source source file
which is intended to be used as a base contract for other contracts to
be inherited from. The package does not define any pre-deployed
addresses for the *owned* contract.

The smallest Package for this example looks like this:

.. code-block:: json

   {
     "manifest": "ethpm/3",
     "version": "1.0.0",
     "name": "owned",
     "sources": {
       "./contracts/owned.sol": {
         "installPath": "./contract/owned.sol",
         "type": "solidity",
         "urls": ["ipfs://QmUjYUcX9kLv2FQH8nwc3RLLXtU3Yv5XFpvEjFcAKXB6xD"]
       }
     }
   }

A Package which includes more than the minimum information would look
like this.

.. literalinclude:: ../../examples/owned/v3.json
   :language: json

This fully fleshed out Package is meant to demonstrate various pieces of
optional data that can be included. However, for the remainder of our
examples we will be using minimalistic Packages to keep our examples as
succinct as possible.

.. _dependent-package-with-an-inheritable-contract:

Dependent Package with an Inheritable Contract
----------------------------------------------

Now that we've seen what a simple package looks like, let's see how to declare
dependencies.

The next package will implement the *transferable* pattern and will
depend on our ``owned`` package for the authorization mechanism to
ensure that only the contract owner may transfer ownership. The
``transferable`` package will contain a single solidity source file
``./contracts/transferable.sol``.

.. literalinclude:: ../../examples/transferable/contracts/Transferable.sol
   :language: solidity

The EPM spec is designed to provide as high a guarantee as possible that
builds are deterministic and reproducable. To ensure that each package
you install gets the *exact* dependencies it needs, all dependencies are
declared as content addressed URIs. This ensures that when a package
manager fetches a dependency it always gets the right one.

The IPFS URI for the previous ``owned`` Package turns out to be
``ipfs://QmXDf2GP67otcF2gjWUxFt4AzFkfwGiuzfexhGuotGTLJH`` which is what
we will use in our ``transferable`` package to declare the dependency.
The Package looks like the following.

.. literalinclude:: ../../examples/transferable/v3.json
   :language: json

It will be up to the package management software to determine how the
``owned`` dependency actually gets installed as well as handling any
import remappings necessary to make the import statement work.

.. _stand-alone-package-with-a-reusable-contract:

Stand Alone Package with a Reusable Contract
--------------------------------------------

In this next example we'll look at a package which contains a reusable
contract. This means that the package provides a contract which can be
on its own in some manner. For this example we will be creating a
package which includes a reusable standard
`ERC20 <https://github.com/ethereum/EIPs/issues/20>`__ token contract.

   The source code for these contracts was pulled from the
   `SingularDTV <https://github.com/ConsenSys/singulardtv-contracts>`__
   github repository. Thanks to them for a very well written contract.

This package will contain two solidity source files:

    - `./contracts/AbstractToken.sol <https://github.com/ethpm/ethpm-spec/blob/master/examples/standard-token/contracts/AbstractToken.sol>`__
    - `./contracts/StandardToken.sol <https://github.com/ethpm/ethpm-spec/blob/master/examples/standard-token/contracts/StandardToken.sol>`__

Given that these source files are relatively large they will not be
included here within the guide but can be found in the
`./examples/standard-token/ <https://github.com/ethpm/ethpm-spec/tree/master/examples/standard-token>`__
directory within this repository.

Since this package includes a contract which may be used as-is, our
Package is going to contain additional information from our previous
examples, specifically, the ``contractTypes`` section. Since we expect
people to compile this contract theirselves we won't need to include any
of the contract bytecode, but it will be useful to include the contract
ABI and Natspec information. Our Package will look something like the
following. The contract ABI and NatSpec sections have been truncated to
improve legibility. The full Package can be found
`here <./examples/standard-token/1.0.0.json>`__

..
   from repo root, run:
      cat ./examples/standard-token/v3.json | jq '
         .contractTypes.StandardToken.abi |= ["..."] |
         .contractTypes.StandardToken.devdoc.methods |= {
            "allowance(address,address)": ."allowance(address,address)",
            "...": "..."
         }
      '

.. code-block:: json

   {
     "manifest": "ethpm/3",
     "version": "1.0.0",
     "name": "standard-token",
     "sources": {
       "./contracts/AbstractToken.sol": {
         "installPath": "./contracts/AbstractToken.sol",
         "type": "solidity",
         "urls": ["ipfs://QmQMXDprXxCunfQjA42LXZtzL6YMP8XTuGDB6AjHzpYHgk"]
       },
       "./contracts/StandardToken.sol": {
         "installPath": "./contracts/StandardToken.sol",
         "type": "solidity",
         "urls": ["ipfs://QmNLr7DzmiaQvk25C8bADBnh9bF5V3JfbwHS49kyoGGEHz"]
       }
     },
     "contractTypes": {
       "StandardToken": {
         "abi": [
           "..."
         ],
         "devdoc": {
           "author": "Stefan George - <stefan.george@consensys.net>",
           "title": "Standard token contract",
           "methods": {
             "allowance(address,address)": {
               "details": "Returns number of allowed tokens for given address.",
               "params": {
                 "_owner": "Address of token owner.",
                 "_spender": "Address of token spender."
               }
             },
             "...": "..."
           }
         }
       }
     }
   }


While it is not required to include the contract ABI and DevDoc
information, it does provide those using this package with the data
they would need to interact with an instance of this contract without
having to regenerate this information from source.

.. _stand-alone-package-with-a-deployed-contract:

Stand Alone Package with a Deployed Contract
--------------------------------------------

Now that we've seen what a package looks like which includes a fully
functional contract that is ready to be deployed, lets explore a package
that also includes a deployed instance of that contract.

Solidity Libraries are an excellent example of this type of package, so
for this example we are going to write a library for *safe* math
operations called ``safe-math-lib``. This library will implement
functions to allow addition and subtraction without needing to check for
underflow or overflow conditions. Our package will have a single
solidity source file ``./contracts/SafeMathLib.sol``

.. literalinclude:: ../../examples/safe-math-lib/contracts/SafeMathLib.sol
   :language: solidity

This will be our first package which includes the ``deployments``
section which is the location where information about deployed contract
instances is found. Lets look at the Package, some parts have been
truncated for readability but the full file can be found
`here <./examples/safe-math-lib/1.0.0.json>`__

..
   from repo root, run:
      cat ./examples/safe-math-lib/1.0.0.json | jq '
         .contractTypes.SafeMathLib.abi |= ["..."] |
         .contractTypes.SafeMathLib.devdoc |= {
            "title": .title,
            "author": .author,
            "...": "..."
         }
      '
.. code-block:: json

   {
     "manifest": "ethpm/3",
     "version": "1.0.0",
     "name": "safe-math-lib",
     "sources": {
       "./contracts/SafeMathLib.sol": {
         "installPath": "./contracts/SafeMathLib.sol",
         "type": "solidity",
         "urls": ["ipfs://QmVN1p6MmMLYcSq1VTmaSDLC3xWuAUwEFBFtinfzpmtzQG"]
       }
     },
     "compilers": [
       {
         "contractTypes": ["SafeMathLib"],
         "name": "solc",
         "version": "0.4.6+commit.2dabbdf0.Darwin.appleclang",
         "settings": {
           "optimize": true
         }
       },
     ],
     "contractTypes": {
       "SafeMathLib": {
         "deploymentBytecode": {
           "bytecode": "0x606060405234610000575b60a9806100176000396000f36504062dabbdf050606060405260e060020a6000350463a293d1e88114602e578063e6cb901314604c575b6000565b603a600435602435606a565b60408051918252519081900360200190f35b603a6004356024356088565b60408051918252519081900360200190f35b6000828211602a57508082036081566081565b6000565b5b92915050565b6000828284011115602a57508181016081566081565b6000565b5b9291505056"
         },
         "runtimeBytecode": {
           "bytecode": "0x6504062dabbdf050606060405260e060020a6000350463a293d1e88114602e578063e6cb901314604c575b6000565b603a600435602435606a565b60408051918252519081900360200190f35b603a6004356024356088565b60408051918252519081900360200190f35b6000828211602a57508082036081566081565b6000565b5b92915050565b6000828284011115602a57508181016081566081565b6000565b5b9291505056"
         },
         "abi": [
           "..."
         ],
         "devdoc": {
           "title": "Safe Math Library",
           "author": "Piper Merriam <pipermerriam@gmail.com>",
           "...": "..."
         }
       }
     },
     "deployments": {
       "blockchain://41941023680923e0fe4d74a34bdac8141f2540e3ae90623718e47d66d1ca4a2d/block/1e96de11320c83cca02e8b9caf3e489497e8e432befe5379f2f08599f8aecede": {
         "SafeMathLib": {
           "contractType": "SafeMathLib",
           "address": "0x8d2c532d7d211816a2807a411f947b211569b68c",
           "transaction": "0xaceef751507a79c2dee6aa0e9d8f759aa24aab081f6dcf6835d792770541cb2b",
           "block": "0x420cb2b2bd634ef42f9082e1ee87a8d4aeeaf506ea5cdeddaa8ff7cbf911810c"
         }
       }
     }
   }


The first thing to point out is that unlike our ``standard-token``
contract, we've included the ``bytecode``, ``runtimeBytecode`` and
``compiler`` information in the ``SafeMathLib`` section of the
``contractType`` definition. This is because we are also including a
deployed instance of this contract and need to require adequate
information for package managers to verify that the contract found at
the deployed address is in fact from the source code included in this
package.

The next thing to look at is the ``deployments`` section. The first
thing you should see is the
`BIP122 <https://github.com/bitcoin/bips/blob/master/bip-0122.mediawiki>`__
URI.

::

   blockchain://41941023680923e0fe4d74a34bdac8141f2540e3ae90623718e47d66d1ca4a2d/block/1e96de11320c83cca02e8b9caf3e489497e8e432befe5379f2f08599f8aecede

This URI defines the chain on which the ``SafeMathLib`` library was
deployed. The first hash you see,
``41941023680923e0fe4d74a34bdac8141f2540e3ae90623718e47d66d1ca4a2d`` is
the genesis block hash for the Ropsten test network. The later hash
``1e96de11320c83cca02e8b9caf3e489497e8e432befe5379f2f08599f8aecede`` is
the block hash for block number 168,238 from the Ropsten chain.

Under that URI there is a single |ContractInstance|. It specifies that
it's |ContractType| is ``SafeMathLib``, the address that the contract
instance can be found at, the transaction hash for the transaction that
deployed the contract, and the block hash in which the deploying
transaction was mined.

.. _dependent-package-with-a-reusable-contract:

Dependent Package with a Reusable Contract
------------------------------------------

For our next example we'll be creating a package includes a deployed
instance of a |ContractType| from that comes from a package dependency.
This differs from our previous ``safe-math-lib`` example where our
deployment is referencing a local contract from the local
``contractTypes``. In this package we will be referencing a
``contractType`` from one of the ``buildDependencies``

We are going to use the ``standard-token`` package we created earlier
and include a deployed version of the ``StandardToken`` contract.

Our package will be called ``piper-coin`` and will not contain any
source files since it merely makes use of the contracts from the
``standard-token`` package. The Package is listed below, and can be found at
`./examples/piper-coin/1.0.0.json <https://github.com/ethpm/ethpm-spec/blob/master/examples/piper-coin/1.0.0.json>`__

.. literalinclude:: ../../examples/piper-coin/v3.json
   :language: json

Most of this should be familiar but it's worth pointing out how we
reference |ContractTypes| from dependencies. Under the ``PiperCoin``
entry within the deployments you should see that the ``contractType``
key is set to ``standard-token:StandardToken``. The first portion
represents the name of the package dependency within the
``buildDependencies`` that should be used. The later portion indicates
the contract type that should be used from that dependencies contract
types.

.. _stand-alone-package-with-a-deployed-contract-linked-against-a-deployed-library:

Stand Alone Package with a Deployed Contract Linked against a Deployed Library
------------------------------------------------------------------------------

In the previous ``safe-math-lib`` package we demonstrated what a package
with a deployed instance of one of it's local contracts looks like. In
this example we will build on that concept with a package which includes
a library and a contract which uses that library as well as deployed
instances of both.

The package will be called ``escrow`` and will implementing a simple
escrow contract. The escrow contract will make use of a library to
safely send ether. Both the contract and library will be part of the
package found in the following two solidity source files.

-  `./contracts/SafeSendLib.sol <https://github.com/ethpm/ethpm-spec/blob/master/examples/escrow/contracts/SafeSendLib.sol>`__
-  `./contracts/Escrow.sol <https://github.com/ethpm/ethpm-spec/blob/master/examples/escrow/contracts/Escrow.sol>`__

The full source for these files can be found here:
`./examples/escrow/ <https://github.com/ethpm/ethpm-spec/tree/master/examples/escrow>`__.

The Package is listed below with some sections truncated for improved
readability. The full Package can be found at
`./examples/escrow/v3.json <https://github.com/ethpm/ethpm-spec/blob/master/examples/escrow/v3.json>`__

..
   from repo root, run:
      cat ./examples/escrow/v3.json | jq '
         .contractTypes.SafeSendLib |= {"...": "..."} |
         .contractTypes.Escrow |= {"...": "..."}
      '
.. code-block:: json

   {
     "manifest": "ethpm/3",
     "version": "1.0.0",
     "name": "escrow",
     "sources": {
       "./contracts/SafeSendLib.sol": {
         "installPath": "./contracts/SafeSendLib.sol",
         "type": "solidity",
         "urls": ["ipfs://QmcnzhWjaV71qzKntv4burxyix9W2yBA2LrJB4k99tGqkZ"]
       },
       "./contracts/Escrow.sol": {
         "installPath": "./contracts/Escrow.sol",
         "type": "solidity",
         "urls": ["ipfs://QmSwmFLT5B5aag485ZWvHmfdC1cU5EFdcqs1oqE5KsxGMw"]
       }
     },
     "contractTypes": {
       "SafeSendLib": {
         "...": "..."
       },
       "Escrow": {
         "...": "..."
       }
     },
     "deployments": {
       "blockchain://41941023680923e0fe4d74a34bdac8141f2540e3ae90623718e47d66d1ca4a2d/block/e76cf1f29a4689f836d941d7ffbad4e4b32035a441a509dc53150c2165f8e90d": {
         "SafeMathLib": {
           "contractType": "SafeSendLib",
           "address": "0x80d7f7a33e551455a909e1b914c4fd4e6d0074cc",
           "transaction": "0x74561167f360eaa20ea67bd4b4bf99164aabb36b2287061e86137bfa0d35d5fb",
           "block": "0x46554e3cf7b768b1cc1990ad4e2d3a137fe9373c0dda765f4db450cd5fa64102"
         },
         "Escrow": {
           "contractType": "Escrow",
           "address": "0x35b6b723786fd8bd955b70db794a1f1df56e852f",
           "transaction": "0x905fbbeb6069d8b3c8067d233f58b0196b43da7a20b839f3da41f69c87da2037",
           "block": "0x9b39dcab3d665a51755dedef56e7c858702f5817ce926a0cd8ff3081c5159b7f",
           "runtimeBytecode": {
             "linkDependencies": [
               {
                 "offsets": [
                   262,
                   412
                 ],
                 "type": "reference",
                 "value": "SafeSendLib"
               }
             ]
           }
         }
       }
     }
   }

This Package is the first one we've seen thus far that include the
``linkDependencies`` section within one of the |ContractInstances|.
The ``runtimeBytecode`` value for the ``Escrow`` contract has been
excluded from the example above for readability, but the full value is
as follows (wrapped to 80 characters).

..
   from repo root, run:
      cat ./examples/escrow/v3.json |
         jq '.contractTypes.Escrow.runtimeBytecode.bytecode' |
         sed -E 's/^"(.*)"$/\1/' | sed -E 's/(.{79})/\1 /g' | tr ' ' '\n'

::

   0x606060405260e060020a600035046366d003ac811461003457806367e404ce1461005d5780636
   9d8957514610086575b610000565b3461000057610041610095565b60408051600160a060020a03
   9092168252519081900360200190f35b34610000576100416100a4565b60408051600160a060020
   a039092168252519081900360200190f35b34610000576100936100b3565b005b600154600160a0
   60020a031681565b600054600160a060020a031681565b60005433600160a060020a03908116911
   6141561014857600154604080516000602091820152815160e260020a6324d048c7028152600160
   a060020a03938416600482015230909316316024840152905173000000000000000000000000000
   000000000000092639341231c926044808301939192829003018186803b156100005760325a03f4
   1561000057506101e2915050565b60015433600160a060020a039081169116141561002f5760008
   05460408051602090810193909352805160e260020a6324d048c7028152600160a060020a039283
   1660048201523090921631602483015251730000000000000000000000000000000000000000926
   39341231c9260448082019391829003018186803b156100005760325a03f41561000057506101e2
   915050565b610000565b5b5b56

This bytecode is unlinked, meaning it has missing portions of data, populated
instead with zeroes (``0000000000000000000000000000000000000000``).
This section of zeroes is referred to as a |LinkReference|. The
entries in the ``linkDependencies`` section of a |ContractInstance|
describe how these link references should be filled in.

The ``offsets`` value specifies the locations (as byte offset) where the
replacement should begin. The ``value`` defines what should be used to replace
the link reference. In this case, the ``value`` is referencing the
``SafeSendLib`` contract instance from this Package.

.. _dependent-package-with-a-deployed-contract-linked-against-a-package-dependency:

Dependent Package with a Deployed Contract Linked against a Package Dependency
------------------------------------------------------------------------------

Now that we've seen how we can handle linking dependencies that rely on
deployed |ContractInstances| from the local package we'll explore an
example with link dependencies that rely on contracts from the package
dependencies.

In this example we'll be writing the ``wallet`` package which includes a
wallet contract which makes use of the previous ``safe-math-lib``
package. We will also make use of the ``owned`` package from our very
first example to handle authorization. Our package will contain a single
solidity source file
`./contracts/Wallet.sol <https://github.com/ethpm/ethpm-spec/blob/master/examples/wallet/contracts/Wallet.sol>`__, shown below.


.. literalinclude:: ../../examples/wallet/contracts/Wallet.sol
   :language: solidity

The Package for our ``wallet`` package can been seen below. It has been
trimmed to improve readability. The full Package can be found at
`./examples/wallet/v3.json <https://github.com/ethpm/ethpm-spec/blob/master/examples/wallet/v3.json>`__

..
   from repo root, run:
      cat ./examples/wallet/1.0.0.json | jq '
         .contractTypes.Wallet |= {
            "deploymentBytecode": {"...": "..."},
            "runtimeBytecode": {"...": "..."},
            "...": "..."
          }
      '

.. code-block:: json

   {
     "manifest": "ethpm/3",
     "version": "1.0.0",
     "name": "wallet",
     "sources": {
       "./contracts/Wallet.sol": {
         "installPath": "./contracts/Wallet.sol",
         "type": "solidity",
         "urls": ["ipfs://QmYKibsXPSTR5UjywQHX8SM4za1K3QHadtFGWmZqGA4uE9"]
       }
     },
     "contractTypes": {
       "Wallet": {
         "deploymentBytecode": {
           "...": "..."
         },
         "runtimeBytecode": {
           "...": "..."
         },
         "...": "..."
       }
     },
     "deployments": {
       "blockchain://41941023680923e0fe4d74a34bdac8141f2540e3ae90623718e47d66d1ca4a2d/block/3ececfa0e03bce2d348279316100913c42ca2dcd51b8bc8d2d87ef2dc6a479ff": {
         "Wallet": {
           "contractType": "Wallet",
           "address": "0xcd0f8d7dab6c682d3726693ef3c7aaacc6431d1c",
           "transaction": "0x5c113857925ae0d866341513bb0732cd799ebc1c18fcec253bbc41d2a029acd4",
           "block": "0xccd130623ad3b25a357ead2ecfd22d38756b2e6ac09b77a37bd0ecdf16249765",
           "runtimeBytecode": {
             "linkDependencies": [
               {
                 "offsets": [
                   340
                 ],
                 "type": "reference",
                 "value": "safe-math-lib:SafeMathLib"
               }
             ]
           }
         }
       }
     },
     "buildDependencies": {
       "owned": "ipfs://QmUwVUMVtkVctrLDeL12SoeCPUacELBU8nAxRtHUzvtjND",
       "safe-math-lib": "ipfs://QmfUwis9K2SLwnUh62PDb929JzU5J2aFKd4kS1YErYajdq"
     }
   }

Just like our previous example, the ``runtimeBytecode`` has been
omitted for improved readability, but the full value is as follows
(wrapped to 80 characters).

..
   from repo root, run:
      cat ./examples/wallet/v3.json |
         jq '.contractTypes.Wallet.runtimeBytecode.bytecode' |
         sed -E 's/^"(.*)"$/\1/' | sed -E 's/(.{79})/\1 /g' | tr ' ' '\n'

::

   0x606060405236156100355760e060020a6000350463095ea7b381146100435780632e1a7d4d146
   1006a578063d0679d341461008e575b34610000576100415b5b565b005b34610000576100566004
   356024356100b5565b604080519115158252519081900360200190f35b346100005761005660043
   56100f8565b604080519115158252519081900360200190f35b3461000057610056600435602435
   6101da565b604080519115158252519081900360200190f35b6000805433600160a060020a03908
   1169116146100d157610000565b50600160a060020a038216600090815260016020819052604090
   91208290555b5b92915050565b600160a060020a033316600090815260016020908152604080832
   0548151830184905281517fa293d1e8000000000000000000000000000000000000000000000000
   0000000081526004810191909152602481018590529051730000000000000000000000000000000
   0000000009263a293d1e89260448082019391829003018186803b156100005760325a03f4156100
   00575050604080518051600160a060020a033316600081815260016020529384209190915592508
   4156108fc0291859190818181858888f1935050505015156101d157610000565b5060015b919050
   565b6000805433600160a060020a039081169116146101f657610000565b604051600160a060020
   a0384169083156108fc029084906000818181858888f19450505050505b5b9291505056

As you can see, this bytecode contains missing link references, in this case to
the ``SafeMathLib`` library from the ``safe-math-lib`` package dependency.
If you look in the ``linkDependencies`` section of our ``Wallet``
contract you'll see it's items are similar to the ones from our previous
example.

..
   from repo root, run:
      cat ./examples/wallet/v3.json | jq '
         .deployments | .. | .Wallet? |
            select(. != null) |
            .runtimeBytecode
      '
.. code-block:: json

   {
     "linkDependencies": [
       {
         "offsets": [
           340
         ],
         "type": "reference",
         "value": "safe-math-lib:SafeMathLib"
       }
     ]
   }

However, unlike the previous example which linked against a *local*
contract type, ``value`` portion is prefixed with the name of the
package which contains the address of the contract instance that this
should be linked against.

.. _dependent-package-with-a-deployed-contract-linked-against-a-deep-dependency:

Dependent Package with a Deployed Contract Linked against a Deep Dependency
---------------------------------------------------------------------------

In the previous example we saw how link dependencies against direct
package dependencies are handled. In this example we will look at how
this works when the link dependency is against a package deeper in the
dependency tree.

This package will contain a single solidity source file
`./contracts/WalletWithSend.sol <https://github.com/ethpm/ethpm-spec/blob/master/examples/wallet-with-send/contracts/WalletWithSend.sol>`__
which extends our previous ``Wallet`` contract, adding a new
``approvedSend`` function.

.. literalinclude:: ../../examples/wallet-with-send/contracts/WalletWithSend.sol
   :language: solidity

This new ``approvedSend`` function allows spending an address's provided
*allowance* by sending it to a specified address.

The Package for our ``wallet-with-send`` package can been seen below. It
has been trimmed to improve readability. The full Package can be found
at
`./examples/wallet-with-send/v3.json <https://github.com/ethpm/ethpm-spec/blob/master/examples/wallet-with-send/v3.json>`__

..
   from repo root, run:
      cat ./examples/wallet-with-send/v3.json | jq '
         .contractTypes.WalletWithSend |= {
            "deploymentBytecode": {"...": "..."},
            "runtimeBytecode": {"...": "..."},
            "...": "..."
          }
      '

.. code-block:: json

   {
     "manifest": "ethpm/3",
     "version": "1.0.0",
     "name": "wallet-with-send",
     "sources": {
       "./contracts/WalletWithSend.sol": {
         "installPath": "./contracts/WalletWithSend.sol",
         "type": "solidity",
         "urls": ["ipfs://QmWAKLzXaxES3tszDXDPP9xvf7xqsB9FU3W7MtapQ47naU"]
       }
     },
     "contractTypes": {
       "WalletWithSend": {
         "deploymentBytecode": {
           "...": "..."
         },
         "runtimeBytecode": {
           "...": "..."
         },
         "...": "..."
       }
     },
     "deployments": {
       "blockchain://41941023680923e0fe4d74a34bdac8141f2540e3ae90623718e47d66d1ca4a2d/block/bfb631d770940a93296e9b93f034f9f920ae311a8b37acd57ff0b55605beee73": {
         "Wallet": {
           "contractType": "WalletWithSend",
           "address": "0x7817d93f681a72758335398913136069c945d34b",
           "transaction": "0xe37d5691b58472f9932545d1d44fc95463a69adb1a3da5b06da2d9f3ff5c2939",
           "block": "0x5479e2f948184e13581d13e6a3d5bd5e5263d898d4514c5ec6fab37e2a1e9d6c",
           "runtimeBytecode": {
             "linkDependencies": [
               {
                 "offsets": [
                   383,
                   587
                 ],
                 "type": "reference",
                 "value": "wallet:safe-math-lib:SafeMathLib"
               }
             ]
           }
         }
       }
     },
     "buildDependencies": {
       "wallet": "ipfs://QmSg2QvGhQrYgQqbTGVYjGmF9hkEZrxQNmSXsr8fFyYtD4"
     }
   }

The important part of this Package is the ``linkDependencies`` section.

..
   from repo root, run:
      cat ./examples/wallet-with-send/v3.json | jq '
         .deployments | .. | .Wallet? |
            select(. != null) |
            .runtimeBytecode
      '
.. code-block:: json

   {
     "linkDependencies": [
       {
         "offsets": [
           383,
           587
         ],
         "type": "reference",
         "value": "wallet:safe-math-lib:SafeMathLib"
       }
     ]
   }

The ``value`` portion here means that the bytecode for this contract is
linked against the ``SafeMathLib`` deployed instance from the
``safe-math-lib`` dependency from the ``wallet`` dependency of this
package. This defines the traversal path through the dependency tree to
the deployed instance of the ``SafeMathLib`` library which was used for
linking during deployment.


.. _compiler-metadata:

Compiler Metadata
-----------------

`Compiler Metadata <>`__ output for a contract conforms to the EthPM spec,
making it easily extendable. ???? It should not include these fields (...)
and should include all other available contract assets.


# note this is pretty print...


