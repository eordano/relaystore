# relaystore

A simple web service that stores bitcoin transactions that are not yet ready to
be included in blocks.

## Installation

1. Run a bitcoind full node
1. Configure the RPC connection details in `config.js`
1. Run `npm start`

## How Does It Work

1. HTTP Endpoints:
  * GET /price: Returns a JSON encoded object with:
    - fee: the estimated fee per kilobyte of the bitcoind node
    - charge: number of satoshis the server is charging per transaction
    - address: a string where the service charge must be sent
  * POST /store: HTTP request body must be an hexa-encoded transaction
  * GET /stored/<txhash>: Returns transaction, hexa-encoded, if found in the
    storage.

1. Service is charged as one output that sends satoshis to a provided address

1. Transactions are dropped if:
  * At least one input has a sequence number different from 4294967295
  * nLockTime is in the past (the transaction will be broadcasted to the
    bitcoin network)
  * They are more than 2 kb in size
  * Transaction fees are ten times below what's currently estimated by the
    bitcoind node (`estimatefee`) for 6 confirmations.
  * One of the outputs that the transaction spends is already spent by another
    transaction included in a block with enough confirmations
  * One of the inputs is already spent by another stored transaction
  * As the outputs that the transaction spends are detected in blocks (with
    enough confirmations), the scriptSigs are verified, and the transaction is
    dropped if evaluation of the scripts fail.
  * Most of these restrictions are configurable

1. The submitted transaction is stored to disk, awaiting the time when it's
  possible to include it in a block.

1. If any output used by a transaction submitted doesn't get enough
  confirmations after 24 hours of being stored, the transaction is deleted.

1. A small implementation of IP request throttling is used. Sixty requests per
  hour per IP address is the default allowed.

## TODO List

1. Create tests to verify that the transaction storing/dropping is working
  correctly

1. Create integration test systems (really slow) to fully test the interactions
  of the system

1. Create a docker image to get this up & running in no time

1. Create a frontend page for the website

1. Integrate with a status check system

## Wishlist

1. BIP68/BIP112 Compatible replacement of transactions based on inputs
  with relative locktimes per input.

1. Nerdy nice to have: Use a "fidelity bond" based system to prevent spam
  and maybe replace the IP throttling mechanism.

1. Webhook to be configured when a transaction is dropped or broadcast.

## Known Issues / DoS

1. Signatures are not checked. This could cause a DoS against using a
  particular UTXO.
1. Error handling against conditions is not well tested. Bad transactions are
  correctly handled.

## License

MIT

## Credits

Author: Esteban Ordano

Supporters: https://www.lightlist.io/projects/relay-store-server-for-time-locked-transactions

