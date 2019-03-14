---
id: api_websocket
title: BUMO Websocket
sidebar_label: Websocket
---



## Overview

### What is Protocol Buffer3

BUMO Blockchain serializes data with `protocol buffer 3`, which is a general serialization protocol launched by Google. Click this [proto3](https://developers.google.com/protocol-buffers/docs/proto3) to get more information. All the data format we use are under the dir: `src\proto`. Other data with reference to transaction, block ,account are in the `chain.proto` file.



### Protocol Buffer3

This section provides examples of proto scripts, as well as proto source code generated by ``cpp``, ``java``, ``javascript``, ``pyton``, ``object-c``, and ``php``. For more information, please refer to the [proto](https://github.com/bumoproject/bumo/tree/develop/src/proto).

Description of the directory structure in the above link is shown below:

1. cpp: [C++ source code](https://github.com/bumoproject/bumo/tree/master/src/proto/cpp)
2. io: [Java test program](https://github.com/bumoproject/bumo/tree/master/src/proto/io)
3. go: [Go test program](https://github.com/bumoproject/bumo/tree/master/src/proto/go)
4. js: [Javascript test program](https://github.com/bumoproject/bumo/tree/master/src/proto/js)
5. Python: [Python test program](https://github.com/bumoproject/bumo/tree/master/src/proto/python)
6. ios: [Object-c test program](https://github.com/bumoproject/bumo/tree/master/src/proto/ios)
7. php: [PHP test program](https://github.com/bumoproject/bumo/tree/master/src/proto/php)



### websocket

BUMO Blockchain offers websocket API. You can find the`"wsserver"` objecct in the downloaded dir:  `/config/bumo.json` , which assign the service port.

```protobuf
"wsserver":
{
    "listen_address": "0.0.0.0:36003"
}
```



### Port Configuration

| network type | WebSocket |
| ------------ | --------- |
| mainnet      | 16003     |
| testnet      | 26003     |
| beta version | 36003     |



### Perform Transaction

- Fill in the transaction → `Transaction`(Details for :[Transaction](#transaction))
- Serializing the transaction (protocol buffer 3) to bytes stream → `transaction_blob`，`Transaction` Object has the serialization method, which is called to get the `transaction_blob`。
- Signing the `transaction_blob` with private key `skey`, and get the `sign_data`. The public key of `skey` is `pkey`。(Details for [Keypair Guide](keypair_guide))
- Submitting transaction, And you can get the message of whether the execution is successful or not through the response message.(Details for [Submit Transaction](#submit-transaction))



## Transactions

- In protobuf format

```protobuf
message Transaction {
	enum Limit{
		UNKNOWN = 0;
		OPERATIONS = 1000;
	};
	string source_address = 1;
	int64 nonce = 2;
	int64  fee_limit = 3;
	int64  gas_price =4;
	int64 ceil_ledger_seq = 5;
	bytes metadata = 6;
	repeated Operation operations = 7;
	int64 chain_id = 8;
}
```

- Keywords in protobuf

| Keyword         | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| source_address  | string | The source account of the transaction, which is the account of the transaction initiator. When the transaction is successful, the nonce field of the source account will be automatically incremented by 1. The nonce in the account number is the number of transactions executed by this account |
| nonce           | int64  | Its value must be equal to the current nonce+1 of the source account of the transaction, which is designed to prevent replay attacks. If you want to know how to query the nonce of an account, you can refer to the [getAccount](api_http#getAccount) interface in HTTP. If the account queried does not display the nonce value, the current nonce of the account is 0. |
| fee_limit       | int64  | The maximum fee that can be accepted for this transaction. The transaction will first charge a fee based on this fee. If the transaction is executed successfully, the actual cost will be charged, otherwise the fee for this field will be charged. The unit is MO, 1 BU = 10^8 MO |
| gas_price       | int64  | It is used to calculate the handling fee for each operation and also involved in the calculation of the transaction byte fee. The unit is MO, 1 BU = 10^8 MO |
| ceil_ledger_seq | int64  | Optional, the block height restriction for this transaction, which is also an advanced feature |
| operations      | array  | The operation list. The payload of this transaction, which is what the transaction wants to do. See [Operations](#operations) for more details |
| metadata        | string | Optional, a user-defined field that can be left blank or filled in a note |



## Operations

The corresponding `operations` in the protobuf structure of the transaction can contain one or more operations.

- In protobuf format

```protobuf
message Operation {
	enum Type {
		UNKNOWN = 0;
		CREATE_ACCOUNT 			= 1;
		ISSUE_ASSET 			= 2;
		PAY_ASSET               = 3;
		SET_METADATA			= 4;
		SET_SIGNER_WEIGHT		= 5;
		SET_THRESHOLD			= 6;
		PAY_COIN                = 7;
		LOG						= 8;
		SET_PRIVILEGE			= 9;
	};
	Type type = 1;
	string source_address = 2;
	bytes metadata	= 3;

	OperationCreateAccount		create_account 	   = 4;
	OperationIssueAsset			issue_asset 	   = 5;
	OperationPayAsset			pay_asset 		   = 6;
	OperationSetMetadata		set_metadata	   = 7;
	OperationSetSignerWeight	set_signer_weight  = 8;
	OperationSetThreshold		set_threshold 	   = 9;
	OperationPayCoin			pay_coin           = 10;
	OperationLog				log				   = 11;
	OperationSetPrivilege		set_privilege	   = 12;
}
```

- Keyword in protobuf

| Keyword        | Type                   | Description                                                  |
| -------------- | ---------------------- | ------------------------------------------------------------ |
| type           | int                    | Operation code, different operation codes perform different operations, see [Operation Codes](#operation-codes) for details |
| source_address | string                 | Optional, the source account of the operation, that is, the operator of the operation. When not filled in, the default is the same as the source account of the transaction |
| metadata       | string                 | Optional, a user-defined field that can be left blank or filled in a note |
| create_account | OperationCreateAccount | The [Creating Accounts](#creating-accounts) operation        |
| issue_asset    | OperationIssueAsset    | The [Issuing Assets](#issuing-assets) operation              |
| pay_asset      | OperationPayAsset      | The  [Transferring Assets](#transferring-assets) operation   |
| set_metadata   | OperationSetMetadata   | The [Setting Metadata](#setting-metadata) operation          |
| pay_coin       | OperationPayCoin       | The [Transferring BU Assets](#transferring-bu-assets) operation |
| log            | OperationLog           | The [Recording Logs](#recording-logs) operation              |
| set_privilege  | OperationSetPrivilege  | The [Setting Privileges](#setting-privileges) operation      |



### Operation Codes

| Operation Code | Description                                       |
| :------------- | ------------------------------------------------- |
| 1              | [Creating Accounts](#creating-accounts)           |
| 2              | [Issuing Assets](#issuing-assets)                 |
| 3              | [Transferring Assets](#transferring-assets)       |
| 4              | [Setting Metadata](#setting-metadata)             |
| 7              | [Transferring BU Assets](#transferring-bu-assets) |
| 8              | [Recording Logs](#recording-logs)                 |
| 9              | [Setting Privileges](#setting-privileges)         |

### Creating Accounts

The source account creates a new account on the blockchain. Creating Accounts are divided into [Creating Normal Accounts](#creating-normal-accounts) and [Creating Contract Accounts](#creating-contract-accounts).

Protobuf format as follow: 

```protobuf
// Key-Value pair
message KeyPair{
	string key = 1;
	string value = 2;
	int64 version = 3;
}

// Privilege
message Signer {
	enum Limit{
		SIGNER_NONE = 0;
		SIGNER = 100;
	};
	string address = 1;
	int64 weight = 2;
}
message OperationTypeThreshold{
	Operation.Type type = 1;
	int64 threshold = 2;
}
message AccountThreshold{
	int64 tx_threshold = 1; //required, [-1,MAX(INT64)] -1: indicates no setting
	repeated OperationTypeThreshold type_thresholds = 2;
}
message AccountPrivilege {
	int64 master_weight = 1;
	repeated Signer signers = 2;
	AccountThreshold thresholds = 3;
}

// Contract
message Contract{
    enum ContractType{
		JAVASCRIPT = 0;
	}
	ContractType type = 1;
	string payload = 2;
}

//　Create Account Operation
message OperationCreateAccount{
	string dest_address = 1;
	Contract contract = 2;
	AccountPrivilege priv = 3;
	repeated KeyPair metadatas = 4;	
	int64	init_balance = 5;
	string  init_input = 6;
}
```



#### Creating Normal Accounts

> **Note**: Both `master_weight` and `tx_threshold` must be 1 in the current operation. And only the following keywords are allowed to be initialized.

- Keyword in protobuf

| Keyword       | Type   | Description                                                  |
| ------------- | ------ | ------------------------------------------------------------ |
| dest_address  | string | The address of the target account. When creating a normal account, it cannot be empty |
| init_balance  | int64  | The initial BU value of the target account, in MO, 1 BU = 10^8 MO |
| master_weight | int64  | The master weight of the target account, which ranges [0, MAX(UINT32)] |
| tx_threshold  | int64  | The threshold for initiating a transaction below which the transaction cannot be initiated, which ranges ​​[0, MAX(INT64)] |

- Query

  The account information is queried through the [getAccount](api_http#getAccount) interface in HTTP.



#### Creating Contract Accounts

> **Note**: In the current operation, `master_weight` must be 0 and `tx_threshold` must be 1. And only the following keywords are allowed to be initialized

- Keyword in protobuf

| Keyword       | Type   | Description                                                  |
| ------------- | ------ | ------------------------------------------------------------ |
| payload       | string | The contract code                                            |
| init_balance  | int64  | The initial BU value of the target account, in MO, 1 BU = 10^8 MO |
| init_input    | string | Optional, the input parameter of the init function in the contract code |
| master_weight | int64  | The master weight of the target account                      |
| tx_threshold  | int64  | The threshold for initiating a transaction below which it is not possible to initiate a transaction. |

- Query
  - The account information is queried through the [getAccount](api_http#getAccount) interface in HTTP.
  - Query with the [getTransactionHistory](api_http#gettransactionhistory) interface in HTTP, and the result is as follows:

```protobuf
[
    {
        "contract_address": "buQm5RazrT9QYjbTPDwMkbVqjkVqa7WinbjM", //The contract account
        "operation_index": 0                                        //The operation index value in the transaction array, 0 means the first transaction
    }
]
```



### Issuing Assets

- Function

  The source account of this operation issues a digital asset, and this asset appears in the asset balance of the source account after successful execution.

- In protobuf format

```protobuf
message OperationIssueAsset{
	string code = 1;
	int64 amount = 2;
}
```

- Keyword in protobuf

| Keyword | Type   | Description                                                  |
| ------- | ------ | ------------------------------------------------------------ |
| code    | string | The code of the asset to be issued, which ranges [1, 64]     |
| amount  | int64  | The amount of the asset to be issued, which ranges ​​(0, MAX(int64)) |



### Transferring Assets

> **Note**: If the target account is a contract account, the current operation triggers the contract execution of the target account. 

- Function

  The source account of this operation transfers an asset to the target account.

- In protobuf format

```protobuf
message AssetKey{
	 string issuer = 1;
	 string code = 2;
	 int32 type = 3;
}
message Asset{
	 AssetKey	key = 1;
	 int64		amount = 2;
}

//　Pay asset operation
message OperationPayAsset{
	string dest_address = 1;
	Asset asset = 2;
	string input = 3;
}
```

- Keyword in protobuf

| Keyword      | Type   | Description                                                  |
| ------------ | ------ | ------------------------------------------------------------ |
| dest_address | string | The address of the target account                            |
| issuer       | string | The address of the issuer                                    |
| code         | string | The asset code which ranges [1, 64]                          |
| amount       | int64  | The amount of the asset which ranges (0,MAX(int64))          |
| input        | string | Optionally, if the target account is a contract account, the input will be passed to the argument of the `main` function of the contract code. This setting is invalid if the target account is a normal account |



### Setting Metadata

- Function

 The source account of this operation modifies or adds metadata to the metadata table.

- In protobuf format

```protobuf
message OperationSetMetadata{
	string	key = 1;  
	string  value = 2;
	int64 	version = 3;
	bool    delete_flag = 4;
}
```

- Keyword in protobuf

| Keyword | Type   | Description                                                  |
| ------- | ------ | ------------------------------------------------------------ |
| key     | string | The keyword of metadata, which ranges (0, 1024)              |
| value   | string | The content of metadata, which ranges [0, 256K].             |
| version | int64  | Optional, metadata version number. The default value is *0*. 0: when the value is zero, it means no limit version; >0: when the value is greater than zero, it means the current value version must be this value; <0: when the value is less than zero, it means the value is illegal |



### Setting Privileges

- Function

Set the weights that the signer has and set the thresholds required for each operation. For details, see [Assignment of Control Rights](api_http#assignment-of-control-rights) in HTTP.

- In protobuf format

```protobuf
message Signer {
	enum Limit{
		SIGNER_NONE = 0;
		SIGNER = 100;
	};
	string address = 1;
	int64 weight = 2;
}
message OperationTypeThreshold{
	Operation.Type type = 1;
	int64 threshold = 2;
}

//　Set privilege object
message OperationSetPrivilege{
	string master_weight = 1;
	repeated Signer signers = 2;
	string tx_threshold = 3;
	repeated OperationTypeThreshold type_thresholds = 4;
}
```



- Keywords in protobuf

| Keyword         | Type   | Description                                                  |
| --------------- | ------ | ------------------------------------------------------------ |
| master_weight   | string | Optional, by default "", it indicates the master weight of the account. "" : do not set this value; "0": set the master weight to 0; ("0", "MAX(UINT32)"]: set the weight value to this value; Other: illegal |
| signers         | array  | Optional, a list of signers that need to operate. By default is an empty object. Empty objects are not set |
| address         | string | The signer's address that needs to operate, which should be in accordance with the address verification rules |
| weight          | int64  | Optional, by default is 0. 0: delete the signer; (0, MAX (UINT32)]: set the weight to this value, others: illegal |
| tx_threshold    | string | Optional, by default "", it means the minimum privilege for the account. "", do not set this value; "0": set `tx_threshold` weight to 0; ("0", "MAX(INT64)"]: set the weight value to this value; others: illegal. |
| type_thresholds | array  | Optional, a list of thresholds ​​required for different operations; by default is an empty object. Empty objects are not set |
| type            | int    | To indicate a certain operation type  (0, 100]               |
| threshold       | int64  | Optional, by default is 0. 0: delete the type operation; (0, MAX(INT64)]: set the weight value to this value; Other: illegal |



### Transferring BU Assets

> **Note**: If the target account is a contract account, the current operation triggers the contract execution of the target account. 

- Function

  Two functions:

  1. The source account of this operation transfers a BU asset to the target account.
  2. The source account of this operation creates a new account on the blockchain.

- In protobuf format

```protobuf
message OperationPayCoin{
	string dest_address = 1;
	int64 amount = 2;
	string input = 3;
}
```

- protobufKeyword

| Keyword      | Type   | Description                                                  |
| ------------ | ------ | ------------------------------------------------------------ |
| dest_address | string | The target account                                           |
| amount       | array  | Optional, a list of signers that need to operate. By default is an empty object. Empty objects are not set. |
| input        | string | Optionally, if the target account is a contract account, and the input will be passed to the argument of the `main` function of the contract code. This setting is invalid if the target account is a normal account. |

### Recording Logs

- Function

 The source account of this operation writes the log to the blockchain.

- In protobuf format

```protobuf
message OperationLog{
	string topic = 1;
	repeated string datas = 2;
}
```

- protobufKeyword

| Keyword | Type   | Description                                             |
| ------- | ------ | ------------------------------------------------------- |
| topic   | string | The log topic and the parameter length is (0,128]       |
| datas   | array  | The log content. The length of each element is (0,1024] |



## Websocket Interfaces

The websocket interface of BUMO handles various defined message types.

### Message Type

```protobuf
enum ChainMessageType {
	CHAIN_TYPE_NONE = 0;
	CHAIN_HELLO = 10; // response with CHAIN_STATUS = 2;
	CHAIN_TX_STATUS = 11;
	CHAIN_PEER_ONLINE = 12;
	CHAIN_PEER_OFFLINE = 13;
	CHAIN_PEER_MESSAGE = 14;
	CHAIN_SUBMITTRANSACTION = 15;
	CHAIN_LEDGER_HEADER = 16; //bumo notifies the client ledger(protocol::LedgerHeader) when closed
	CHAIN_SUBSCRIBE_TX = 17; //response with CHAIN_RESPONSE
	CHAIN_TX_ENV_STORE = 18;
}
```



### Notification Message Registration

- Function

  The client registers the message with the blockchain through the interface, that is, the type of the message that needs to be received (currently the function is unavailable). The version information of the blockchain can only be obtained through this interface.

- Request Message Type

  `CHAIN_HELLO`

- Request Data Object

  ```protobuf
  message ChainHello {
  	repeated ChainMessageType api_list = 1;	//By default, enable all apis
  	int64	timestamp = 2;
  }
  ```

- Request Parameter

  | Parameter      | Type             | Description               |
  | --------- | ---------------- | ------------------ |
  | api_list  | array<ChainMessageType> | List of message types for applying for registration |
  | timestamp | int64            | Application time           |

- Reponse Message Type

  `CHAIN_HELLO`

- Reponse Data Object

  ```protobuf
  message ChainStatus {
  	string self_addr		= 1;
  	int64 ledger_version	= 2;
  	int64 monitor_version	= 3;
  	string bumo_version		= 4;
  	int64	timestamp		= 5;
  }
  ```

- Reponse Result

  | Variable        | Type   | Description           |
  | --------------- | ------ | -------------- |
  | self_addr       | string | Connected node address |
  | ledger_version  | int64  | Ledger version     |
  | monitor_version | int64  | Monitor version |
  | bumo_version    | string | BUMO version |
  | timestamp       | int64  | Timestamp         |



### Submit Transaction

- Function

  The transaction that will need to be executed is sent to the blockchain  execution through the message type. Please refer to [Transaction](#Transaction) for details of the transaction structure.

- Request Message Type

  `CHAIN_SUBMITTRANSACTION`

- Request Message Object

  ```protobuf
  //　Signature
  message Signature {
  	string public_key = 1;
  	bytes sign_data = 2;
  }
  
  //　Request object
  message TransactionEnv {
  	Transaction transaction = 1;
  	repeated Signature signatures 	= 2;
  	Trigger trigger = 3;
  }
  ```

- Request Parameter

  | Parameter      | Type             | Description                  |
  | ----------- | ----------- | ---------------------------------------- |
  | transaction | Transaction | Details for [Transaction](#transaction)。                  |
  | public_key  | string      | The public key of transaction sender |
  | sign_data   | bytes       | Signature data obtained by signing `transaction_blob` |

- Reponse Message Type

  - `CHAIN_TX_STATUS`：Returns the result of the transaction submission (the successful submission of the transaction does not mean that the transaction was executed successfully).
  - `CHAIN_TX_ENV_STORE`：Returns the result of the transaction execution.

- `CHAIN_TX_STATUS` Object

  ```protobuf 
  message ChainTxStatus {
  	enum TxStatus {
  		UNDEFINED	= 0;
  		CONFIRMED	= 1;	// web server will check tx parameters, signatures etc fist, noitfy CONFIRMED if pass
  		PENDING		= 2;	// master will check futher before put it into pending queue
  		COMPLETE	= 3;	// notify if Tx write ledger successfully
  		FAILURE		= 4;	// notify once failed and set error_code
  	};
  
  	TxStatus	status = 1;
  	string		tx_hash = 2;
  	string		source_address = 3;
  	int64		source_account_seq = 4;
  	int64		ledger_seq = 5;			//on which block this tx records
  	int64		new_account_seq = 6;		//new account sequence if COMPLETE
  	ERRORCODE	error_code = 7;			//use it if FAIL
  	string		error_desc = 8	;			//error desc
  	int64		timestamp = 9;			
  }
  ```

- `ChainTxStatus` Member

  | Variable           | Type   | Description                       |
  | ------------------ | --------- | -------------------------- |
  | status             | TxStatus  | Transaction status. |
  | tx_hash            | string    | Transaction hash. |
  | source_address     | string    | Source account address of transaction sender. |
  | source_account_seq | int64     | Transaction Sequence of transaction sender. |
  | ledger_seq         | int64     | The height of the block where this transaction record is located. |
  | new_account_seq    | int64     | Block height when the transaction is completed. |
  | error_code         | ERRORCODE | Error code           |
  | error_desc         | string    | Error description |
  | timestamp          | int64     | Timestamp       |

- `CHAIN_TX_ENV_STORE` Object

  ```protobuf
  message TransactionEnvStore{
  	TransactionEnv transaction_env = 1;
  	int32 error_code = 2;
  	string error_desc = 3;
  	int64 ledger_seq = 4;
  	int64 close_time = 5;
  	//for notify
  	bytes hash = 6;
  	int64 actual_fee = 7;
  	repeated bytes contract_tx_hashes = 8;
  }
  ```

- `TransactionEnvStore` Member

  | Variable           | Type   | Description                 |
  | ------------------ | -------------- | -------------------- |
  | transaction_env    | TransactionEnv | The submitted transaction content. |
  | error_code         | int32          | Error code. |
  | error_desc         | string         | Error desciption. |
  | ledger_seq         | int64          | The height of the block where this transaction record is located. |
  | close_time         | int64          | Transaction completion time. |
  | hash               | bytes          | Transaction hash. |
  | actual_fee         | int64          | Actual transaction fee，uint is MO |
  | contract_tx_hashes | bytes          | contract transaction hashes |




### Message Subscription

- Function

  This interface implements transaction notifications that only specify the account address of the interface.

- Request Message Type

  `CHAIN_SUBSCRIBE_TX`

- Request Data Object

  ```protobuf
  message ChainSubscribeTx{
  	repeated string address = 1;
  }
  ```

- Request Parameter

  | Parameter | Type | Description         |
  | ------- | ----------- | ------------------ |
  | address | Transaction | The account address to subscribe to. |

- Reponse Message Type

  `ChainResponse`

- Reponse Data Object

  ```protobuf 
  message ChainResponse{
  		int32 error_code = 1;
  		string error_desc = 2;
  }
  ```

- Reponse Result

  | Variable   | Type   | Description     |
  | ---------- | ------ | -------- |
  | error_code | int32  | Error code |
  | error_desc | string | Error description |



## Error Codes

  The error code is composed of two parts:

- error_code : Error code, approximate error classification
- error_desc : Error Description, which can accurately find the error specific information from the error description

The error list is as follows:

| Error code | Name                                   | Description                                                  |
| ---------- | -------------------------------------- | ------------------------------------------------------------ |
| 0          | ERRCODE_SUCCESS                        | Successful operation                                         |
| 1          | ERRCODE_INTERNAL_ERROR                 | Inner service defect                                         |
| 2          | ERRCODE_INVALID_PARAMETER              | Parameters error                                             |
| 3          | ERRCODE_ALREADY_EXIST                  | Objects already exist, such as repeated transactions submitted |
| 4          | ERRCODE_NOT_EXIST                      | Objects do not exist, such as null account, transactions and blocks etc |
| 5          | ERRCODE_TX_TIMEOUT                     | Transactions expired. It means the transaction has been removed from the buffer, but it still has probability to be executed |
| 7          | ERRCODE_MATH_OVERFLOW                  | Math calculation is overflown                                |
| 20         | ERRCODE_EXPR_CONDITION_RESULT_FALSE    | The expression returns false. It means the TX failed to be executed currently, but it still has probability to be executed in the following blocks |
| 21         | ERRCODE_EXPR_CONDITION_SYNTAX_ERROR    | The syntax of the expression returns is false. It means that the TX must fail |
| 90         | ERRCODE_INVALID_PUBKEY                 | Invalid public key                                           |
| 91         | ERRCODE_INVALID_PRIKEY                 | Invalid private key                                          |
| 92         | ERRCODE_ASSET_INVALID                  | Invalid assets                                               |
| 93         | ERRCODE_INVALID_SIGNATURE              | The weight of the signature does not match the threshold of the operation |
| 94         | ERRCODE_INVALID_ADDRESS                | Invalid address                                              |
| 97         | ERRCODE_MISSING_OPERATIONS             | Absent operation of TX                                       |
| 98         | ERRCODE_TOO_MANY_OPERATIONS            | Over 100 operations in a single transaction                  |
| 99         | ERRCODE_BAD_SEQUENCE                   | Invalid sequence or nonce of TX                              |
| 100        | ERRCODE_ACCOUNT_LOW_RESERVE            | Low reserve in the account                                   |
| 101        | ERRCODE_ACCOUNT_SOURCEDEST_EQUAL       | Sender and receiver accounts are the same                    |
| 102        | ERRCODE_ACCOUNT_DEST_EXIST             | The target account already exists                            |
| 103        | ERRCODE_ACCOUNT_NOT_EXIST              | Accounts do not exist                                        |
| 104        | ERRCODE_ACCOUNT_ASSET_LOW_RESERVE      | Low reserve in the account                                   |
| 105        | ERRCODE_ACCOUNT_ASSET_AMOUNT_TOO_LARGE | Amount of assets exceeds the limitation ( int64 )            |
| 106        | ERRCODE_ACCOUNT_INIT_LOW_RESERVE       | Insufficient initial reserve for account creation            |
| 111        | ERRCODE_FEE_NOT_ENOUGH                 | Low transaction fee                                          |
| 114        | ERRCODE_OUT_OF_TXCACHE                 | TX buffer is full                                            |
| 120        | ERRCODE_WEIGHT_NOT_VALID               | Invalid weight                                               |
| 121        | ERRCODE_THRESHOLD_NOT_VALID            | Invalid threshold                                            |
| 144        | ERRCODE_INVALID_DATAVERSION            | Invalid data version of metadata                             |
| 146        | ERRCODE_TX_SIZE_TOO_BIG                | TX exceeds upper limitation                                  |
| 151        | ERRCODE_CONTRACT_EXECUTE_FAIL          | Failure in contract execution                                |
| 152        | ERRCODE_CONTRACT_SYNTAX_ERROR          | Failure in syntax analysis                                   |
| 153        | ERRCODE_CONTRACT_TOO_MANY_RECURSION    | The depth of contract recursion exceeds upper limitation     |
| 154        | ERRCODE_CONTRACT_TOO_MANY_TRANSACTIONS | the TX submitted from the  contract exceeds upper limitation |
| 155        | ERRCODE_CONTRACT_EXECUTE_EXPIRED       | Contract expired                                             |
| 160        | ERRCODE_TX_INSERT_QUEUE_FAIL           | Failed to insert the TX into buffer                          |