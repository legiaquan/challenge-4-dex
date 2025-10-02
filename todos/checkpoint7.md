# 🛰️ Checkpoint 7: Deploy Contracts to Sepolia Testnet

## 📋 Tổng quan

Checkpoint 7 deploy DEX contracts từ local devnet lên **Sepolia Testnet** - một testnet thực tế của Starknet với real network conditions.

## 🎯 Mục tiêu

- Deploy contracts lên Sepolia testnet
- Configure network settings
- Setup wallet & environment
- Get testnet tokens
- Test trên real network

## 🛠️ Step-by-Step Implementation

### **Step 1: Update Network Config** ✅

**File**: `packages/nextjs/scaffold.config.ts`

```typescript
const scaffoldConfig = {
  targetNetworks: [chains.sepolia], // Changed from chains.devnet
  onlyLocalBurnerWallet: false,
  pollingInterval: 30_000,
  autoConnectTTL: 60000,
  walletAutoConnect: true,
} as const satisfies ScaffoldConfig;
```

**Status**: ✅ Completed

### **Step 2: Configure Environment**

#### **Create Environment File**

Create `packages/snfoundry/.env`:

```bash
# Starknet Sepolia Environment Variables
PRIVATE_KEY_SEPOLIA=your_private_key_here
ACCOUNT_ADDRESS_SEPOLIA=your_account_address_here
# Optional: Custom RPC URL
# RPC_URL_SEPOLIA=https://starknet-sepolia.infura.io/v3/YOUR_PROJECT_ID
```

#### **Get Private Key và Account Address**

**From Argent X Wallet**:
1. Open Argent X wallet
2. Go to Settings → Export Private Key
3. Copy private key (without 0x prefix)
4. Copy account address (without 0x prefix)

**From Braavos Wallet**:
1. Open Braavos wallet
2. Go to Settings → Export Private Key  
3. Copy private key (without 0x prefix)
4. Copy account address (without 0x prefix)

**Important**: 
- Use **testnet wallet only** (not mainnet)
- Make sure wallet is on **Sepolia network**
- Private key format: `1234567890abcdef...` (no 0x prefix)
- Account address format: `1234567890abcdef...` (no 0x prefix)

