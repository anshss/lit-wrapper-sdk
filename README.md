## Lit Wrapper SDK

Using Lit Network has never been easier before
</br> An Ethereum private key is used to send requests to the Lit Network. Fund your wallet with a [faucet](https://chronicle-yellowstone-faucet.getlit.dev/) on Lit's custom rollup chain.

## Installation

```bash
npm install lit-wrapper-sdk
```

## Examples

### 1) Creating a Key on Solana and Sending Transaction

Using Lit to create a Solana Key and send a Txn with it, an Ethereum private key is used as an auth method for generating signatures with a newly created Solana Key. 

```js
import { LitWrapper } from "lit-wrapper-sdk";
import "dotenv/config";

const litWrapper = new LitWrapper("datil-dev")

async function generateSolanaWallet() {
    const res = await litWrapper.createSolanaWK(ETHEREUM_PRIVATE_KEY);
    console.log("Solana Public Key", res.wkInfo.generatedPublicKey);
}

async function sendSolTxn() {
    const signedTx = await litWrapper.sendSolanaWKTxnWithSol({
        amount: 0.0022 * Math.pow(10, 9),
        toAddress: "BTBPKRJQv7mn2kxBBJUpzh3wKN567ZLdXDWcxXFQ4KaV",
        network: "mainnet-beta",
        broadcastTransaction: true,
        userPrivateKey: ETHEREUM_PRIVATE_KEY,
        wkResponse: res.wkInfo,
        pkp: res.pkpInfo,
    });
    console.log("Transaction Hash: ", signedTx);
}

async function sendBONKTxn() {
    const signedTx = await litWrapper.sendSolanaWKTxnWithCustomToken({
        tokenMintAddress: "DezXAZ8z7PnrnRJjz3wXBoRgixCa6xjnB7YaB1pPB263", // BONK MINT TOKEN
        amount: 4 * Math.pow(10, 5),
        toAddress: "BTBPKRJQv7mn2kxBBJUpzh3wKN567ZLdXDWcxXFQ4KaV",
        network: "mainnet-beta",
        broadcastTransaction: true,
        userPrivateKey: ETHEREUM_PRIVATE_KEY,
        wkResponse: res.wkInfo,
        pkp: res.pkpInfo,
    });
    console.log("Transaction Hash: ", signedTx);
}
```

### 2) Creating Conditioned Signing on Solana

Checks against a conditional logic and only creates signatures for a specified transaction when condition is satisfies

```js
async function createLitActionAndSignSolanaTxn() {
    const response = await litWrapper.createSolanaWK(ETHEREUM_PRIVATE_KEY);
    console.log("Solana Public Key", response?.wkInfo.generatedPublicKey);

    const conditionLogic = `
    const url = "https://api.weather.gov/gridpoints/TOP/31,80/forecast";
    const resp = await fetch(url).then((response) => response.json());
    const temp = resp.properties.periods[0].temperature;

    console.log(temp);

    // only sign if the temperature is below 60
    if (temp < 60) {
        createSignatureWithAction();
    }`;

    const txn = await litWrapper.createSerializedLitTxn({
        wk: response?.wkInfo,
        toAddress: "BTBPKRJQv7mn2kxBBJUpzh3wKN567ZLdXDWcxXFQ4KaV",
        amount: 0.004 * Math.pow(10, 9),
        network: "mainnet-beta",
        flag: FlagForLitTxn.SOL,
    });

    const checkResult = await litWrapper.conditionalSigningOnSolana({
        userPrivateKey: ETHEREUM_PRIVATE_KEY,
        litTransaction: txn,
        conditionLogic,
        broadcastTransaction: true,
        wk: response?.wkInfo,
        pkp: response?.pkpInfo,
    });
    console.log(checkResult);
}
```

### 3) Creating a key on EVM and Executing a Lit Action

Create a key, Upload Lit Action to IPFS, Permit on IPFS and Execute the action.

```js
import { LitWrapper } from "lit-wrapper-sdk";
import "dotenv/config";

const litWrapper = new LitWrapper("datil-dev");

async function createKeyAndExecuteAction() {
    const _litActionCode = async () => {
        try {
            const sigShare = await Lit.Actions.ethPersonalSignMessageEcdsa({
                message: dataToSign,
                publicKey: pkpPublicKey,
                sigName,
            });
            Lit.Actions.setResponse({ response: sigShare });
        } catch (error) {
            Lit.Actions.setResponse({ response: error.message });
        }
    };
    const litActionCode = `(${_litActionCode.toString()})();`;

    const { pkp, ipfsCID } = await litWrapper.createPKPWithLitAction(
        process.env.ETHEREUM_PRIVATE_KEY,
        litActionCode,
        process.env.PINATA_API_KEY
    );

    const params = {
        dataToSign: ethers.utils.arrayify(
            ethers.utils.keccak256([1, 2, 3, 4, 5])
        ),
        sigName: "sig1",
    }

    const response = litWrapper.executeLitAction(usePrivateKey, pkp, ipfsCID, params);
    console.log(response)
}
```

### 4) Testing a Lit Action

Instantly create a Lit Action and test its execution over Lit Network.

```js
async function testAction() {
    if (!process.env.ETHEREUM_PRIVATE_KEY) {
        throw new Error("ETHEREUM_PRIVATE_KEY is not set");
    }

    const _litActionCode = async () => {
        try {
            const sigShare = await Lit.Actions.ethPersonalSignMessageEcdsa({
                message: dataToSign,
                publicKey: pkpPublicKey,
                sigName,
            });
            Lit.Actions.setResponse({ response: sigShare });
        } catch (error) {
            Lit.Actions.setResponse({ response: error.message });
        }
    };
    const litActionCode = `(${_litActionCode.toString()})();`;

    const tester = await LitTester.init(
        process.env.ETHEREUM_PRIVATE_KEY,
        "datil-dev"
    );

    const params = [
        {
            dataToSign: ethers.utils.arrayify(
                ethers.utils.keccak256([1, 2, 3, 4, 5])
            ),
            sigName: "sig1",
        }
    ];

    const results = await tester.testLitAction(litActionCode, params[0]);
    console.log("Test Results: ", results);
}
```
