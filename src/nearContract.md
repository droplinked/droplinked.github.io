
## <u>Contract Objects (Entities) </u>

> _**Here we explain each structure used within the contract and how they are used**_

Each contract has a storage section which is used to store data. This data can be of any type, including structs. Also there are some built-in functionalities on different chains that can be leveraged for storing data. For example, we have the ability to store data in a dictionary on the Casper network (which is common in lots of chains). 

_**Assuming we can use dictionaries, we use the following structures to store data in our contract:**_

_No dictionary? move [here](nodict.md)_

_No Loop in Contract Language? read [noLoop](noloop.md) docs_
> - **NFTHolder** 


```rust
/*
*   This struct holds the token ID and its amount for a specific account.
*/
pub struct NFTHolder {
    pub token_id: u128,
    pub amount: u128,
}
```

> - **NFTMetadata**

```rust
/*
*   This struct holds the metadata of a token. It has a name, a URI(it can be IPFS hash), and a checksum (the hash of the file uploaded off-chain), 
*   and a price (in USD). Also it has a commission which is the amount of commission that a publisher will get for selling this token.
*/
pub struct NFTMetadata {
    pub name: String,
    pub uri: String,
    pub checksum: String,
    pub price: u128,
    pub commission: u8,
}
```

> - **PublishRequest**


```rust
/*
*   This struct holds the request of a publisher to a producer to publish a token. It has a holder_id, amount, a publisher address and a producer 
*   address. this struct will be saved in a dictionary which maps a request_id to a PublishRequest.
*/
pub struct PublishRequest {
    pub holder_id: u128,
    pub amount: u128,
    pub publisher: Account,
    pub producer: Account
}
```

> - **ApprovedNFT**

```rust
/*
*   This struct holds the data of the approved tokens (for publishers), it has a holder_id, amount, owner and publisher account address, the 
*   token_id. After approving a PublishRequest by a producer, it will be saved in a dictionary which maps every 
*   approved_id to this object
*/
pub struct ApprovedNFT {
    pub holder_id: u128,
    pub amount: u128,
    pub owner: Account,
    pub publisher: Account,
    pub token_id: u128,
}
```

---

## <u>Contract Methods</u>

Here we explain each method used within the contract and how they are used. Note that these are signatures of the methods, and not the actual implementation. The actual implementation can be found in the contract files. Also note that methods are not limited to the ones listed here, and more methods can be added to the contract.

### <u>Mint</u>

```rust
/*
*   This method is used to mint a token. It takes a token_id, a name, a uri, a checksum, and a price. It will create a new NFTMetadata object
*   and save it in a dictionary which maps the token_id to the NFTMetadata object.
*   It will also create a new NFTHolder object and save it in a dictionary which maps the account address to list of NFTHolder ids.
*   The method will also emit a Mint event.
*   It could also return the holder_id of the newly minted token.
*/
pub fn mint(token_id: u128, name: String, uri: String, checksum: String, price: u128, commission : u8) -> u128;
```

### <u>Transfer</u>

```rust
/*
*   This method is used to transfer a token. It takes a holder_id, amount, and a receiver account address. It will get the NFTHolder object from
*   the dictionary which maps the account address to list of NFTHolder ids. It will then get the NFTHolder object from the list of NFTHolder ids
*   which has the same holder_id as the one passed to the method. It will then check if the amount passed to the method is less than or equal to the
*   amount of the NFTHolder object. If it is, it will subtract the amount from the NFTHolder object and save it in the dictionary. It will also
*   create a new NFTHolder object and save it in the dictionary which maps the receiver account address to list of NFTHolder ids. It will also
*   emit a Transfer event.
*   It could also return the holder_id of the newly transferred token.
*   Note that if the coresponding NFTHolder object has an amount of 0, it will be removed from the dictionary.
*   Note that if the holder's token_id could be found in receiver's list of NFTHolder ids, it will be updated with the new amount instead of 
*   creating a new NFTHolder object.
*  Note that this method can only be called by the owner of the token.
*/

pub fn transfer(holder_id: u128, amount: u128, receiver: Account) -> u128;
```


