# TERN BOT
## Buat folder & Pindah ke folder
```
mkdir t3rn-bot
cd t3rn-bot
```

## Buat bot file

```
nano main.js
```

- Isi pake ini
```
const { BigNumber } = require("ethers");
const { parseEther, formatEther } = require("ethers/lib/utils");
const ethers = require("ethers");
const fs = require("fs");

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms));
}

async function bridge(privateKey) {
  console.log("=========================================");
  let arbProvider = `https://base-sepolia.blockpi.network/v1/rpc/public`;
  const providerJSON = new ethers.providers.JsonRpcProvider(arbProvider);
  const wallet = new ethers.Wallet(privateKey, providerJSON);
  console.log(`Address: ${wallet.address}`);
  console.log(
    `Balance: ${formatEther(await providerJSON.getBalance(wallet.address))} ETH`
  );

  let addrBridge = "0x30A0155082629940d4bd9Cd41D6EF90876a0F1b5";
  let amountBridge = parseEther("0.01");
  console.log(`bridge amount ${formatEther(amountBridge)} ETH`);

  let dataBridge = `0x56591d596f707370000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000062a459f164fbb4acf8be5e2fed615dd85baa407000000000000000000000000000000000000000000000000002386e0fc36d11400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000002386f26fc10000`;

  let txEstimateGas = await wallet
    .estimateGas({
      to: addrBridge,
      value: amountBridge,
      data: dataBridge,
    })
    .catch((e) => ({ status: 500, message: e.reason }));

  if (BigNumber.isBigNumber(txEstimateGas)) {
    let txSend = await wallet
      .sendTransaction({
        to: addrBridge,
        value: amountBridge,
        data: dataBridge,
      })
      .catch((e) => ({ status: 500, message: e.reason, hash: null }));

    console.log(`√ Tx Bridge: https://sepolia.basescan.org/tx/${txSend.hash}`);
  } else {
    console.log(txEstimateGas);
  }
}

const readline = require("readline");

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
});

async function main() {
  rl.question(
    "Enter private keys separated by commas: ",
    async (keysAnswer) => {
      const privateKeys = keysAnswer
        .split(",")
        .filter((key) => key.trim() !== "");

      rl.question(
        "Enter the number of repetitions for all keys: ",
        async (repeatAnswer) => {
          const repetitions = parseInt(repeatAnswer);

          if (isNaN(repetitions) || repetitions <= 0) {
            console.log("Invalid number of repetitions. Exiting...");
            rl.close();
            return;
          }

          console.clear(); // Clear the terminal after entering repetitions

          for (let i = 0; i < repetitions; i++) {
            console.log(`\nStarting repetition ${i + 1}`);
            for (const privateKey of privateKeys) {
              await bridge(privateKey);
              await sleep(5000); // Optional: Sleep for 1 second between each transaction
            }
          }

          rl.close();
        }
      );
    }
  );
}

main().catch(console.error);
```

## Install dependenciesss
```
npm i ethers@5
```

## Run
```
screen -S t3rn-bot
```

```
node main.js
```

## Input
PK pisahin pake koma kalo mau banyak wallet, bot ini dari base ke op kalo ga salah
