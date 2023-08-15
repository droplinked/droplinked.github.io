# BlockChain front-end function interface

## Login

Login with wallet is done using Metamask wallet, using a typically `login{ChainName}` function, which will open up a Wallet window, and user should Sign a message to login.

The result of login contains the following information: 

- `address` | `publicKey`: the address of the wallet

- `signature`: the signature of the message signed by the wallet

- `network` : either `mainnet` or `testnet`

- `account_hash` : Some chains (like casper) have a different account identifier than the address which is the account hash. (This is not used in Polygon or other EVM chains)

        Below is the example of the login function
    ```javascript
        let account_info = await PolygonLogin();
        console.log(account_info);
    ```

The returned result should be used to verify the user's identity in the backend.

## Decentralization

### Record Product

Record product is done using `record_merch` function, which will record the product information on the blockchain. The function will return 
`deploy_hash` (or `tx_hash`), which should be used to verify the transaction on the blockchain (on Web3 project).

The information you'll need to record are:

- `product_title` (`string`): the name of the product

- `price` (`number`): the price of the product (Note that this price is multiplied in 100, so 1$ would be passed as 100)

- `amount` (`number`): the amount to mint 

- `product_description` (`string`): the description of the product

- `image_url` (`string`) : the image of the product (image URL)

- `producer_comission` (`number`): the commission of the producer (Note that comission must be multiplied by 100, so 12.34% would be passed as 1234)

- `address` (`string`): the address of the wallet (would be passed from the login result)

- `sku_properties` (`{}`): the properties of the product (Passed in as a JSON object)

        Below is the example of the record_merch function
    ```javascript
    // Get Account information from login
    let account_information = await PolygonLogin();
    let product_title = "test product";
    let discription = "test product description";
    let image_url = "https://i.imgur.com/removed.png";
    let price = 200; // It is actually 2 dollars (the price should be multiplied by 100)
    let amount = 2000;
    let comission = 1234; // It is actually 12.34% (the comission should be multiplied by 100)
    let tx_hash = await record_merch({
        "type" : "t-shirt",
        "size" : "large",
        "color" : "red"
    } , account_information.address, product_title , discription, image_url, price, amount, comission);
    console.log(tx_hash);
    ```

### Publish Request

Publish request is done using `publish_request` function, which will create a new request from the publisher to the producer. The function will 
return `deploy_hash` (or `tx_hash`), which should be used to verify the transaction on the blockchain (on Web3 project).

The information you'll need to use publish request are:

- `address` (`string`): the address of the wallet (would be passed from the login result)

- `producer_address` (`string`): the address of the producer

- `token_id` (`number`): the token id of the product (passed from the Web3 project)

        Below is the example of the publish_request function
    ```javascript
    let account_information = await PolygonLogin();
    let producer_address = "0x89281F2dA10fB35c1Cf90954E1B3036C3EB3cc78";
    let token_id = 1;
    publish_request(account_information.address, producer_address, token_id);
    ```

### Approve Request

Approve request is done using `approve_request` function, which will approve the request from the publisher to the producer. The function will 

return `deploy_hash` (or `tx_hash`), which should be used to verify the transaction on the blockchain (on Web3 project).

The information you'll need to use approve request are:

- `address` (`string`): the address of the wallet (would be passed from the login result)

- `request_id` (`number`): the request id of the request (passed from the Web3 project)

        Below is the example of the approve_request function
    ```javascript
    let account_information = await PolygonLogin();
    let request_id = 1;
    approve_request(account_information.address, request_id);
    ```

### Cancel Request

Cancel request is done using `cancel_request` function, which will cancel the request from the publisher to the producer. The function will

return `deploy_hash` (or `tx_hash`), which should be used to verify the transaction on the blockchain (on Web3 project).

The information you'll need to use cancel request are:

- `address` (`string`): the address of the wallet (would be passed from the login result)

- `request_id` (`number`): the request id of the request (passed from the Web3 project)

        Below is the example of the cancel_request function
    ```javascript
    let account_information = await PolygonLogin();
    let request_id = 1;
    cancel_request(account_information.address, request_id);
    ```

### Disapprove Request

Disapprove request is done using `disapprove_request` function, which will disapprove the request from the publisher to the producer. The function will return `deploy_hash` (or `tx_hash`), which should be used to verify the transaction on the blockchain (on Web3 project).

The information you'll need to use disapprove request are:

- `address` (`string`): the address of the wallet (would be passed from the login result)

- `request_id` (`number`): the request id of the request (passed from the Web3 project)

        Below is the example of the disapprove_request function
    ```javascript
    let account_information = await PolygonLogin();
    let request_id = 1;
    disapprove(account_information.address, request_id);
    ```
