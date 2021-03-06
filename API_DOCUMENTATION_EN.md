### [Blockchair.com](https://blockchair.com/) API v.2.0.0 Documentation

#### Migrating from API v.1 to v.2

There are many breaking changes, please read through the documentation.

* Until September 1st, 2018, `api.blockchair.com` will serve API v.1
* Until September 1st, 2018, `alpha-2.blockchair.com` will serve API v.2 in beta test mode
* Beginning September 1st, 2018, `api.blockchair.com` will serve API v.2 in production mode
* Beginning September 1st, 2018, `api-v1.blockchair.com` will serve API v.1 (we guarantee support for that for at least a month)

Please test your application and make needed changes. If you have any questions, please don't hesitate to contact us.

##### Global Changes

* Unified output for all requests (a server response always returns a JSON array consisting of two sub-arrays: `data` with requested data, and `context` with metadata)
* Corrected types of output values (e.g. integers are always integers, and not strings)
* Query execution speed is increased by 3-10 times
* There are lots of new dashboard calls!

##### API updates for working with blockchains (`bitcoin[-cash]/[mempool/](blocks|transactions|outputs)`, `ethereum/[mempool/](blocks|uncles|transactions|calls)`)

* Available up to 2 simultaneous sortings, which can be enumerated by commas (example: `?s=field1(asc|desc),field2(asc|desc)`)
* Available up to 5 simultaneous filters 
* The `?limit=N` section now allows you to specify `N` from 1 to 100. The default is 10. Note: The `LIMIT` section is ignored in some cases, in such cases `context.limit` will yeild `null`
* Added operator `...` for a strict comparison, e.g. `?q=id(1...3)` returns only 2, `?q=id(1...)` returns all matches starting with 2
* Added `OR` and `NOT` operators
* Cancelled `?next=N` and `?next_sort=M` sections, now the `OFFSET` section is used, e.g.: use `?offset =10` to get the next 10 records after the standard limit of 10. The maximum value of `OFFSET` is 100,000.
* Context information moved to a special `context` subarray
* Many other new features

#### General Provisions

* Requests to our server should be made through the HTTPS protocol by GET requests to the domain `api.blockchair.com`

* The server response returns a JSON array always consisting of two subarrays:
	* `data` - contains some data
	* `context` - contains metadata, e.g., a status code, a query execution time, and so on.
	
