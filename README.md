# **Venn Project**

## **Deskripsi**
Proyek ini mengintegrasikan **Venn Firewall** ke dalam **smart contract** di jaringan **Holesky**.

---

## **Instalasi**

### **1. Clone repository**
```sh
git clone https://github.com/OneNov0209/venn-project.git
cd venn-project
```

### **2. Install dependensi**
```sh
npm install
```

### **3. Buat file `.env` dan isi dengan informasi berikut**
```sh
VENN_NODE_URL=https://signer2.testnet.venn.build/api/17000/sign
VENN_POLICY_ADDRESS=0xYOUR_POLICY_ADDRESS
RPC_URL=https://holesky.infura.io/v3/YOUR_INFURA_ID
PRIVATE_KEY=YOUR_PRIVATE_KEY
CONTRACT_ADDRESS=0xYOUR_CONTRACT_ADDRESS
```

---

## **Membuat Struktur Proyek**

Jalankan perintah berikut untuk membuat struktur proyek:
```sh
mkdir contracts scripts
touch contracts/MyContract.sol scripts/deploy.js hardhat.config.js .env index.js
```

---

## **Menulis Smart Contract**

**Buka file `contracts/MyContract.sol` dan isi dengan kode berikut:**
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.19;
import "@ironblocks/firewall-consumer/contracts/consumers/VennFirewallConsumer.sol";

contract MyContract is VennFirewallConsumer {
    constructor(address firewall) {
        _initializeFirewall(firewall); // Pastikan metode ini benar
    }

    function protectedFunction() external firewallProtected {
        // Fungsi ini hanya bisa diakses dengan persetujuan Venn
    }
}
```

---

## **Konfigurasi Hardhat**

**Buka `hardhat.config.js` dan tambahkan konfigurasi berikut:**
```js
require("@nomicfoundation/hardhat-toolbox");
require("dotenv").config();

module.exports = {
  solidity: "0.8.19",
  networks: {
    holesky: {
      url: process.env.RPC_URL,
      accounts: [process.env.PRIVATE_KEY],
    }
  }
};
```

---

## **Membuat Skrip Deploy**

**Buka `scripts/deploy.js` dan tambahkan kode berikut:**
```js
const hre = require("hardhat");

async function main() {
    const [deployer] = await hre.ethers.getSigners();
    console.log("Deploying contract with the account:", deployer.address);

    // Policy Address dari Venn
    const policyAddress = process.env.VENN_POLICY_ADDRESS;

    const MyContract = await hre.ethers.getContractFactory("MyContract");
    const myContract = await MyContract.deploy(policyAddress);

    console.log("Waiting for deployment...");
    await myContract.waitForDeployment();

    console.log("MyContract deployed to:", await myContract.getAddress());
}

main().catch((error) => {
    console.error(error);
    process.exitCode = 1;
});
```

---

## **Deploy Smart Contract**

### **1. Bersihkan & Compile ulang kode smart contract**
```sh
npx hardhat clean
npx hardhat compile
```

Jika muncul error **modifier invocation** saat kompilasi, pastikan Anda menggunakan `_initializeFirewall(firewall);` di dalam kontrak.

### **2. Deploy ke jaringan Holesky**
```sh
npx hardhat run scripts/deploy.js --network holesky
```
**Output jika berhasil:**
```sh
MyContract deployed to: 0xABCDEF123456789...
```
**Tambahkan alamat kontrak ini ke dalam file `.env`:**
```sh
CONTRACT_ADDRESS=0xABCDEF123456789...
```

---

## **Enable Venn Firewall**

Setelah kontrak berhasil didaftarkan, jalankan:
```sh
venn enable
```
**Tambahkan `policyAddress` ke `.env`:**
```sh
VENN_POLICY_ADDRESS=0x123456789ABCDEF...
```

---

## **Menjalankan Transaksi melalui Venn**

**Buka `index.js` dan isi dengan kode berikut:**
```js
require("dotenv").config();
const { VennClient } = require("@vennbuild/venn-dapp-sdk");
const { ethers } = require("ethers");

console.log("ðŸ” Memulai Venn DApp SDK...");

const vennURL = process.env.VENN_NODE_URL;
const vennPolicyAddress = process.env.VENN_POLICY_ADDRESS;
const contractAddress = process.env.CONTRACT_ADDRESS;

console.log("âœ… VENN_NODE_URL:", vennURL);
console.log("âœ… VENN_POLICY_ADDRESS:", vennPolicyAddress);
console.log("âœ… CONTRACT_ADDRESS:", contractAddress);

if (!vennURL || !vennPolicyAddress || !contractAddress) {
    throw new Error("âŒ Environment variables tidak lengkap!");
}

const vennClient = new VennClient({ vennURL, vennPolicyAddress });

async function sendTransaction() {
    console.log("ðŸš€ Mempersiapkan transaksi...");
    const provider = new ethers.JsonRpcProvider(process.env.RPC_URL);
    console.log("âœ… Provider Ethereum berhasil diinisialisasi.");

    const wallet = new ethers.Wallet(process.env.PRIVATE_KEY, provider);
    console.log("âœ… Wallet berhasil dimuat dengan alamat:", wallet.address);

    const contract = new ethers.Contract(contractAddress, [
        "function protectedFunction() external"
    ], wallet);

    console.log("ðŸ”„ Meminta persetujuan transaksi melalui Venn...");
    const txData = contract.interface.encodeFunctionData("protectedFunction");

    const approvedTx = await vennClient.approve({
        from: wallet.address,
        to: contractAddress,
        data: txData,
        value: "0"
    });

    console.log("âœ… Transaksi disetujui oleh Venn, mengirim transaksi...");
    const tx = await wallet.sendTransaction(approvedTx);
    console.log("âœ… Transaksi berhasil dikirim, hash:", tx.hash);
    
    await tx.wait();
    console.log("ðŸŽ‰ Transaksi berhasil dikonfirmasi!");
}

sendTransaction().catch((error) => {
    console.error("âŒ Terjadi kesalahan:", error);
});
```

### **Jalankan transaksi**
```sh
node index.js
```
**Output jika berhasil:**
```sh
✅ Transaksi berhasil dikirim, hash: 0xe186f5fc...
✅ Transaksi berhasil dikonfirmasi!
```

---

## **Menguji Kontrak di Hardhat Console**
Untuk memastikan fungsi `protectedFunction` berjalan dengan baik:

```sh
npx hardhat --network holesky console
```

Kemudian di dalam console Node.js:
```js
const contract = await ethers.getContractAt("MyContract", process.env.CONTRACT_ADDRESS);
await contract.protectedFunction();
```

---

## **Troubleshooting**

Jika terjadi error seperti berikut:

1. **`execution reverted: ApprovedCallsPolicy: call hashes empty`**  
   âœ… Pastikan kontrak telah terhubung dengan Venn Firewall menggunakan `venn enable`.

2. **`Address: low-level delegate call failed`**  
   âœ… Periksa apakah **policy address** telah dikonfigurasi dengan benar di dalam `.env`.

3. **`unknown function`**  
   âœ… Pastikan **ABI** pada `index.js` sesuai dengan definisi smart contract terbaru.

---

## **Author**
- **OneNov0209**
- **ðŸš€ GitHub:** [OneNov0209](https://github.com/OneNov0209)
