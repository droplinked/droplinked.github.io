
# <u>Droplinked Decentralization</u>

## • <u>Authentication Process</u>

The `casper_login` method is called to authenticate the user. This method receives a function as input. The input to the function is `acc-info`. `acc-info` is the user's account information which consists of three parts:  
• publicKey  
• account_hash (it is used only in casper chain)  
• signature  
Below you can see an implementation of the `casper_login` method.

```javascript
import { CLPublicKey } from "casper-js-sdk";
let casperWalletInstance;
export let account_information;
export const getCasperWalletInstance = () => {
  try {
    if (casperWalletInstance == null) {
      casperWalletInstance = window.CasperWalletProvider();
    }
    return casperWalletInstance;
  } catch (err) {}
  throw Error("Please install the Casper Wallet Extension.");
};

export const isCasperWalletExtentionInstalled = () => {
  try {
    let walletInstance = window.CasperWalletProvider();
    return walletInstance != null;
  } catch (error) {
    return false;
  }
};

let get_account_information = async function (publicKey) {
  let sign = await getCasperWalletInstance().signMessage(
    "Please sign this message to let droplinked access to your PublicKey and validate your identity.",
    await getCasperWalletInstance().getActivePublicKey()
  );
  return {
    publicKey: publicKey,
    account_hash: CLPublicKey.fromHex(publicKey).toAccountRawHashStr(),
    signature: sign.signatureHex,
  };
};
export async function casper_login(on_connected) {
  let called = false;
  await getCasperWalletInstance().requestConnection();
  if (await getCasperWalletInstance().isConnected()) {
    if (!called) {
      called = true;
      on_connected(
        await get_account_information(
          await getCasperWalletInstance().getActivePublicKey()
        )
      );
    }
    return;
  }
  await getCasperWalletInstance().requestConnection();
  const handleConnected = async (event) => {
    try {
      const action = JSON.parse(event.detail);
      if (action.activeKey) {
        if (!called) {
          called = true;
          on_connected(await get_account_information(action.activeKey));
        }
      }
    } catch (err) {
      console.log(err);
    }
  };
  window.addEventListener(CasperWalletEventTypes.Connected, handleConnected);
  if (!called)
    on_connected(
      await get_account_information(
        await getCasperWalletInstance().getActivePublicKey()
      )
    );
}
```

## • <u>Affiliate Operations</u>

### - <u>Recording a Product</u>

Each product has unique properties such as color, size, title, pictures, etc. The method `record_merch` is used to record a product on blockchain. The method inputs are as follows:  
• sku_properties: the product's information  
• account_information: the information of the user who is recording the product. The information is available after the user's login.  
• product_title: the title of the product  
• price: the price of the product  
• amount: the quantity of the product  
• commission: publisher's commission of the product  
The function `record_merch` then returns a `deploy_hash`.

Product recording process consists of the following steps:

1. The product information is uploaded on ipfs and an address is created for the product. This address can be used in a url as `ipfs.io/ipfs/{address}` to access the product's information. `uploadToIPFS` gets the product's metadata as input and uploads the metadata in ipfs to get an address. The function then returns the hash of that address.
2. The function `record_merch` returns a `deploy_hash`.
3. `deploy_hash` is sent to backend and then to the web3 project.
4. The `deploy_hash` is validated in the web3 project to verify the transaction.
5. If the product has never been recorded before, the web3 project uses the validated `deploy_hash` to create a new `token_id` for the product.
6. A new `holder_id` containing the `token_id` and the `amount` of the product is created. The `holder_id` is saved to the producer's account.
7. The web3 project returns the created `token_id` and `holder_id`.
8. A request is sent to backend to update the database, storing `token_id` and `holder_id` in the database.
9. If the recording was successful, you can see the product as a recorded product after refreshing the frontend.