* `data` can contain either an associative array (e.g., for `bitcoin/stats`), or for infinitable-queries (see below) - an unnumbered array (for example, `bitcoin/blocks`), or for dashboard-queries (see below) - an associative array with keys which are parts of the query (e.g., for `bitcoin/dashboards/transactions/A,B`, the keys are `A` and `B`), and with values ​- arrays of data.
* `context`, depending on the call type, can contain the following useful values:
	* `context.code` - server response code, can return:
		* `200` if the request succeeds
		* `400` if there is a user error in the request
		* `402` if any limit on the number or complexity of requests is exceeded (see the restrictions below, please contact us at <info@blockchair.com> for private API access)
		* `500` or` 503` in case of a server error (it makes sense to wait and repeat the same request or open a ticket at https://github.com/Blockchair/Blockchair.Support/issues/new or write to <info@blockchair.com>)
	* There is a field `context.error` with an error description in the case of` 40x` and `50x` errors
	* `context.state` contains the number of the latest block in case of requests to the blockchains (for example, for all requests beginning with `bitcoin` there will be the latest block number for Bitcoin). It is useful in order, e.g., to calculate the number of network сonfirmations, or correctly iterate trough the results using `offset`
	* `context.results` - contains the number of found results for dashboard-calls
	* `context.limit` - applied limit to the number of results
	* `context.offset` - applied offset
	* `context.rows` - ​​contains the number of returned rows for infinitable-calls 
	* `context.pre_rows` - ​​for some infinitable-calls contains the number of rows that should've been returned before duplicate removal (note: this architecture is used for the `bitcoin[-cash].outputs` tables only)
	* `context.total_rows` - ​​number of rows that a query returns
	* `context.api` - an array of data on the status of the API:
		* `context.api.version` - version of API
		* `context.api.last_major_update` - time of the last update, that somehow broke backward compatibility
		* `context.api.next_major_update` - time of the next scheduled update, that can break compatibility, or` null`, if no updates are scheduled

Note: it makes sense to check `context.api.version` and, if `context.api.next_major_update` is not `null`, notify yourself and review the changelog. If there are no changes in the changelog that violate the compatibility of your application, make sure that the value of `context.api.next_major_update` won't exceed the current one. If there are changes, adjust the application logic so, that after the specified time a new logic is applied. Additional note: in case the backward compatibility is violated only for one API call, then `context.api.next_major_update` won't be `null` just for this call.

* Request limits: as of now, we allow up to 30 requests per minute to our API. If this limit is exceeded, an error `402` will be returned. In case of abuse, your IP address can be blocked. If your application needs more requests, please contact us at <info@blockchair.com>. If you need to unload a large amount of information once, please contact us at <export@blockchair.com> and, in case the academic or research unloading goal - you will receive the data for free in a convenient format.

* Disclaimer: we do not guarantee the reliability or integrity of the provided information. Information provided by our API should not be used for making critical decisions. We do not guarantee an uptime for our free API.

#### Infinitable Calls (blockhain tables)

Return data from the tables according to the filters (`q`), sorting (`s`), limit (`limit`), and offset (`offset`).

A request should be construced like this: `https://api.blockchair.com/{blockchain}/[/mempool]{table}[?q={query}][&s={sorting}][&limit={limit}][&offset={offset}]`

**Possible combinations of blockchains and tables:**
* Bitcoin:
    * `bitcoin/blocks` - contains all Bitcoin blocks, including the latest one
    * `bitcoin/transactions` - contains all Bitcoin transactions, excluding mempool transactions and transactions from the latest block   
    * `bitcoin/outputs` - contains all Bitcoin outputs, excluding the outputs contained in the mempool transactions as well as in the transactions from the latest block
    * `bitcoin/mempool/blocks` - contains only the latest Bitcoin block
    * `bitcoin/mempool/transactions` - contains Bitcoin mempool transactions as well as transactions from the latest block
	* `bitcoin/mempool/outputs` - contains Bitcoin outputs included in mempool transactions as well as in transactions from the latest block
* Bitcoin Cash:
    * `bitcoin-cash/blocks` - contains all Bitcoin Cash blocks, including the latest one
	* `bitcoin-cash/transactions` - contains all Bitcoin Cash transactions, excluding mempool transactions as well as transactions from the latest block
	* `bitcoin-cash/outputs` - contains all Bitcoin Cash outlets, excluding the outputs contained in mempool transactions, and also transactions from the latest block
	* `bitcoin-cash/mempool/blocks` - contains only the latest block 
	* `bitcoin-cash/mempool/transactions` - contains Bitcoin Cash mempool transactions as well as transactions from the latest block
	* `bitcoin-cash/mempool/outputs` - contains Bitcoin outputs included in mempool transactions as well as in transactions from the latest block
* Ethereum:
	* `ethereum/blocks` - contains all Ethereum blocks, except the last 6
	* `ethereum/uncles` - contains all Ethereum uncles, except those that belong to the last 6 blocks
	* `ethereum/transactions` - contains all Ethereum transactions, except transactions from the last 6 blocks
	* `ethereum/calls` - contains all transaction calls, except calls for the last 6 blocks
	* `ethereum/mempool/blocks` - contains the last 6 Ethereum blocks, some columns contain null
	* `ethereum/mempool/transactions` - contains all Ethereum transactions from the last 6 blocks as well as mempool transactions 

Notes: to speed up the process, our architecture contains separate tables (`mempool*`) for unconfirmed transactions, as well as for blocks that with a certain probability can be forked off from the main chain. For Bitcoin and Bitcoin Cash, `mempool*` contains the latest block transactions in addition to mempool transactions, and for Ethereum, that's the latest 6 blocks plus the mempool. Exception: for Bitcoin and Bitcoin Cash, the `blocks` table also contains information about the latest block (this may change in the future). For Ethereum, we do not "replay" transactions entirely (i.e. not looking for internal calls) for the last 6 blocks, so there is no `mempool/calls` table.

**You can use filters** as follows: `?q=field(value)[,field(value)...]`, where `field` is the column by which a filter is needed, and `value` is a value, special value, or a range of values. The possible columns are listed in the tables below. Possible expressions for values:
* `value` - e.g., ` bitcoin/blocks?q=id(0)` finds information about block 0
* `left..` - non-strict inequality - e.g., `bitcoin/blocks?q=id(1..)` finds information about block 1 and above
* `left...` - strict inequality - e.g., `bitcoin/blocks?q=id(1...)` finds information about block 2 and above
* `..right` - non-strict inequality - e.g., `bitcoin/blocks?q=id(..1)` finds information about blocks 0 and 1
* `...right` - strict inequality - e.g.,` bitcoin/blocks?q=id(...1)` finds information only about block 0
* `left..right` - non-strict inequality - e.g., `bitcoin/blocks?q=id(1..3)` finds information about blocks 1, 2 and 3
* `left...right` - strict inequality - e.g., `bitcoin/blocks?q=id(1...3)` finds information only about block 2
* `~like` - occurrence in a string (`LIKE` operator), e.g., `bitcoin/blocks?q=coinbase_data_bin(~hello)` finds all blocks which contain `hello` in `coinbase_data_bin` 
* `^like` - occurrence at the beginning of a string (`STARTS WITH` operator), e.g., `bitcoin/blocks?q=coinbase_data_hex(^00)` finds all blocks for which` coinbase_data_hex` begins with `00`

For `time*`-fields, the value can be specified in the following formats:
* `YYYY-MM-DD HH:ii:ss`
* `YYYY-MM-DD`
* `YYYY-MM`

Inequalities are also supported for such values, but the left and right values ​​must be in the same format, e.g.: `bitcoin/blocks?q=time(2009-01-03..2009-01-31)`.

If you need to list several filters, you need to sepatate them by commas in the `?q=` section, e.g., `bitcoin/blocks?q=id(500000..),coinbase_data_bin(~hello)`

We're currently testing support for `NOT` and `OR` operators (this may change in the future, including possible removal of these operators).

The operator `NOT` is comma-separated before the expression to be inverted, e.g., `bitcoin/blocks?q=not,id(1..)` returns the block `0`.

The `OR` operator is specified between the expressions and takes precedence (like it's when two expressions around `OR` are wrapped in parentheses), e.g., `bitcoin/blocks?q=id(1),or,id(2)` returns information about blocks 1 and 2.

Maximum guaranteed supported number of filters in one query: 5.

**Sorting** can be used as follows: `?s=field(direction)`, where `direction` can be either `asc` for sorting in ascending order, or `desc` for sorting in descending order, e.g. `bitcoin/blocks?s=id(asc)`

If you need to apply several sorts, you can list them by commas, similar to filters. The maximum guaranteed number of sorts is 2.

**Limit** is used like this: `?limit=N`, where N is a natural number from 1 to 100. The default is 10. In some cases `LIMIT` is ignored, and in such cases `context.limit` will be set to `NULL`.

**Offset** can be used as a paginator, e.g., `?offset=10` returns the next 10 results. `context.offset` takes the value of the set `OFFSET`. The maximum value is 10000. If you need just the last page, it's easier and quicker to change the direction of the sorting to the opposite. Important: when iterating through the results, it is extremely likely that the number of rows in the database will increase because new blocks were found. To avoid that, you may add an additional condition that limits the block id to the value obtained in `context.state` in the first query.

##### bitcoin[-cash]/[mempool/]blocks

Returns data about blocks

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| id | int | Block height | + | + |
| hash | string `[0-9a-f]{64}` | Block hash | + | + |
| date | string `YYYY-MM-DD` | Block date (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Block time (UTC) | + | + |
| median_time | string `YYYY-MM-DD HH:ii:ss` | Block median time (UTC) | | | + |
| size | int | Block size, in bytes (may be more than 1 MB for Bitcoin, up to 32 MB for Bitcoin Cash) | + | + |
| stripped_size(\*) | int | Block size in bytes without taking witness information into account (up to 1 MB for Bitcoin) | + | + |
| weight (\*) | int | Block weight in weight units (up to 4 MWU) | + | + |
| version | int | Version field | + | + |
| version_hex | string `[0-9a-f]*` | Version field in hex | | |
| version_bits | string `[01]{30}` | Version field in binary form | | |
| merkle_root | `[0-9a-f]{64}` | Merkle root hash | | |
| nonce | int | Nonce value | + | + |
| bits | int | Bits field | + | + |
| difficulty | float | Difficulty | + | + |
| chainwork | string `[0-9a-f]{64}` | Chainwork field | | |
| coinbase_data_hex | string `[0-9a-f]*` | Hex information contained in the input of the coinbase transaction | + | |
| transaction_count | int | Number of transactions in the block | + | + |
| witness_count (\*) | int | Number of transactions in the block containing witness information | + | + |
| input_count | int | Number of inputs in all block transactions | + | + |
| output_count | int | Number of outputs in all block transactions | + | + |
| input_total | int | Sum of inputs in satoshi | + | + |
| input_total_usd | float | Sum of outputs in USD (hereinafter for USD - at the moment `date`) | + | + |
| output_total | int | Sum of outputs in Satoshi | + | + |
| output_total_usd | float | Sum of outputs in USD | + | + |
| fee_total | int | Total fee in Satoshi | + | + |
| fee_total_usd | float | Total fee in USD | + | + |
| fee_per_kb | float | Fee per kilobyte (1000 bytes of data) in satoshi | + | + |
| fee_per_kb_usd | float | Fee for kilobyte of data in USD | + | + |
| fee_per_kwu (\*) | float | Fee for 1000 weight units of data in Satoshi | + | + |
| fee_per_kwu_usd (\*) | float | Fee for 1000 weight units of data in USD | + | + |
| cdd_total | float | Number of coindays destroyed by all transactions of the block | + | + |
| generation | int | Miner reward for the block in Satoshi | + | + |
| generation_usd | float | Miner reward for the block in USD | + | + |
| reward | int | Miner total reward (reward + total fee) in Satoshi | + | + |
| reward_usd | float | Miner total reward  (reward + total fee) in USD | + | + |
| guessed_miner | string `.*` | The supposed name of the miner who found the block (the heuristic is based on `coinbase_data_bin` and the addresses to which the reward goes) | + | + |

Additional synthetic columns (you can search over them and / or sort them, but they are not shown)

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| coinbase_data_bin | string `.*` | Text representation of coinbase data. Allows you to use the `LIKE` operator: `?q=coinbase_data_bin(~hello)` | + | |

Notes:
- for the columns `id` and` hash` the increased efficiency at unloading of one record is applied
- there is no possibility to search over the `date` column directly, you can search like `?q=time(YYYY-MM-DD)`
- the search over the column `coinbase_data_hex` is done by the operator `^`, you can also use `~` for `coinbase_data_bin` (however, the field `coinbase_data_bin` will not be shown anyway)
- (\*) - only for Bitcoin
- the default sorting - id DESC

##### bitcoin[-cash]/[mempool/]transactions

Returns transaction data

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| block_id | int | The height (id) of the block containing the transaction | + | + |
| id | int | Internal transaction id (not related to the blockchain, used for internal purposes) | + | + |
| hash | string `[0-9a-f]{64}` | Transaction hash | + | |
| date | string `YYYY-MM-DD` | The date of the block containing the transaction (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Time of the block containing the transaction (UTC) | + | + |
| size | int | Transaction size in bytes | + | + |
| weight (\*) | int | Weight of transaction in weight units | + | + |
| version | int | Transaction version field | + | + |
| lock_time | int | Lock time - can be either a block height, or a unix timestamp | + | + |
| is_coinbase | boolean | Is it a coinbase (generating new coins) transaction? (For such a transaction `input_count` is equal to` 1` and means a synthetic coinbase input) | | + | |
| has_witness (\*) | boolean | Is there a witness part in the transaction (using SegWit)? | | + | |
| input_count | int | Number of inputs | + | + |
| output_count | int | Number of outputs | + | + |
| input_total | int | Input value in satoshi | + | + |
| input_total_usd | float | Input value in USD | + | + |
| output_total | int | Output value in Satoshi | + | + |
| output_total_usd | float | Total output value in USD | + | + |
| fee | int | Fee in Satoshi | + | + |
| fee_usd | float | Fee in USD | + | + |
| fee_per_kb | float | Fee per kilobyte (1000 bytes) of data in Satoshi | + | + |
| fee_per_kb_usd | float | Fee for kilobyte of data in USD | + | + |
| fee_per_kwu (\*) | float | Fee for 1000 weight units of data in Satoshi | + | + |
| fee_per_kwu_usd (\*) | float | Fee for 1000 weight units of data in USD | + | + |
| cdd_total | float | The number of destroyed coindays | + | + |

Notes:
- for the columns `id` and` hash` the increased efficiency at unloading of one record is applied
- there is no possibility to search over `date` column, you can use `?q=time(YYYY-MM-DD instead
- (\*) - only for Bitcoin
- the default sort is id DESC

##### bitcoin[-cash]/[mempool/]outputs

Returns information about the outputs (that become inputs when they are spent, and then `spending*` information appears)

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| block_id | int | Id of a block containing the transaction cointaining the output | + | + |
| transaction_id | int | The internal transaction id containing the output | + | + |
| index | int | Output index in a transaction (from 0) | + | + |
| transaction_hash | string `[0-9a-f]{64}` | Transaction hash | | |
| date | string `YYYY-MM-DD` | Date of a block containing the output (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Time of a block containing the output (UTC) | + | + |
| value | int | Monetary value of output | + | + |
| value_usd | float | Monetary value of output in USD at the moment `date` | + | + |
| recipient | string `[0-9a-zA-Z\-]*` | Bitcoin address or synthetic address of an output recipient | + | + |
| type | string (enum) | Output type, one of the following: `pubkey/pubkeyhash/scripthash/multisig/nulldata/nonstandard/witness_v0_scripthash/witness_v0_keyhash` | + | + |
| script_hex | string `[0-9a-f]*` | Hex of the output script | + | |
| is_from_coinbase | boolean | Is it a coinbase transaction output? | | + | |
| is_spendable | null or boolean | Is it theoretically possible to spend this output? For `pubkey` and` multisig` outputs, the existence of the corresponding private key is tested, in that case `true` and `false` are the possible values, depending on the result of the check. For `nulldata` outputs, it is always `false`. For other types it is impossible to check trivially, in this case `null` is shown | + | |
| is_spent | boolean | Is this output spent? (Each field further contains `null` if it is not spent | + | |
| spending_block_id | null or int | Id of the block containing the spending transaction. `null` if the output is not yet spent. | | + | + |
| spending_transaction_id | null or int | Internal transaction id where the output is spent | + | + |
| spending_index | null or int | Input index in the spending transaction (from 0) | + | + |
| spending_transaction_hash | null or string `[0-9a-f]{64}` | Spending transaction hash | | |
| spending_date | null or string `YYYY-MM-DD` | Date of the block, in which the output is spent | | |
| spending_time | null or string `YYYY-MM-DD HH:ii:ss` | Time of the block in which the output is spent | + | + |
| spending_value_usd | null or float | Monetary value of output in USD at the time of `spending_date` | + | + |
| spending_sequence | null or int | The technical sequence value | + | + |
| spending_signature_hex | null or string `[0-9a-f]*` | Hex of the spending script (signature) | | |
| spending_witness (\*) | null or string (JSONB) | Witness information in the escaped JSONB format | | |
| lifespan | null or int | The number of seconds from the time of the output creation (`time`) to its spending (`spending_time`), `null` if the output is not spent | + | + |
| cdd | null or float | The number of coindays destroyed spending the output | + | + |

Additional synthetic columns (you can search over them and / or sort them, but they are not shown)

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| script_bin | string `.*` | Text representation of script_hex. Allows you to use the `LIKE` operator: `?q=script_bin(~hello)`| + | |

Notes:
- for columns `transaction_id` and `spending_transaction_id`, the increased efficiency at unloading records if one specific transaction is specified (and not the range), is applied
- there is no possibility to search over the `date` and `spending_date` columns, you can use `?q=time(YYYY-MM-DD)` and `?q=spending_time(YYYY-MM-DD)` instead
- the search over `script_hex` column can be done by the operator `^`, you can also use `~` for `script_bin` (however, the field `script_bin` will still not be shown)
- (\*) - only for Bitcoin
- the default sort is - transaction_id DESC

##### ethereum/[mempool/]blocks

Returns block data

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| id | int | Block id | + | + |
| hash | string `0x[0-9a-f]{64}` | Block hash (with 0x) | + | |
| date | string `YYYY-MM-DD` | Block date (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Block time (UTC) | + | + |
| size | int | Block size in bytes | + | + |
| miner | string `0x[0-9a-f]{40}` | Address of a rewarded miner (from 0x) | + | |
| extra_data_hex | string `[0-9a-f]*` | Additional data included by the miner | + | |
| difficulty | int | Difficulty | + | + |
| gas_used | int | Gas amount used by block transactions | + | + |
| gas_limit | int | Gas limit for a block set up by the miner | + | + |
| logs_bloom | string `[0-9a-f]*` | Logs Bloom | | |
| mix_hash | string `[0-9a-f] {64}` | Hash Mix hash | | |
| nonce | string `[0-9a-f]*` | Nonce value | | |
| receipts_root | string `[0-9a-f] {64}` | Receipts Root hash | | |
| sha3_uncles | string `[0-9a-f] {64}` | SHA3 Uncles hash | | |
| state_root | string `[0-9a-f] {64}` | State Root hash | | |
| total_difficulty | numeric string | Total difficulty | | |
| transactions_root | string `[0-9a-f] {64}` | Transactions Root hash | | |
| uncle_count | int | Number of uncles | + | + |
| transaction_count | int | Number of transactions in the block | + | + |
| synthetic_transaction_count | int | The number of synthetic transactions (they do not exist as separate transactions, but they change the state, e.g., genesis block transactions, miner rewards, DAO-fork transactions) | + | + |
| call_count (\*) | int | Total number of calls spawned by transactions | + | + |
| synthetic_call_count | int | Number of synthetic calls (same as synthetic transactions) | + | + |
| value_total | numeric string | Monetary value of all block transactions in wei, hereinafter `numeric string` - numeric value passed as a string, because wei-values do not fit into uint64 | + | + |
| value_total_usd | float | Monetary value of all block transactions in USD | + | + |
| internal_value_total (\*) | numeric string | Monetary value of all internal calls in the block (see note below) in wei | + | + |
| internal_value_total_usd (\*) | float | Monetary value of all internal calls in a block in USD | + | + |
| generation | numeric string | The reward of a miner for the block generation in wei (3 or 5 ether + reward for uncle inclusion) | + | + |
| generation_usd | float | The reward of a miner for the block generation in USD | + | + |
| uncle_generation (\*) | numeric string | Total reward of uncle miners in wei | + | + |
| uncle_generation_usd (\*) | float | Total reward of uncle miners in USD | + | + |
| fee_total (\*) | numeric string | Total fee in wei | + | + |
| fee_total_usd (\*) | float | Total fee in USD | + | + |
| reward (\*) | numeric string | Total reward of the miner in the wei (reward for finding the block + fees) | + | + |
| reward_usd (\*) | float | Total reward of the miner in USD | + | + |

Additional synthetic columns (you can search over them and / or sort them, but they are not shown)

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| extra_data_bin | string `.*` | Text representation of extra data. Allows you to use the `LIKE` operator: `?q=extra_data_bin(~hello)`| + | |

Notes:
- (\*) - always `null` for `mempool / blocks`
- for `id` and` hash` columns the increased efficiency at unloading of one record is applied
- there is no possibility to search the `date` column, but you can use `?q=time(YYYY-MM-DD)` instead
- search by fields that contain values in wei (`value_total`,` internal_value_total`, `generation`,` uncle_generation`, `fee_total`,` reward`) can be with some inaccuracies
- the search over `extra_data_hex` column can be done by the operator `^`, you can also use `~`for `extra_data_bin` (however, the field `extra_data_bin` will still not be shown)
- the difference between `value_total` and `internal_value_total`: e.g., a transaction itself sends 0 eth, but this transaction is a call of a contract that sends someone, let's say, 10 eth. Then `value` will be 0 eth, and `internal_value` - 10 eth
- the default sort is id DESC

##### ethereum/uncles

Returns information about uncles

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| parent_block_id | int | Parent block id | + | + |
| index | int | Uncle index in the block | + | + |
| hash | string `0x[0-9a-f]{64}` | Uncle hash (with 0x) | + | |
| date | string `YYYY-MM-DD` | Date of generation (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Time of generation (UTC) | + | + |
| size | int | Uncle size in bytes | + | + |
| miner | string `0x[0-9a-f]{40}` | Address of the rewarded miner (with 0x) | + | |
| extra_data_hex | string `[0-9a-f]*` | Additional data included by the miner | + | |
| difficulty | int | Difficulty | + | + |
| gas_used | int | Amount of gas used by transactions | + | + |
| gas_limit | int | Gas limit for the block set up by the miner | + | + |
| logs_bloom | string `[0-9a-f]*` | Logs Bloom | | |
| mix_hash | string `[0-9a-f]{64}` | Hash Mix hash | | |
| nonce | string `[0-9a-f]*` | Nonce value | | |
| receipts_root | string `[0-9a-f]{64}` | Receipts Root hash | | |
| sha3_uncles | string `[0-9a-f]{64}` | Uncles hash | | |
| state_root | string `[0-9a-f]{64}` | State Root hash | | |
| total_difficulty | numeric string | Total difficulty | | |
| transactions_root | string `[0-9a-f]{64}` | Transactions Root hash | | |
| generation | numeric string | The reward of the miner who generated the uncle, in wei | + | + |
| generation_usd | float | The award of the miner who generated uncle, in USD | + | + |

Additional synthetic columns (you can search over them and / or sort them, but they are not shown)

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| extra_data_bin | string `.*` | Text representation of extra data. Allows you to use the `LIKE` operator:`?Q=extra_data_bin(~hello)` | + | |

Notes:

- for the columns `parent_block_id` and `hash` increased efficiency when uploading one or more records is applied
- there is no possibility to search the `date` column directly, but you can use `?q=time(YYYY-MM-DD)` instead
- search by fields that contain values in wei (`generation`) can be with some inaccuracies
- the search over `extra_data_hex` column can be done by the operator `^`, you can also use `~` for `extra_data_bin` (however, the field `extra_data_bin` will still not be shown)
- sort by default - parent_block_id DESC

##### ethereum/[mempool/]transactions

Returns transaction information

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| block_id | int | Id of the block containing the transaction | + | + |
| id | int | Transaction id (not related to the blockchain, used for internal purposes) | + | + |
| index (\*) (\*\*) | int | The transaction index number in the block | + | + |
| hash (\*\*) | string `0x[0-9a-f]{64}` | Transaction hash (with 0x) | + | |
| date | string `YYYY-MM-DD` | Date of the block containing the transaction (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Time of the block containing the transaction (UTC) | + | + |
| size | int | Transaction size in bytes | + | + |
| failed (\*) | bool | Failed transaction or not? + | | |
| type (\*) | string (enum) | Transaction type with one of the following values: `call/create/call_tree/create_tree/synthetic_coinbase`. Description in the note. | | + | |
| sender (\*\*) | string `0x[0-9a-f]{40}` | Address of the transaction sender (with 0x) | + | |
| recipient | string `0x[0-9a-f]{40}` | Address of the transaction recipient (with 0x) | + | |
| call_count (\*) | int | Number of calls in the transaction  | + | + |
| value | numeric string | Monetary value of transaction in wei, here and below `numeric string` - is a numeric value passed as a string, because wei-values do not fit into uint64 | + | + |
| value_usd | float | Value of transaction in USD | + | + |
| internal_value (\*) | numeric string | Value of all inner calls in the transaction in wei | + | + |
| internal_value_usd (\*) | float | Value of all internal calls in the transaction in USD | + | + |
| fee (\*) (\*\*) | numeric string | Fee in wei | + | + |
| fee_usd (\*) (\*\*) | float | Fee in USD | + | + |
| gas_used (\*) (\*\*) | int | Amount of gas used by a transaction | + | + |
| gas_limit (\*\*) | int | Gas limit for transaction set by the sender | + | + |
| gas_price (\*\*) | int | Price for gas set by the sender | + | + |
| input_hex (\*\*) | string `[0-9a-f]*` | Transaction input data | + | |
| nonce (\*\*) | string `[0-9a-f]*` | Nonce value | | |
| v (\*\*) | string `[0-9a-f]*` | V value | | |
| r (\*\*) | string `[0-9a-f]*` | R value | | |
| s (\*\*) | string `[0-9a-f]*` | S value | | |

Additional synthetic columns (you can search over them and / or sort them, but they are not shown)

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| input_bin | string `.*` | Text representation of input data. Allows you to use the `LIKE` operator: `?q=input_bin(~hello)` | + | |

Notes:

- (\*) - is always `null` for `mempool/transactions`
- (\*\*) - is always equal to `null` if `type` = `synthetic_coinbase`
- for the columns `id` and` hash` the increased efficiency at unloading of one record is applied
- there is no possibility to search over `date` column, you can search like `?q=time(YYYY-MM-DD)` instead
- search by fields that contain values in wei (`value`,` internal_value`) can be with some inaccuracies
- the search over `input_hex` column can be done by the operator `^`, you can also use `~` for `input_bin` (however, the field `input_bin` will still not be included in output)
- the difference between `value_total` and `internal_value_total`: e.g., a transaction itself sends 0 eth, but this transaction is a call of a contract that sends someone, let's say, 10 eth. Then `value` will be 0 eth, and `internal_value` - 10 eth
- the default sort is - id DESC
- possible types (`type`) of transactions:
    * call - the transaction transfers the value, but there are no more calls (a simple ether sending, not in favor of a contract, or the call to a contract that does nothing)
    * create - create a new contract
    * call_tree - the transaction calls a contract that makes some other calls
    * create_tree - create a new contract that create contracts or starts making calls
    * synthetic_coinbase - a synthetic transaction for awarding a reward to the miner (block or uncle)

##### ethereum/calls

Returns information about calls

| Column | Type | Description | Q? | S? | 
|--------|------|-------------|----|----|
| block_id | int | Block id containing a call | + | + |
| transaction_id | int | Transaction id containing the call | + | + |
| transaction_hash (\*\*) | string `0x[0-9a-f]{64}` | Transaction hash (with 0x) containing the call | | |
| index | string | Call index within the transaction (tree-like, e.g., "0.8.1") | + | + |
| depth | int | Call depth within the call tree (starting at 0) | + | + |
| date | string `YYYY-MM-DD` | Date of the block that contains the call (UTC) | | |
| time | string `YYYY-MM-DD HH:ii:ss` | Time of the block that contains the call (UTC) | + | + |
| failed | bool | Failed call or not | + | |
| fail_reason | string `.*` or null | If failed, then the failure description, if not, then `null` | + | |
| type | string (enum) | The call type, one of the following values: `call/delegatecall/staticcall/callcode/selfdesctruct/create/synthetic_coinbase` | + | |
| sender (\*\*) | string `0x[0-9a-f]{40}` | Sender's address (with 0x) | + | |
| recipient | string `0x[0-9a-f]{40}` | Recipient's address (with 0x) | + | |
| child_call_count | int | Number of child calls | + | + |
| value | numeric string | Call value in wei, hereinafter `numeric string` - is a numeric string passed as a string, because wei-values do not fit into uint64 | + | + |
| value_usd | float | Call value in USD | + | + |
| transferred | bool | Has ether been transferred? (`false` if `failed`, or if the type of transaction does not change the state, e.g., `staticcall` | + | |
| input_hex (\*\*) | string `[0-9a-f]*` | Input call data | | |
| output_hex (\*\*) | string `[0-9a-f]*` | Output call data | | |

Notes:

- (\*\*) - is always `null` if` type` = `synthetic_coinbase`
- for the column `transaction_id` increased efficiency when uploading one record is applied
- there is no possibility to search over `date` column, use searching `?q=time(YYYY-MM-DD)` instead
- search by fields that contain values in wei (`value`,` internal_value`) can be with some inaccuracies
- the default sort is transaction_id DESC
- sorting by `index` is alphabetical (ie "0.2" goes after "0.11"), in some cases a switch to natural sorting is used (for example, when there is a filter for `transaction_id`)

##### Notes

- for unconfirmed transactions (and outputs in the case of bitcoin[-cash]), the following rules are applied:
    - their `block_id` is equal to `-1`
    - `date` and` time` indicate the time when the transaction was received by our node
- when using `offset`, it is reasonable to add to the filters the maximum block number (`?q=block_id(..N)`), since it is very likely that during the iteration new rows will be added to the table. For convenience, you can take the value of `context.state` from the first result of any query containing the number of the latest block at the query time and use this result later on.

#### Dashboard calls

The API supports a number of calls that produce some aggregated data, or data in a more convenient form for certain entities.

##### (bitcoin[-cash]|ethereum)/dashboards/block/{A} and (bitcoin[-cash]|ethereum)/dashboards/blocks/{A[,B,...]}

As the input data, it takes the height or hash of the block(s). `data` returns an array with block heights or block hashes used as keys, and arrays of elements as values:
* `block` - information about the block in infinitable-format `(bitcoin[-cash]|ethereum)/blocks`
* `transactions` - the array of all hashes of transactions included in the block
* only for Ethereum - `synthetic_transactions` - array of internal ids of synthetic transactions (they do not have a hash) (`null` instead of the array until the block receives 6 confirmations)
* only for Ethereum - `uncles` - the array of hashes of the block's uncles (`null` instead of the array until the block receives 6 confirmations, in case there are no uncles and more than 6 confirmations - an empty array (`[]`))

`context.results` contains the number of found blocks.

##### ethereum/dashboards/uncle/{A} and ethereum/dashboards/uncles/{A[,B,...]}

As the input data, it takes an uncle hash(es). `data` returns an array with uncle hashes used as keys, and arrays of elements as values:
* `uncle` - information about the block in infinitable-format `ethereum/uncles`

`context.results` contains the number of found uncles.

##### (bitcoin[-cash]|ethereum)/dashboards/transaction/{A} and (bitcoin[-cash]|ethereum)/dashboards/transactions/{A[,B,...]}

At the input data, it takes an internal blockchair-id or a hash of a transaction (transactions). `data` returns an array with identifiers or hashes of transactions used as keys, and arrays of elements as keys:
* `transaction` - transaction information in infinitable-format `bitcoin[-cash]/transactions`
* (only bitcoin[-cash]) `inputs` - array of all transaction inputs, sorted by `spending_index` in infinitable-format `bitcoin[-cash]/outputs`
* (only bitcoin[-cash]) `outputs` - array of all transaction outputs, sorted by `index` in infinitable-format `bitcoin[-cash]/outputs`
* (only ethereum) `calls` - the array of all calls made during the execution of the transaction (always `null` for mempool transactions and the last 6 blocks)

`context.results` contains the number of found transactions.

##### (bitcoin[-cash]|ethereum)/dashboards/transaction/{hash}/priority

For mempool transactions shows priority (`position`) (for Bitcoin - by `fee_per_kwu`, for Bitcoin Cash - by `fee_per_kb`, for Ethereum - by `gas_price`) over other transactions (`out_of` mempool transactions). It has the same structure as the `(bitcoin[-cash]|ethereum)/dashboards/transaction/{A}` call

##### bitcoin[-cash]/dashboards/address/{A}

Uses address as the input data. `data` returns an array with one element (if the address is found), in that case the address is the key, and the value is an array consisting of the following elements:
* `address`
    * `address.type` - address type (type of output is the same as `bitcoin[-cash].outputs.type`)
    * `address.script_hex` - address script
    * `address.balance` - address balance in satoshi (hereinafter - including unconfirmed outputs) - int
    * `address.balance_usd` - address balance in USD - float
    * `address.received` - total received in satoshi
    * `address.received_usd` - total received in USD
    * `address.spent` - total spent in satoshi
    * `address.spent_usd` - total spent in USD
    * `address.output_count` - the number of outputs this address received
    * `address.unspent_output_count` - number of unspent outputs for this address (i.e. the number of inputs for an address can be calculated as `output_count`-`unspent_output_count`)
    * `address.first_seen_receiving` - timestamp when the first time this address received bitcoins
    * `address.last_seen_receiving` - timestamp when the last time this address received bitcoins
    * `address.first_seen_spending` - timestamp when the first time this address sent bitcoins
    * `address.last_seen_spending` - timestamp when the last time this address sent bitcoins
    * `address.transaction_count` - number of unique transactions this address participating in
* `transactions` - an array of the last 100 hashes the address is participating in

`context.results` contains the number of found addresses (0 or 1, until the `addresses` call is implemented).

To iterate `transactions`, `?offset=N` is supported.

##### ethereum/dashboards/address/{A}

Uses address as the input data. `data` returns an array with one element (if the address is found), in that case the address is the key, and the value is an array consisting of the following elements:
* `address`
    * `address.type` - address type (`account` - for an address, `contract` - for a contract)
    * `address.contract_code_hex` -  hex code of the contract at the momemt of creation (for a contract), or null (for an address)
    * `address.contract_destroyed` - for a contract - if the contact was destroyed (SELFDESCTRUCT) then true, if not - false, or null (for an address)
    * `address.balance` - exact address balance in wei (here and below for values in wei - numeric string)
    * `address.balance_usd` - address balance in USD - float
    * `address.received_approximate` - total received in wei (approximately) (\*)
    * `address.received_usd` - total received in USD (approximately) (\*)
    * `address.spent_approximate` - total spent in wei (approximately) (\*)
    * `address.spent_usd` - total spent in USD (approximately) (\*)
    * `address.fees_approximate` - total spent in transaction fees in wei (approximately) (\*)
    * `address.fees_usd` - total spent in transaction fees in USD (approximately) (\*)
    * `address.receiving_call_count` - number of calls in favor of this address, where  value transfer has occured (\*\*)
    * `address.spending_call_count` -  number of calls that was made by this address, where  value transfer has occured (\*\*)
    * `address.call_count` - total number of calls this address participating in (may be greater than` receiving_call_count` + `spending_call_count`, because it also takes into account failed calls)
    * `address.transaction_count` - number of transactions this address participating in
    * `address.first_seen_receiving` - timestamp when this address received a successful incoming call for the first time 
    * `address.last_seen_receiving` - timestamp when this address received a successful incoming call for the last time
    * `address.first_seen_spending` - timestamp when this address sent a successful call for the first time
    * `address.last_seen_spending` - timestamp when this address sent a successful call for the last time
* `calls` - an array of the last 100 calls with the address, each element of an array containing the following columns of `ethereum/calls`: `block_id`, `transaction_hash`,` index`, `time`,` sender`, `recipient`, `value`,` value_usd`, `transferred`

`context.results` contains the number of found addresses  (0 or 1, until the `addresses` call is implemented).

To iterate `calls`, `?offset=N` is supported.

Notes:
- (\*) - in these columns, the value in wei can be rounded. For a million of calls, the error can be more than 1 ether.
- (\*\*) - counted only those calls that fit the following condition: ethereum/calls.transferred = true (see the `ethereum/calls` documentation), i.e. those calls as well as failed calls that do not change state (staticcall, etc.) are not considered

##### (bitcoin[-cash]|ethereum)/stats

Returns an array with blockchain statistics:
* `blocks` - total number of blocks
* (only ethereum) `uncles` - total number of uncles
* `transactions` - total number of transactions
* (only ethereum) `calls` - total number of internal calls
* `blocks_24h` - blocks for the last 24 hours
* `circulation` for bitcoin[-cash], `circulation_approximate` for ethereum - number of coins in circulation (in Satoshi, or in wei for Ethereum - an approximate value)
* `transactions_24h` - transactions for the last 24 hours
* `difficulty` - current difficulty
* `volume_24h` for bitcoin[-cash], `volume_24h_approximate` for ethereum - monetary volume of transactions for the last 24 hours (for ethereum - an approximate value)
* `mempool_transactions` - number of transactions in the mempool
* (only ethereum) `mempool_median_gas_price` - median gas price in the mempool
* (only bitcoin[-cash]) `mempool_size` - the mempool size in bytes
* `mempool_tps` - number of transactions per second added to the mempool
* (only ethereum) `mempool_total_value_approximate` - mempool monetary value
* (only bitcoin[-cash]) `mempool_total_fee_usd` - total mempool fee, in USD
* `best_block_height` - the latest block height
* `best_block_hash` - the latest block hash
* `best_block_time` - the latest block time
* (only ethereum) `uncles_24h` - number of uncles for the last 24 hours
* (only bitcoin[-cash]) `nodes` - number of complete nodes 
* `hashrate_24h` - hashrate (hashes per second) in average for the last 24 hours
* `market_price_usd` - average market price of 1 coin in USD (market data source: CoinGecko)
* `market_price_btc` - average market price of 1 coin in BTC (for Bitcoin always returns 1)
* `market_price_usd_change_24h_percentage` - market price change in percent for 24 hours
* `market_cap_usd` - market capitalization (coins in circulation * price per coin in USD)
* `market_dominance_percentage` - dominance index (how much % of the total cryptocurrency market is the market capitalization of the coin)

##### stats

Returns data on three calls:
* `bitcoin/stats`
* `bitcoin-cash/stats`
* `ethereum/stats`

#### API request examples

Suppose we would like to receive all the latest transactions from the Ethereum blockchain which amount to more than $1M USD. The following request should be done for this:
* `https://api.blockchair.com/ethereum/transactions?q=internal_value_usd(10000000..)&s=transaction_id(desc)`

In this request we refer to the blockhain (`ethereum`), a table (`transactions`), set up the amount condition (`q=internal_value_usd(10000000..)`), and sort in descending order (`&s=transaction_id(desc)`).

Suppose, a script with this request to the API for some reason did not work for a while, or a huge amount of transactions worth more than $1 million appeared. With the standard limit of 10 results, the script skipped some transactions. Then firstly we should do the following:
* `https://api.blockchair.com/ethereum/transactions?q=internal_value_usd(10000000..)&s=transaction_id(desc)`

From its result we save `context.state`, put it in a variable `_S_`, and further to obtain the following results we apply `offset`:
* `https://api.blockchair.com/ethereum/transactions?q=internal_value_usd(10000000..),block_id(.._S_)&s=transaction_id(desc)&offset=10`

Increase offset value until getting a data set with the transaction that we already knew about.

#### Support

* E-mail: [info@blockchair.com](mailto:info@blockchair.com)
* Telegram chat: [@Blockchair](https://telegram.me/Blockchair)
* Twitter: [@Blockchair](https://twitter.com/Blockchair)
