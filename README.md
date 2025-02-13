# Venn Project

## Deskripsi
Proyek ini mengintegrasikan **Venn Firewall** ke dalam **smart contract** di jaringan **Holesky**.

## Instalasi

### 1. Clone repository
```sh
git clone https://github.com/OneNov0209/venn-project.git
cd venn-project
```

### 2. Install dependensi
```sh
npm install
```

### 3. Buat file `.env` dan isi dengan informasi berikut
```ini
VENN_NODE_URL=https://signer2.testnet.venn.build/api/17000/sign
VENN_POLICY_ADDRESS=0xYOUR_POLICY_ADDRESS
RPC_URL=https://holesky.infura.io/v3/YOUR_INFURA_ID
PRIVATE_KEY=YOUR_PRIVATE_KEY
CONTRACT_ADDRESS=0xYOUR_CONTRACT_ADDRESS
```

## Deploy Smart Contract

1. **Bersihkan & Compile ulang kode smart contract**:
   ```sh
   npx hardhat clean
   npx hardhat compile
   ```

2. **Deploy ke jaringan Holesky**:
   ```sh
   npx hardhat run scripts/deploy.js --network holesky
   ```
   Setelah proses selesai, catat **alamat kontrak yang baru dideploy** dan masukkan ke dalam file `.env` pada `CONTRACT_ADDRESS`.

## Enable Venn Firewall
Setelah kontrak berhasil didaftarkan, jalankan:
```sh
venn enable
```

## Menjalankan Transaksi melalui Venn
Untuk mengirim transaksi yang **terproteksi oleh Venn**, jalankan:
```sh
node index.js
```

## Menguji Kontrak di Hardhat Console
Untuk memastikan **fungsi `protectedFunction`** berjalan dengan baik:
```sh
npx hardhat --network holesky console
```

Kemudian di dalam console Node.js:
```js
const contract = await ethers.getContractAt("MyContract", process.env.CONTRACT_ADDRESS);
await contract.protectedFunction();
```

## Troubleshooting
Jika terjadi error seperti:
- **"execution reverted: ApprovedCallsPolicy: call hashes empty"** â†’ Pastikan kontrak telah terhubung dengan Venn Firewall menggunakan `venn enable`.
- **"Address: low-level delegate call failed"** â†’ Periksa apakah policy address telah dikonfigurasi dengan benar di `.env`.
- **"unknown function"** â†’ Pastikan ABI pada `index.js` sesuai dengan smart contract terbaru.

---
**Author**: OneNov0209  
ðŸ“Œ **GitHub**: [OneNov0209](https://github.com/OneNov0209)