```javascript
import * as casper_consts from "./constants";
import {
  CLAccountHash,
  CLKey,
  CLPublicKey,
  CLString,
  CLU256,
  CLU64,
  Contracts,
  DeployUtil,
  NamedArg,
  RuntimeArgs,
} from "casper-js-sdk";
import { NFTStorage } from "nft.storage";
import { getCasperWalletInstance } from "./casper_wallet_auth";

const client = new NFTStorage({
  token:
    "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJkaWQ6ZXRocjoweDQ1MjMzMDAzNDY0YzcyNkNhY2QyOEIyMTkyYWFBNDdhMDg2MmJmQzUiLCJpc3MiOiJuZnQtc3RvcmFnZSIsImlhdCI6MTY3NTE3NzYwNzI1NywibmFtZSI6ImRyb3BsaW5rZWRfTkZUIn0.B44NWDZ7GAORBwXB36hLEw3VuWG8tOYRl8g6QNOUv-Q",
});
async function uploadToIPFS(metadata) {
  if (typeof metadata == typeof {} || typeof metadata == typeof []) {
    metadata = JSON.stringify(metadata);
  }
  const ipfs_hash = await client.storeBlob(new Blob([metadata]));
  return ipfs_hash;
}
async function get_sha256(sku_properties) {
  if (typeof sku_properties == typeof {}) {
    sku_properties = JSON.stringify(sku_properties);
  }
  const hashBuffer = await window.crypto.subtle.digest(
    "SHA-256",
    new TextEncoder().encode(sku_properties)
  );
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  const digest = hashArray.map((b) => b.toString(16).padStart(2, "0")).join("");
  return digest;
}
async function record_merch(
  sku_properties,
  account_information,
  product_title,
  price,
  amount,
  comission
) {
  let IPFSHASH = await uploadToIPFS(sku_properties);
  let checksum = await get_sha256(sku_properties);
  let producer_public_key = account_information.publicKey;
  const publicKey = CLPublicKey.fromHex(producer_public_key);
  let gasPrice = 8 * 1000000000;
  const ttl = 1800000;
  let deployParams = new DeployUtil.DeployParams(
    publicKey,
    casper_consts.network,
    1,
    ttl
  );
  let contract_hash_string = casper_consts.contract_hash;
  let contract_byte_array =
    Contracts.contractHashToByteArray(contract_hash_string);
  let named_args = [];
  named_args.push(
    new NamedArg(
      "metadata",
      new CLString(
        `{ "name" : "${product_title}" , "token_uri" : "${IPFSHASH}" , "checksum" : "${checksum}"}`
      )
    )
  );
  named_args.push(new NamedArg("price", new CLU64(price)));
  named_args.push(new NamedArg("amount", new CLU64(amount)));
  named_args.push(new NamedArg("comission", new CLU64(comission)));
  let recipient = CLPublicKey.fromHex(producer_public_key).toAccountHash();
  named_args.push(
    new NamedArg("recipient", new CLKey(new CLAccountHash(recipient)))
  );
  let runtime_args = RuntimeArgs.fromNamedArgs(named_args);
  const kk = DeployUtil.ExecutableDeployItem.newStoredContractByHash(
    contract_byte_array,
    "mint",
    runtime_args
  );
  const payment = DeployUtil.standardPayment(gasPrice);
  let deploy = DeployUtil.makeDeploy(deployParams, kk, payment);
  const json = DeployUtil.deployToJson(deploy);
  const signature = await getCasperWalletInstance()
    .sign(JSON.stringify(json), producer_public_key)
    .catch((reason) => {
      return "Cancelled";
    });
  const signedDeploy = DeployUtil.setSignature(
    deploy,
    signature.signature,
    CLPublicKey.fromHex(producer_public_key)
  );
  const deployres = await casper_consts.casperService.deploy(signedDeploy);
  return { deploy: deployres, deployHash: deployres.deploy_hash };
}
export { record_merch };
```

### - <u>Publish Request</u>

A publisher can send a request to the producer to access the recorded product for co-selling procedure. To send this request, the publisher calls the `publish_request` method. This method receives the following inputs:  
• holder_id: the holder_id related to a recorded product that is stored in the producer's account  
• amount: amount of the recorded products that the publisher requests to have access to (it is going to be omitted)  
• producer_account_hash: producer's account information hash  
• account_info: publisher's account information  
Upon calling the `publish_request` method a new `PublishRequest` is created which is identified by a `request_id`.