### <u>PublisRequest</u>

```rust
/*
*   This method is used to request a producer to publish a token. It takes a holder_id, amount, producer, and commission. It will create a new
*   PublishRequest object and save it in a dictionary which maps a request_id to the PublishRequest object.
*   It will also put the request_id in a list of request_ids for the producer and publisher (which maps their account into a list of request_ids).
*   It will also emit a PublishRequest event.
*   It could also return the request_id of the newly created PublishRequest.
*/
pub fn publish_request(producer : Account, holder_id: u128, amount: u128) -> u128;
```

### <u>Approve</u>

```rust
/*
*   This method is used to approve a PublishRequest. It takes a request_id. It will get the PublishRequest object from the dictionary which maps
*   the request_id to the PublishRequest object. It will then create a new ApprovedNFT object and save it in a dictionary which maps the
*   approved_id to the ApprovedNFT object. It will also put the approved_id in a list of approved_ids for the publisher and producer (which maps
*   their account into a list of approved_ids). It will also emit an Approve event.
*   It could also return the approved_id of the newly created ApprovedNFT.
*   Note that this method can only be called by the producer of the token.
*/
pub fn approve(request_id: u128) -> u128;
```

### <u>Buy</u>

We have two kinds of networks : one that has a pegged token (like USDT) and one that doesn't (like Casper). So based on the network, we have two different methods for buying tokens.

#### <u>Buy (Pegged Token)</u>

```rust
/*
*   This method is used to buy a token. It takes an approved_id, amount, and uses the `payment_contract` address from the contract's storage to 
*   get the transfer amount of USDTs from the caller's account to the producer and publisher's account with respect to the commission. It will
*   get the ApprovedNFT object from the dictionary which maps the approved_id to the ApprovedNFT object. It will then check if the amount passed
*   to the method is less than or equal to the amount of the ApprovedNFT object. If it is, it will subtract the amount from the ApprovedNFT object
*   and save it in the dictionary. 
*   It will also create a new NFTHolder object and save it in the dictionary which maps the account address of caller to list of NFTHolder ids.
*   It will also emit a Buy event.
*   It could also return the holder_id of the newly bought token.
*   Note : if the transfer call to the payment contract fails, the method will revert.
*   Note that if the coresponding NFTHolder object has an amount of 0, it will be removed from the dictionary.
*   Note that if the holder's token_id could be found in caller's list of NFTHolder ids, it will be updated with the new amount instead of
*   creating a new NFTHolder object.
*/
pub fn buy(approved_id: u128, amount: u128) -> u128;
```

#### <u>Buy (Non-Pegged Token)</u>

```rust
/*
*   This method is used to buy a token. It takes a approved_id, amount, and a price_ratio, and a signature of the price_ratio. It will get the
*   ApprovedNFT object from the dictionary which maps the approved_id to the ApprovedNFT object. It will then check if the amount passed to the
*   method is less than or equal to the amount of the ApprovedNFT object. If it is, it will subtract the amount from the ApprovedNFT object and
*   save it in the dictionary. Then it will get some amount of tokens from the caller's account, check if the amount is equal to the price_ratio * amount (Also, checks the signature of it), if it was, it will transfer the amount of tokens, with the commission, to the publisher's account and producer's account. It will also create a new NFTHolder object and save it in the dictionary which maps the account address of caller to list of NFTHolder ids. It will also emit a Transfer event.
*   It could also return the holder_id of the newly transferred token.
*   Note that if the coresponding ApprovedNFT object has an amount of 0, it will be removed from the dictionary.
*   Note that if the holder's token_id could be found in receiver's list of NFTHolder ids, it will be updated with the new amount instead of
*   creating a new NFTHolder object.
*/
pub fn buy(holder_id: u128, amount: u128, price: u128);
```