#### **Security Notes** ⚠️
- **NEVER** commit `.env` file to git
- Use testnet wallet only (don't use mainnet)
- Private key should be for testnet account only

### **Step 3: Get STRK Sepolia Tokens**

#### **Faucet Options**:

1. **Starknet Faucet** (Recommended):
   - URL: https://starknet-faucet.vercel.app/
   - Connect wallet (Argent X/Braavos)
   - Request STRK tokens
   - Usually get 0.1 STRK per request

2. **Alchemy Faucet**:
   - URL: https://sepoliafaucet.com/
   - Connect wallet
   - Request tokens

3. **Chainlink Faucet**:
   - URL: https://faucets.chain.link/starknet-sepolia
   - Connect wallet
   - Request tokens

#### **Required Amount**:
- **Minimum**: 0.05 STRK (for deployment)
- **Recommended**: 0.1+ STRK (for testing)

### **Step 4: Deploy Contracts**

#### **Deploy Command**:
```bash
cd packages/snfoundry
yarn deploy --network sepolia
```

#### **Expected Output**:
```
📄 Deploying contracts...
   Compiling contracts v0.2.0 (/path/to/contracts/Scarb.toml)
    Finished `dev` profile target(s) in X seconds
🚀 Deploying Contract  Balloons
✅ Balloons deployed at: 0x...
🚀 Deploying Contract  Dex  
✅ Dex deployed at: 0x...
📦 Found package name in Scarb.toml: contracts
📝 Updated TypeScript contract definition file
✅ Contracts deployed
```

#### **Deployment Files Created**:
- `deployments/sepolia_latest.json` - Contract addresses
- Updated `packages/nextjs/contracts/deployedContracts.ts`

### **Step 5: Update fromBlock for Events**

#### **File**: `packages/nextjs/app/events/page.tsx`

Find the `fromBlock` parameter and update it:

```typescript
// Update fromBlock to deployment block for faster loading
const fromBlock = "latest"; // or specific block number
```

**Why**: Sepolia has many blocks, loading from block 0 is slow.

## 🧪 Testing on Sepolia

### **Test 1: Verify Deployment**

1. **Check deployment file**:
   ```bash
   cat packages/snfoundry/deployments/sepolia_latest.json
   ```

2. **Verify on Starknet Explorer**:
   - Go to: https://sepolia.starkscan.co/
   - Search for contract addresses
   - Verify contracts are deployed

### **Test 2: Frontend Integration**

1. **Start frontend**:
   ```bash
   yarn start
   ```

2. **Connect wallet**:
   - Make sure wallet is on Sepolia network
   - Connect to frontend

3. **Test functionality**:
   - Check token balances
   - Test swap functions
   - Verify events

### **Test 3: Contract Interaction**

1. **Initialize DEX**:
   - Use faucet to get STRK tokens
   - Approve BAL tokens
   - Call init() function

2. **Test swaps**:
   - STRK → BAL swaps
   - BAL → STRK swaps
   - Verify slippage

3. **Test liquidity**:
   - Add liquidity
   - Remove liquidity
   - Verify LP tokens

## 🔍 Verification Checklist

### **Deployment** ✅
- [ ] Network config updated to Sepolia
- [ ] Environment file created with private key
- [ ] STRK tokens obtained from faucet
- [ ] Contracts deployed successfully
- [ ] Deployment addresses saved

### **Contract Verification** ✅
- [ ] Balloons contract deployed
- [ ] DEX contract deployed
- [ ] Contracts visible on Starknet Explorer
- [ ] Contract addresses match deployment file

### **Frontend Integration** ✅
- [ ] Frontend connects to Sepolia
- [ ] Wallet connects successfully
- [ ] Token balances display correctly
- [ ] Swap interface works
- [ ] Events load properly

### **Functionality Testing** ✅
- [ ] DEX initialization works
- [ ] Token swaps functional
- [ ] Liquidity operations work
- [ ] Events emit correctly
- [ ] Slippage calculations accurate

## 🐛 Troubleshooting

### **Issue 1: "Insufficient balance for deployment"**
**Solution**:
- Get more STRK from faucet
- Check wallet has enough for gas fees
- Minimum: 0.05 STRK

### **Issue 2: "Network not supported"**
**Solution**:
- Verify scaffold.config.ts has `chains.sepolia`
- Restart frontend after config change
- Check wallet is on Sepolia network

### **Issue 3: "Contract deployment failed"**
**Solution**:
- Check private key is correct
- Verify RPC URL is working
- Check contract compilation
- Try deploying one contract at a time

### **Issue 4: "Events not loading"**
**Solution**:
- Update fromBlock in events/page.tsx
- Use "latest" or specific block number
- Check network connection

### **Issue 5: "Wallet not connecting"**
**Solution**:
- Ensure wallet is on Sepolia network
- Clear browser cache
- Try different wallet (Argent X/Braavos)

## 📊 Deployment Information

### **Contract Addresses** (Example):
```json
{
  "Balloons": {
    "address": "0x...",
    "classHash": "0x..."
  },
  "Dex": {
    "address": "0x...",
    "classHash": "0x..."
  }
}
```

### **Network Details**:
- **Network**: Sepolia Testnet
- **Chain ID**: 0x534e5f5345504f4c4941
- **RPC URL**: https://starknet-sepolia.public.blastapi.io
- **Explorer**: https://sepolia.starkscan.co/

### **Gas Costs** (Approximate):
- **Balloons deployment**: ~0.001 STRK
- **DEX deployment**: ~0.002 STRK
- **Swap transaction**: ~0.0001 STRK
- **Liquidity operation**: ~0.0002 STRK

## 🚀 Next Steps

After successful deployment:

1. **Test all functionality** on Sepolia
2. **Document contract addresses**
3. **Prepare for Checkpoint 8** (Frontend deployment)
4. **Share with friends** for testing

## ✅ Checklist Checkpoint 7

### **Setup** ✅
- [ ] Network config updated to Sepolia
- [ ] Environment file created
- [ ] Private key configured
- [ ] STRK tokens obtained

### **Deployment** ✅
- [ ] Contracts compiled successfully
- [ ] Balloons contract deployed
- [ ] DEX contract deployed
- [ ] Deployment addresses saved

### **Verification** ✅
- [ ] Contracts visible on explorer
- [ ] Frontend connects to Sepolia
- [ ] Wallet integration works
- [ ] Basic functionality tested

### **Testing** ✅
- [ ] Token balances display
- [ ] Swap functions work
- [ ] Liquidity operations work
- [ ] Events load properly

---

**Status**: ✅ Checkpoint 7 ready for implementation  
**Date**: 2025-10-02  
**Network**: Sepolia Testnet  
**Next**: Checkpoint 8 - Deploy Frontend