```javascript
import {
  CLAccountHash,
  CLKey,
  CLPublicKey,
  CLU64,
  Contracts,
  DeployUtil,
  NamedArg,
  RuntimeArgs,
} from "casper-js-sdk";
import * as casper_consts from "./constants";
import { getCasperWalletInstance } from "./casper_wallet_auth";
/**
 * Deploys the publish request
 * @param {int} holder_id
 * @param {int} amount
 * @param {int} comission
 * @param {string} producer_account_hash
 * @param {{'publicKey' : string , 'account_hash' : string, 'signature' : string}} account_info
 * @returns
 */
export let publish_request = async function (
  holder_id,
  amount,
  producer_account_hash,
  account_info
) {
  const fromHexString = (hexString) =>
    Uint8Array.from(
      hexString.match(/.{1,2}/g).map((byte) => parseInt(byte, 16))
    );
  let producer_hash = new CLAccountHash(fromHexString(producer_account_hash));
  console.log(producer_hash);
  const publicKeyHex = account_info.publicKey;
  const publicKey = CLPublicKey.fromHex(publicKeyHex);
  let gasPrice = 3800000000;
  const ttl = 1800000;
  let deployParams = new DeployUtil.DeployParams(
    publicKey,
    casper_consts.network,
    1,
    ttl
  );
  let contract_hash_string = casper_consts.contract_hash;
  let contract_byte_array =
    Contracts.contractHashToByteArray(contract_hash_string);
  let named_args = [];
  named_args.push(new NamedArg("holder_id", new CLU64(holder_id)));
  named_args.push(new NamedArg("amount", new CLU64(amount)));
  named_args.push(new NamedArg("producer-account", new CLKey(producer_hash)));
  let runtime_args = RuntimeArgs.fromNamedArgs(named_args);
  const kk = DeployUtil.ExecutableDeployItem.newStoredContractByHash(
    contract_byte_array,
    "publish_request",
    runtime_args
  );
  const payment = DeployUtil.standardPayment(gasPrice);
  let deploy = DeployUtil.makeDeploy(deployParams, kk, payment);
  const json = DeployUtil.deployToJson(deploy);
  console.log(publicKeyHex);
  console.log(json);
  const signature = await getCasperWalletInstance()
    .sign(JSON.stringify(json), publicKeyHex)
    .catch((reason) => {
      return "Cancelled";
    });
  const signedDeploy = DeployUtil.setSignature(
    deploy,
    signature.signature,
    CLPublicKey.fromHex(publicKeyHex)
  );
  const deployres = await casper_consts.casperService.deploy(signedDeploy);
  return { deploy: deployres, deployHash: deployres.deploy_hash };
};
```

### - <u>Approving a Request</u>

After creating a `request_id`, a new request is placed in the producer's incoming requests. To approve a publisher's request, the producer calls the `approve_request` method. This method receives the following inputs:  
• request_id: the request that the producer tends to approve  
• account_info: producer's account information  
Upon approving a request, an `approved_id` is created for the request. The `approved_id` is saved in the web3 project.

```javascript
import {
  CLPublicKey,
  CLU64,
  Contracts,
  DeployUtil,
  NamedArg,
  RuntimeArgs,
} from "casper-js-sdk";
import * as casper_consts from "./constants";
import { getCasperWalletInstance } from "./casper_wallet_auth";
/**
 * Approves the request with request_id using casper-signer
 * @param {int} request_id
 * @param {{'publicKey' : string , 'account_hash' : string, 'signature' : string}} account_info
 * @returns
 */
export let approve_request = async function (request_id, account_info) {
  const publicKeyHex = account_info.publicKey;
  const publicKey = CLPublicKey.fromHex(publicKeyHex);
  let gasPrice = 8941000000;
  const ttl = 1800000;
  let deployParams = new DeployUtil.DeployParams(
    publicKey,
    casper_consts.network,
    1,
    ttl
  );
  let contract_hash_string = casper_consts.contract_hash;
  let contract_byte_array =
    Contracts.contractHashToByteArray(contract_hash_string);
  let named_args = [];
  named_args.push(new NamedArg("request_id", new CLU64(request_id)));
  let runtime_args = RuntimeArgs.fromNamedArgs(named_args);
  const kk = DeployUtil.ExecutableDeployItem.newStoredContractByHash(
    contract_byte_array,
    "approve",
    runtime_args
  );
  const payment = DeployUtil.standardPayment(gasPrice);
  let deploy = DeployUtil.makeDeploy(deployParams, kk, payment);
  const json = DeployUtil.deployToJson(deploy);
  const signature = await getCasperWalletInstance()
    .sign(JSON.stringify(json), publicKeyHex)
    .catch((reason) => {
      return "Cancelled";
    });
  const signedDeploy = DeployUtil.setSignature(
    deploy,
    signature.signature,
    CLPublicKey.fromHex(publicKeyHex)
  );
  const deployres = await casper_consts.casperService.deploy(signedDeploy);
  return { deploy: deployres, deployHash: deployres.deploy_hash };
};
```

### - <u>Disapproving a Request</u>

The producer can disapprove an approved request at any time, so the publisher would not have access to the product anymore. For disapproving a request, the `disapprove_request` method is called. This method receives the following inputs:  
• approved_id: the id that represents an approved request  
• publisher_account_hash: representing the publisher which the producer decided to disapprove  
• account_info: producer's account information  
By calling this function, the publisher's access to the product is disapproved.

```javascript
import {
  CLAccountHash,
  CLKey,
  CLPublicKey,
  CLU64,
  Contracts,
  DeployUtil,
  NamedArg,
  RuntimeArgs,
} from "casper-js-sdk";
import * as casper_consts from "./constants";
import { getCasperWalletInstance } from "./casper_wallet_auth";
/**
 * Approves the request with request_id using casper-signer
 * @param {int} request_id
 * @param {{'publicKey' : string , 'account_hash' : string, 'signature' : string}} account_info
 * @returns
 */
export let disapprove_request = async function (
  approved_id,
  amount,
  publisher_account_hash,
  account_info
) {
  const fromHexString = (hexString) =>
    Uint8Array.from(
      hexString.match(/.{1,2}/g).map((byte) => parseInt(byte, 16))
    );
  let publisher_hash = new CLAccountHash(fromHexString(publisher_account_hash));
  const publicKeyHex = account_info.publicKey;
  const publicKey = CLPublicKey.fromHex(publicKeyHex);
  let gasPrice = 8941000000;
  const ttl = 1800000;
  let deployParams = new DeployUtil.DeployParams(
    publicKey,
    casper_consts.network,
    1,
    ttl
  );
  let contract_hash_string = casper_consts.contract_hash;
  let contract_byte_array =
    Contracts.contractHashToByteArray(contract_hash_string);
  let named_args = [];
  named_args.push(new NamedArg("approved_id", new CLU64(approved_id)));
  named_args.push(new NamedArg("amount", new CLU64(amount)));
  named_args.push(new NamedArg("publisher-account", new CLKey(publisher_hash)));

  let runtime_args = RuntimeArgs.fromNamedArgs(named_args);
  const kk = DeployUtil.ExecutableDeployItem.newStoredContractByHash(
    contract_byte_array,
    "disapprove",
    runtime_args
  );
  const payment = DeployUtil.standardPayment(gasPrice);
  let deploy = DeployUtil.makeDeploy(deployParams, kk, payment);
  const json = DeployUtil.deployToJson(deploy);
  const signature = await getCasperWalletInstance()
    .sign(JSON.stringify(json), publicKeyHex)
    .catch((reason) => {
      return "Cancelled";
    });
  const signedDeploy = DeployUtil.setSignature(
    deploy,
    signature.signature,
    CLPublicKey.fromHex(publicKeyHex)
  );
  const deployres = await casper_consts.casperService.deploy(signedDeploy);
  return { deploy: deployres, deployHash: deployres.deploy_hash };
};
```

### - <u>Cancelling a Request</u>

To cancel a request, the publisher needs to call the `cancel_request` method. This method receives the following input:  
• request_id: the request that the publisher tends to cancel  
• account_info: publisher's account information  
`cancel_request` then removes the request.

```javascript
import {
  CLPublicKey,
  CLU64,
  Contracts,
  DeployUtil,
  NamedArg,
  RuntimeArgs,
} from "casper-js-sdk";
import * as casper_consts from "./constants";
import { getCasperWalletInstance } from "./casper_wallet_auth";
/**
 * Approves the request with request_id using casper-signer
 * @param {int} request_id
 * @param {{'publicKey' : string , 'account_hash' : string, 'signature' : string}} account_info
 * @returns
 */
export let cancel_request = async function (request_id, account_info) {
  const publicKeyHex = account_info.publicKey;
  const publicKey = CLPublicKey.fromHex(publicKeyHex);
  let gasPrice = 8941000000;
  const ttl = 1800000;
  let deployParams = new DeployUtil.DeployParams(
    publicKey,
    casper_consts.network,
    1,
    ttl
  );
  let contract_hash_string = casper_consts.contract_hash;
  let contract_byte_array =
    Contracts.contractHashToByteArray(contract_hash_string);
  let named_args = [];
  named_args.push(new NamedArg("request_id", new CLU64(request_id)));
  let runtime_args = RuntimeArgs.fromNamedArgs(named_args);
  const kk = DeployUtil.ExecutableDeployItem.newStoredContractByHash(
    contract_byte_array,
    "cancel_request",
    runtime_args
  );
  const payment = DeployUtil.standardPayment(gasPrice);
  let deploy = DeployUtil.makeDeploy(deployParams, kk, payment);
  const json = DeployUtil.deployToJson(deploy);
  const signature = await getCasperWalletInstance()
    .sign(JSON.stringify(json), publicKeyHex)
    .catch((reason) => {
      return "Cancelled";
    });
  const signedDeploy = DeployUtil.setSignature(
    deploy,
    signature.signature,
    CLPublicKey.fromHex(publicKeyHex)
  );
  const deployres = await casper_consts.casperService.deploy(signedDeploy);
  return { deploy: deployres, deployHash: deployres.deploy_hash };
};
```
