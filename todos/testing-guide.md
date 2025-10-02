# 🧪 Hướng Dẫn Testing Frontend - DEX Challenge

Hướng dẫn chi tiết cách kiểm tra các checkpoints đã implement trên Frontend.

---

## 📋 Tổng quan

| Checkpoint | Có thể check FE? | Cần deploy? | Công cụ check |
|------------|------------------|-------------|---------------|
| **Checkpoint 1** (Balloons mint) | ✅ Có | ✅ Cần | Debug Contracts |
| **Checkpoint 2** (init) | ✅ Có | ✅ Cần | Debug Contracts, DEX Tab |
| **Checkpoint 3** (price) | ✅ Có | ✅ Cần | Debug Contracts |
| **Checkpoint 4** (trading) | ✅ Có | ✅ Cần | Debug Contracts, DEX Tab |
| **Checkpoint 5** (liquidity) | ✅ Có | ✅ Cần | Debug Contracts, DEX Tab |

---

## 🚀 Bước 0: Setup và Deploy

### Prerequisites
- Devnet đang chạy
- Contracts đã deploy
- Frontend đang chạy

### Commands

```bash
# Terminal 1 - Start Devnet
make start-devnet
# hoặc
yarn chain

# Terminal 2 - Deploy Contracts
make deploy
# hoặc
yarn deploy

# Terminal 3 - Start Frontend
make start
# hoặc
yarn start
```

### Lấy thông tin cần thiết

Sau khi deploy, ghi lại các địa chỉ từ console log hoặc file deployment:

```bash
# Xem file deployment
cat packages/snfoundry/deployments/devnet_latest.json
```

**Thông tin cần:**
- ✅ **Deployer Address**: `0x64b48806902a367c8598f4f95c305e8c1a1acba5f082d294a43793113115691` (default devnet)
- ✅ **DEX Address**: Xem trong console log hoặc deployment file
- ✅ **Balloons Address**: Xem trong console log hoặc deployment file
- ✅ **STRK Address**: `0x4718F5A0FC34CC1AF16A1CDEE98FFB20C31F5CD61D6AB07201858F4287C938D`

**Frontend URL**: http://localhost:3000

---

## ✅ Checkpoint 1: Balloons Token Mint

### Mục tiêu
- Verify deployer nhận 1000 $BAL tokens
- Verify frontend wallet nhận 10 $BAL (bonus)

### Cách check trên Debug Contracts

#### Test 1: Deployer balance
1. Mở http://localhost:3000
2. Vào tab **"Debug Contracts"** (navigation bar)
3. Chọn contract **"Balloons"**
4. Tìm function **`balance_of`** (trong Read Functions section)
5. Input parameters:
   ```
   account: 0x64b48806902a367c8598f4f95c305e8c1a1acba5f082d294a43793113115691
   ```
6. Click button **"Read"**
7. **Kỳ vọng**: 
   ```
   Output: 1000000000000000000000
   (= 1000 * 10^18 = 1000 tokens với 18 decimals)
   ```

#### Test 2: Frontend wallet balance
1. Connect wallet (Argent X hoặc Braavos)
2. Copy wallet address của bạn
3. Debug Contracts → Balloons → `balance_of`
4. Input: địa chỉ wallet của bạn
5. Click **"Read"**
6. **Kỳ vọng**:
   ```
   Output: 10000000000000000000
   (= 10 * 10^18 = 10 tokens)
   ```

#### Test 3: Total supply
1. Debug Contracts → Balloons → `total_supply`
2. Click **"Read"** (không cần input)
3. **Kỳ vọng**:
   ```
   Output: 1000000000000000000000
   (= 1000 tokens total supply)
   ```

### Checklist Checkpoint 1
- [ ] Deployer balance = 1000 * 10^18
- [ ] Frontend wallet balance = 10 * 10^18 (nếu đã uncomment transfer)
- [ ] Total supply = 1000 * 10^18
- [ ] Token name = "Balloons"
- [ ] Token symbol = "BAL"

---

## ✅ Checkpoint 2: Init Function (Reserves & Liquidity)

### Mục tiêu
- Verify DEX đã được khởi tạo với reserves
- Verify liquidity được track đúng

### Cách check trên Debug Contracts

#### Test 1: Total Liquidity
1. Debug Contracts → Chọn **"Dex"**
2. Tìm function **`get_total_liquidity()`**
3. Click **"Read"**
4. **Kỳ vọng**:
   ```
   Output: 5000000000000000000
   (= 5 * 10^18 = 5 LP tokens)
   ```

#### Test 2: Deployer Liquidity
1. Debug Contracts → Dex → `get_liquidity`
2. Input parameters:
   ```
   lp_address: 0x64b48806902a367c8598f4f95c305e8c1a1acba5f082d294a43793113115691
   ```
3. Click **"Read"**
4. **Kỳ vọng**:
   ```
   Output: 5000000000000000000
   (= 5 LP tokens, deployer owns 100% of liquidity)
   ```

#### Test 3: DEX BAL Balance
1. Debug Contracts → Balloons → `balance_of`
2. Input: DEX contract address (từ deployment)
3. Click **"Read"**
4. **Kỳ vọng**:
   ```
   Output: 5000000000000000000
   (= 5 BAL tokens in reserve)
   ```

#### Test 4: DEX STRK Balance
1. Debug Contracts → Chọn contract **"STRK"** hoặc **"MockSTRKToken"**
2. Function: `balance_of`
3. Input: DEX contract address
4. Click **"Read"**
5. **Kỳ vọng**:
   ```
   Output: 5000000000000000000
   (= 5 STRK tokens in reserve)
   ```

### Cách check trên DEX Tab (nếu có UI)

1. Vào tab **"DEX"**
2. Xem section "Pool Information" hoặc "Liquidity"
3. **Kỳ vọng**:
   ```
   BAL Reserve: 5.0
   STRK Reserve: 5.0
   Total Liquidity: 5.0
   Your Liquidity: 5.0 (nếu dùng deployer account)
   Your Liquidity: 0.0 (nếu dùng wallet khác)
   ```

### Checklist Checkpoint 2
- [ ] Total liquidity = 5 * 10^18
- [ ] Deployer liquidity = 5 * 10^18
- [ ] DEX BAL balance = 5 * 10^18
- [ ] DEX STRK balance = 5 * 10^18
- [ ] Ratio = 1:1 (5 BAL : 5 STRK)

---

## ✅ Checkpoint 3: Price Function

### Mục tiêu
- Verify function `price()` tính đúng giá swap
- Verify fee 0.3% được apply
- Verify slippage với different scenarios

### Cách check trên Debug Contracts

Function signature:
```cairo
fn price(x_input: u256, x_reserves: u256, y_reserves: u256) -> u256
```

#### Test Case 1: Small swap, balanced ratio (1:1)
**Scenario**: Swap 1 token trong pool 1000:1000

1. Debug Contracts → Dex → `price`
2. Input parameters:
   ```
   x_input: 1000000000000000000        (1 token)
   x_reserves: 1000000000000000000000  (1000 tokens)
   y_reserves: 1000000000000000000000  (1000 tokens)
   ```
3. Click **"Read"**
4. **Kỳ vọng**:
   ```
   Output: ~996000000000000000 (0.996 tokens)
   
   Giải thích:
   - Input: 1 token
   - Fee 0.3% → keep 99.7%
   - Expected: 0.997, nhưng có slippage nhỏ
   - Actual: 0.996 (fee + slippage)
   ```

#### Test Case 2: Small swap, unbalanced ratio (5:1)
**Scenario**: Swap 1 BAL trong pool có nhiều STRK hơn

1. Debug Contracts → Dex → `price`
2. Input parameters:
   ```
   x_input: 1000000000000000000        (1 BAL)
   x_reserves: 1000000000000000000000  (1000 BAL)
   y_reserves: 5000000000000000000000  (5000 STRK)
   ```
3. Click **"Read"**
4. **Kỳ vọng**:
   ```
   Output: ~4980000000000000000 (4.98 STRK)
   
   Giải thích:
   - Ratio 5:1 → mong đợi ~5 STRK
   - Fee 0.3% → 4.985 STRK
   - Slippage nhỏ → 4.98 STRK
   ```

#### Test Case 3: Large swap, high slippage
**Scenario**: Swap 100 BAL (10% of reserve)

1. Debug Contracts → Dex → `price`
2. Input parameters:
   ```
   x_input: 100000000000000000000       (100 BAL)
   x_reserves: 1000000000000000000000   (1000 BAL)
   y_reserves: 5000000000000000000000   (5000 STRK)
   ```
3. Click **"Read"**
4. **Kỳ vọng**:
   ```
   Output: ~453305000000000000000 (453.3 STRK)
   
   Giải thích:
   - Ratio 5:1 → mong đợi 500 STRK
   - Nhưng swap quá lớn (10% reserve)
   - High slippage: chỉ nhận ~453 STRK (90.6%)
   ```

### Manual Calculation (Optional)

Verify bằng công thức:
```
xInputWithFee = xInput * 997
numerator = yReserves * xInputWithFee
denominator = (xReserves * 1000) + xInputWithFee
yOutput = numerator / denominator
```

**Example**: Test case 1
```
xInputWithFee = 1 * 10^18 * 997 = 997 * 10^18
numerator = 1000 * 10^18 * 997 * 10^18 = 997 * 10^39
denominator = (1000 * 10^18 * 1000) + (997 * 10^18) 
            = 1,000,997 * 10^18
yOutput = (997 * 10^39) / (1,000,997 * 10^18)
        = 996,003... * 10^15
        ≈ 0.996 * 10^18
```

### Checklist Checkpoint 3
- [ ] Test case 1: Small swap, balanced → ~0.996 output
- [ ] Test case 2: Unbalanced ratio → price theo ratio
- [ ] Test case 3: Large swap → high slippage
- [ ] Fee 0.3% được apply đúng
- [ ] Function không bị revert với valid inputs

---

## 🧪 Checkpoint 4: Trading Functions

### Mục tiêu
- Verify functions `strk_to_token()` và `token_to_strk()` hoạt động đúng
- Verify reserves và balances được update sau mỗi swap
- Verify events được emit
- Verify slippage và fees được tính đúng

### Prerequisites
- ✅ Checkpoint 3 hoàn thành (price function working)
- ✅ DEX đã được init với reserves (5 BAL : 5 STRK)
- ✅ User có đủ BAL tokens để swap (approve trước)
- ✅ User có đủ STRK để swap và trả gas

### Functions cần test:
- `strk_to_token(strk_input: u256) -> u256` - Swap STRK → BAL
- `token_to_strk(token_input: u256) -> u256` - Swap BAL → STRK

### 🎯 Frontend UI Testing (Recommended)

**DEX Tab Features**:
- **STRK to Token Swap**: Input STRK amount → get BAL tokens
- **Token to STRK Swap**: Input BAL amount → get STRK tokens  
- **Expected Output Display**: Shows predicted output với slippage
- **Reserve Display**: Shows current BAL/STRK reserves
- **Balance Display**: Shows your current token balances
- **Auto-approve**: Frontend tự động handle approve + swap trong 1 transaction

**Advantages của Frontend UI**:
- ✅ User-friendly interface
- ✅ Real-time balance updates
- ✅ Expected output preview
- ✅ Slippage warnings
- ✅ Auto-approve mechanism
- ✅ Visual feedback
- ✅ Error handling với user-friendly messages

**Navigation**:
- Main page: http://localhost:3000
- DEX tab: http://localhost:3000/dex
- Events tab: http://localhost:3000/events
- Debug Contracts: http://localhost:3000/debug

### 📋 Step-by-Step Frontend Testing Guide

#### **Setup Phase**

1. **Start all services**:
   ```bash
   # Terminal 1 - Start devnet
   make start-devnet
   
   # Terminal 2 - Deploy contracts  
   make deploy
   
   # Terminal 3 - Start frontend
   make start
   ```

2. **Open browser**: http://localhost:3000

3. **Connect wallet**: Click "Connect Wallet" → chọn Argent X hoặc Braavos

4. **Get STRK tokens** (nếu cần): Click icon 💰 (faucet button) để get 1 STRK

#### **Test Phase 1: STRK → BAL Swap**

1. **Navigate to DEX tab**:
   - Click tab **"DEX"** trên navigation bar
   - URL: http://localhost:3000/dex

2. **Locate "STRK to Token Swap" section**:
   - Input field: "Amount to swap"
   - Nhập: `1` (STRK)
   - Button: "Swap STRK to Token"

3. **Execute swap**:
   - Click button "Swap STRK to Token"
   - Wallet popup → Click "Approve" (auto-approve + swap)
   - Wait for transaction confirmation

4. **Verify results**:
   - ✅ Check "Expected Output" hiển thị ~0.996 BAL
   - ✅ Check reserves updated trong UI
   - ✅ Check your BAL balance increased

#### **Test Phase 2: BAL → STRK Swap**

1. **Locate "Token to STRK Swap" section**:
   - Input field: "Amount to swap" 
   - Nhập: `1` (BAL)
   - Button: "Swap Token to STRK"

2. **Execute swap**:
   - Click button "Swap Token to STRK"
   - Wallet popup → Click "Approve" (auto-approve + swap)
   - Wait for transaction confirmation

3. **Verify results**:
   - ✅ Check "Expected Output" hiển thị ~0.996 STRK
   - ✅ Check reserves updated trong UI
   - ✅ Check your STRK balance increased

#### **Test Phase 3: Large Swap (Slippage Test)**

1. **STRK to Token Swap với large amount**:
   - Input amount: `2` (STRK)
   - Check "Expected Output" - should show high slippage warning
   - Click "Swap STRK to Token"
   - Approve transaction

2. **Verify high slippage**:
   - ✅ Expected output: ~1.67 BAL (thay vì 2 BAL)
   - ✅ Slippage: ~16.5% (do swap quá lớn so với reserve)

#### **Test Phase 4: Events Verification**

1. **Navigate to Events tab**:
   - Click tab **"Events"** trên navigation bar
   - URL: http://localhost:3000/events

2. **Filter and check events**:
   - Filter by contract: **Dex**
   - Look for recent swap events:
     - ✅ `StrkToTokenSwap` event
     - ✅ `TokenToStrkSwap` event
   - Verify event parameters match transaction details

#### **Expected Results Summary**

```
BEFORE SWAP:
- DEX BAL reserve: 5.0
- DEX STRK reserve: 5.0
- Your BAL: 0
- Your STRK: X

AFTER SWAP (1 STRK → ~0.996 BAL):
- DEX BAL reserve: ~4.004
- DEX STRK reserve: ~6.0
- Your BAL: ~0.996
- Your STRK: X - 1 - gas
```

#### **Frontend UI Features to Verify**

- ✅ **Reserve Display**: Shows current BAL/STRK reserves
- ✅ **Balance Display**: Shows your current token balances  
- ✅ **Expected Output**: Shows predicted output với slippage
- ✅ **Slippage Warning**: For large swaps
- ✅ **Auto-approve**: Handles approve + swap trong 1 transaction
- ✅ **Real-time Updates**: Balances và reserves update sau mỗi swap

#### **Troubleshooting Frontend Issues**

**Issue 1: "Insufficient balance"**
- Solution: Click faucet button 💰 để get STRK
- Check bạn có đủ tokens để swap

**Issue 2: "Transaction failed"**
- Solution: 
  - Check devnet đang chạy: `curl http://localhost:5050/is_alive`
  - Redeploy contracts: `make deploy`
  - Restart frontend: `Ctrl+C` → `make start`

**Issue 3: "Expected output = 0"**
- Solution:
  - Check Checkpoint 3 (price function) đã implement chưa
  - Check DEX đã được init chưa (Checkpoint 2)

**Issue 4: "Contract not found"**
- Solution:
  - Check deployment: `cat packages/snfoundry/deployments/devnet_latest.json`
  - Redeploy: `make deploy`

**Issue 5: "Events not showing"**
- Solution:
  - Verify events are added to enum Event trong dex.cairo
  - Check từ Checkpoint 4 trở đi mới có events

---

### Test Case 1: STRK → BAL Swap (Small Amount)

**Scenario**: Swap 1 STRK để lấy BAL tokens

#### Method A: Frontend UI (Recommended) 🎯

1. **Connect Wallet & Navigate**:
   - Mở http://localhost:3000
   - Connect wallet (Argent X hoặc Braavos)
   - Navigate to **"DEX"** tab trong navigation bar

2. **Check Prerequisites**:
   - Verify bạn có đủ STRK balance (dùng faucet nếu cần)
   - Check DEX reserves hiển thị: ~5 BAL : ~5 STRK

3. **Execute STRK → BAL Swap**:
   - Tìm section "STRK to Token Swap"
   - Input amount: `1` (STRK)
   - Click button **"Swap STRK to Token"**
   - Approve transaction trong wallet (sẽ approve STRK và execute swap)
   - Wait for transaction confirmation

4. **Verify Results**:
   - Check "Expected Output" hiển thị ~0.996 BAL
   - Verify reserves updated trong UI
   - Check your BAL balance increased

#### Method B: Debug Contracts (Alternative)

1. **Prepare**:
   - Connect wallet với đủ STRK balance
   - Get STRK balance trước khi swap:
     - Debug Contracts → STRK → `balance_of`
     - Input: wallet address của bạn

2. **Execute STRK → BAL Swap**:
   - Debug Contracts → Dex → `strk_to_token`
   - Input parameters:
     ```
     strk_input: 1000000000000000000
     (= 1 STRK = 1 * 10^18)
     ```
   - Click **"Write"** (không phải Read)
   - Approve transaction trong wallet
   - Wait for transaction confirmation

#### Step 3: Verify Results
1. **Check BAL output** (từ transaction receipt):
   - Expected: ~996000000000000000 (0.996 BAL)
   - Based on price function với reserves 5:5

2. **Check DEX reserves changed**:
   - Debug Contracts → Dex → `get_total_liquidity` (should still be 5)
   - Debug Contracts → Balloons → `balance_of` (DEX address)
   - Debug Contracts → STRK → `balance_of` (DEX address)

3. **Check user balances**:
   - Debug Contracts → Balloons → `balance_of` (your wallet)
   - Debug Contracts → STRK → `balance_of` (your wallet)

#### Expected Results:
```
BEFORE SWAP:
- DEX BAL reserve: 5000000000000000000 (5 BAL)
- DEX STRK reserve: 5000000000000000000 (5 STRK)
- Your BAL: 0 (or previous amount)
- Your STRK: X amount

AFTER SWAP (1 STRK → ~0.996 BAL):
- DEX BAL reserve: ~4004000000000000000 (4.004 BAL)
- DEX STRK reserve: ~6000000000000000000 (6 STRK)
- Your BAL: ~996000000000000000 (0.996 BAL)
- Your STRK: X - 1000000000000000000 - gas
```

---

### Test Case 2: BAL → STRK Swap (Small Amount)

**Scenario**: Swap 1 BAL để lấy STRK tokens

#### Method A: Frontend UI (Recommended) 🎯

1. **Connect Wallet & Navigate**:
   - Mở http://localhost:3000
   - Connect wallet (Argent X hoặc Braavos)
   - Navigate to **"DEX"** tab

2. **Check Prerequisites**:
   - Verify bạn có BAL tokens (nếu chưa có, swap STRK→BAL trước)
   - Check DEX reserves hiển thị

3. **Execute BAL → STRK Swap**:
   - Tìm section "Token to STRK Swap"
   - Input amount: `1` (BAL)
   - Click button **"Swap Token to STRK"**
   - Approve transaction trong wallet (sẽ approve BAL và execute swap)
   - Wait for transaction confirmation

4. **Verify Results**:
   - Check "Expected Output" hiển thị ~0.996 STRK
   - Verify reserves updated trong UI
   - Check your STRK balance increased

#### Method B: Debug Contracts (Alternative)

1. **Approve BAL (Required!)**:
   - Debug Contracts → Balloons → `approve`
   - Input parameters:
     ```
     spender: [DEX contract address]
     amount: 1000000000000000000
     (= 1 BAL)
     ```
   - Click **"Write"** và approve trong wallet

2. **Execute BAL → STRK Swap**:
   - Debug Contracts → Dex → `token_to_strk`
   - Input parameters:
     ```
     token_input: 1000000000000000000
     (= 1 BAL)
     ```
   - Click **"Write"** và approve transaction

3. **Verify Results**:
   - **Check STRK output**: Expected ~996000000000000000 (0.996 STRK)
   - **Check reserves updated**:
     - DEX BAL reserve: ~6000000000000000000 (6 BAL)
     - DEX STRK reserve: ~4004000000000000000 (4.004 STRK)

---

### Test Case 3: Large Swap (High Slippage)

**Scenario**: Swap 2 STRK (40% of reserve) để test slippage

#### Method A: Frontend UI (Recommended) 🎯

1. **Navigate to DEX Tab**:
   - Mở http://localhost:3000 → DEX tab
   - Connect wallet

2. **Execute Large Swap**:
   - Tìm section "STRK to Token Swap"
   - Input amount: `2` (STRK)
   - Check "Expected Output" - should show high slippage warning
   - Click **"Swap STRK to Token"**
   - Approve transaction trong wallet

3. **Verify High Slippage**:
   - Expected output: ~1.67 BAL (thay vì 2 BAL)
   - Slippage: ~16.5% (do swap quá lớn so với reserve)
   - Check reserves updated trong UI

#### Method B: Debug Contracts (Alternative)

1. **Execute Large Swap**:
   - Debug Contracts → Dex → `strk_to_token`
   - Input parameters:
     ```
     strk_input: 2000000000000000000
     (= 2 STRK)
     ```
   - Click **"Write"** và approve

2. **Expected Results**:
   ```
   Expected output: ~1.67 BAL (thay vì 2 BAL)
   Slippage: ~16.5% (do swap quá lớn so với reserve)
   New reserves:
   - DEX BAL: ~3.33 BAL
   - DEX STRK: ~7 STRK
   ```

---

### Test Case 4: Edge Cases

#### Test 4.1: Swap with insufficient approval
1. Try `token_to_strk` với amount > approved amount
2. **Expected**: Transaction should fail/revert

#### Test 4.2: Swap with insufficient balance
1. Try swap với amount > wallet balance
2. **Expected**: Transaction should fail/revert

#### Test 4.3: Swap zero amount
1. Try `strk_to_token(0)`
2. **Expected**: Should return 0 hoặc revert

---

### Test Case 5: Events Verification

#### Method A: Frontend Events Tab (Recommended) 🎯

1. **Navigate to Events**:
   - Sau khi swap thành công, go to `/events` page
   - Filter by contract: **Dex**
   - Look for recent swap events

2. **Expected Events**:
   ```
   StrkToTokenSwap {
     swapper: [your wallet address],
     token_output: [amount received],
     strk_input: [amount sent]
   }
   
   TokenToStrkSwap {
     swapper: [your wallet address],
     tokens_input: [amount sent],
     strk_output: [amount received]
   }
   ```

#### Method B: Transaction Receipt (Alternative)

1. **Check Transaction Receipt**:
   - Sau khi swap thành công, check transaction receipt
   - Look for emitted events trong receipt

2. **Expected events** (same as above)

---

### Manual Calculation Verification

Verify swap results bằng công thức AMM:

**For STRK → BAL (1 STRK):**
```
x_input = 1 * 10^18 (STRK)
x_reserves = 5 * 10^18 (STRK reserves)
y_reserves = 5 * 10^18 (BAL reserves)

xInputWithFee = 1 * 10^18 * 997 = 997 * 10^18
numerator = 5 * 10^18 * 997 * 10^18 = 4985 * 10^36
denominator = (5 * 10^18 * 1000) + (997 * 10^18) = 5997 * 10^18
yOutput = numerator / denominator ≈ 0.996 * 10^18
```

---

### Checklist Checkpoint 4

#### Basic Functionality ✅
- [ ] `strk_to_token()` works với small amount
- [ ] `token_to_strk()` works với small amount  
- [ ] Approve mechanism works correctly
- [ ] Functions return expected amounts

#### Reserve Updates ✅
- [ ] DEX BAL reserve updates after STRK→BAL swap
- [ ] DEX STRK reserve updates after BAL→STRK swap
- [ ] Total liquidity remains constant (5 LP tokens)
- [ ] Reserves maintain AMM relationship

#### User Balances ✅
- [ ] User BAL balance increases after STRK→BAL
- [ ] User STRK balance increases after BAL→STRK
- [ ] Gas fees deducted correctly
- [ ] Approval amounts updated correctly

#### Edge Cases ✅
- [ ] Insufficient approval → transaction fails
- [ ] Insufficient balance → transaction fails
- [ ] Zero amount swap → returns 0 or reverts
- [ ] Large swap → high slippage observed

#### Events ✅
- [ ] `StrkToTokenSwap` event emitted correctly
- [ ] `TokenToStrkSwap` event emitted correctly
- [ ] Event parameters match transaction details
- [ ] Events visible in `/events` page

#### Price Consistency ✅
- [ ] Swap output matches `price()` function calculation
- [ ] Fee 0.3% applied correctly
- [ ] Slippage increases with larger swaps
- [ ] No reverts with valid inputs

---

### Troubleshooting Checkpoint 4

#### Issue 1: "Insufficient allowance"
**Solution**: Call `approve()` on Balloons contract first

#### Issue 2: "Insufficient balance"
**Solution**: 
- Check STRK balance for STRK→BAL swaps
- Check BAL balance for BAL→STRK swaps
- Use faucet if needed

#### Issue 3: "Transaction reverted"
**Solution**:
- Check function implementation in dex.cairo
- Verify events are added to enum Event
- Check reserves are initialized

#### Issue 4: "Wrong output amount"
**Solution**:
- Verify price function implementation
- Check reserves are correct (5:5 ratio)
- Verify fee calculation (997/1000)

---

### Quick Test Commands

```bash
# Get current reserves
curl -X POST http://localhost:5050/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "starknet_call",
    "params": {
      "request": {
        "contract_address": "[DEX_ADDRESS]",
        "entry_point_selector": "get_total_liquidity",
        "calldata": []
      },
      "block_id": "latest"
    },
    "id": 1
  }'
```

---

## 💧 Checkpoint 5: Liquidity Functions

### Mục tiêu
- Verify functions `deposit()` và `withdraw()` hoạt động đúng
- Verify LP token calculations
- Verify multiple liquidity providers
- Verify events được emit
- Verify proportional withdrawals

### Prerequisites
- ✅ Checkpoint 4 hoàn thành (trading functions working)
- ✅ User có đủ STRK và BAL tokens để deposit
- ✅ User đã approve BAL tokens cho DEX contract

### Functions cần test:
- `deposit(strk_amount: u256) -> u256` - Add liquidity và mint LP tokens
- `withdraw(amount: u256) -> (u256, u256)` - Remove liquidity và burn LP tokens
- `get_deposit_token_amount(strk_amount: u256) -> u256` - Calculate token amount needed
- `get_liquidity(lp_address: ContractAddress) -> u256` - Get user's LP tokens
- `get_total_liquidity() -> u256` - Get total pool liquidity

### 🎯 Frontend UI Testing (Recommended)

**DEX Tab Features**:
- **Deposit Section**: Input STRK amount → calculate token amount needed
- **Withdraw Section**: Input LP amount → calculate STRK + BAL output
- **LP Balance Display**: Show user's current LP tokens
- **Total Liquidity Display**: Show pool's total liquidity

**Advantages của Frontend UI**:
- ✅ User-friendly interface
- ✅ Real-time balance updates
- ✅ Expected output preview
- ✅ Auto-approve mechanism
- ✅ Visual feedback
- ✅ LP token calculations displayed

---

### Test Case 1: First Deposit

**Scenario**: User đầu tiên deposit liquidity vào empty pool

#### Method A: Frontend UI (Recommended) 🎯

1. **Navigate to DEX Tab**:
   - Mở http://localhost:3000/dex
   - Connect wallet

2. **Prepare for Deposit**:
   - Get STRK tokens (faucet nếu cần)
   - Get BAL tokens (swap STRK→BAL nếu cần)
   - Approve BAL tokens cho DEX contract

3. **Execute First Deposit**:
   - Tìm section "Add Liquidity" hoặc "Deposit"
   - Input STRK amount: `1`
   - Check required BAL amount (should be 1 BAL)
   - Click "Deposit" button
   - Approve transactions trong wallet

4. **Verify Results**:
   - ✅ LP tokens received = 1
   - ✅ Total liquidity = 1
   - ✅ User liquidity = 1
   - ✅ Reserves = 6 STRK : 6 BAL (5 + 1)

#### Method B: Debug Contracts (Alternative)

1. **Approve BAL tokens**:
   - Debug Contracts → Balloons → `approve`
   - Input: `spender: [DEX address]`, `amount: 1000000000000000000` (1 BAL)

2. **Execute Deposit**:
   - Debug Contracts → Dex → `deposit`
   - Input: `strk_amount: 1000000000000000000` (1 STRK)
   - Click "Write" và approve

3. **Verify Results**:
   - Check `get_liquidity` với user address
   - Check `get_total_liquidity`
   - Check reserves updated

---

### Test Case 2: Subsequent Deposit

**Scenario**: User thứ 2 deposit liquidity vào pool đã có liquidity

#### Method A: Frontend UI (Recommended) 🎯

1. **Use Different Wallet**:
   - Connect wallet khác (hoặc tạo new account)
   - Get STRK và BAL tokens
   - Approve BAL tokens

2. **Execute Subsequent Deposit**:
   - Input STRK amount: `1`
   - Check required BAL amount (should be 1 BAL)
   - Click "Deposit" button
   - Approve transactions

3. **Verify Results**:
   - ✅ LP tokens received < 1 (proportional)
   - ✅ Total liquidity increased
   - ✅ User liquidity = LP tokens received
   - ✅ Reserves = 7 STRK : 7 BAL

#### Expected LP Calculation:
```
LP = (strk_amount * total_liquidity) / strk_reserves
LP = (1 * 1) / 6 = 0.167 LP tokens
```

---

### Test Case 3: Partial Withdraw

**Scenario**: User rút một phần LP tokens

#### Method A: Frontend UI (Recommended) 🎯

1. **Navigate to Withdraw Section**:
   - Tìm section "Remove Liquidity" hoặc "Withdraw"
   - Input LP amount: `0.5` (hoặc một phần LP tokens)

2. **Execute Withdraw**:
   - Check expected STRK + BAL output
   - Click "Withdraw" button
   - Approve transaction

3. **Verify Results**:
   - ✅ User received proportional STRK + BAL
   - ✅ LP tokens decreased
   - ✅ Total liquidity decreased
   - ✅ Reserves decreased proportionally

#### Method B: Debug Contracts (Alternative)

1. **Execute Withdraw**:
   - Debug Contracts → Dex → `withdraw`
   - Input: `amount: 500000000000000000` (0.5 LP tokens)
   - Click "Write" và approve

2. **Verify Results**:
   - Check returned values: (strk_amount, token_amount)
   - Check user liquidity decreased
   - Check total liquidity decreased

---

### Test Case 4: Full Withdraw

**Scenario**: User rút tất cả LP tokens

#### Execute Full Withdraw

1. **Input All LP Tokens**:
   - Input amount = user's current LP balance
   - Check expected output

2. **Verify Results**:
   - ✅ User received all deposited tokens back
   - ✅ User liquidity = 0
   - ✅ Total liquidity decreased
   - ✅ Reserves decreased by user's contribution

---

### Test Case 5: Multiple Users

**Scenario**: Test với multiple liquidity providers

#### Setup Multiple Users

1. **User A Deposit**:
   - Connect wallet A
   - Deposit 2 STRK + 2 BAL
   - Note: LP tokens received

2. **User B Deposit**:
   - Connect wallet B
   - Deposit 1 STRK + 1 BAL
   - Note: LP tokens received (should be less than User A)

3. **Verify Independence**:
   - User A withdraw một phần
   - User B withdraw một phần
   - Verify không ảnh hưởng lẫn nhau

---

### Test Case 6: Edge Cases

#### Test 6.1: Insufficient Liquidity
1. Try withdraw với amount > user's LP balance
2. **Expected**: Transaction should fail/revert

#### Test 6.2: Zero Amount Withdraw
1. Try withdraw với amount = 0
2. **Expected**: Should return (0, 0) hoặc revert

#### Test 6.3: Large Deposit
1. Deposit với amount lớn (e.g., 10 STRK)
2. **Expected**: LP tokens calculated correctly
3. **Verify**: Reserves updated correctly

---

### Test Case 7: Events Verification

#### Check Liquidity Events

1. **Navigate to Events Tab**:
   - Go to http://localhost:3000/events
   - Filter by contract: **Dex**

2. **Expected Events**:
   ```
   LiquidityProvided {
     liquidity_provider: [user address],
     liquidity_minted: [LP tokens minted],
     strk_input: [STRK deposited],
     tokens_input: [BAL deposited]
   }
   
   LiquidityRemoved {
     liquidity_remover: [user address],
     liquidity_withdrawn: [LP tokens burned],
     tokens_output: [BAL received],
     strk_output: [STRK received]
   }
   ```

---

### Checklist Checkpoint 5

#### Basic Functionality ✅
- [ ] `deposit()` works với small amount
- [ ] `withdraw()` works với small amount
- [ ] `get_deposit_token_amount()` calculates correctly
- [ ] `get_liquidity()` returns user's LP tokens
- [ ] `get_total_liquidity()` returns total pool liquidity

#### LP Token Calculations ✅
- [ ] First deposit mints correct LP tokens
- [ ] Subsequent deposits mint proportional LP tokens
- [ ] Withdraw calculations are proportional
- [ ] Total liquidity updates correctly

#### Multiple Users ✅
- [ ] Multiple users can deposit independently
- [ ] Each user gets proportional LP tokens
- [ ] Users can withdraw independently
- [ ] No interference between users

#### Edge Cases ✅
- [ ] Insufficient liquidity → transaction fails
- [ ] Zero amount withdraw → returns (0, 0) or reverts
- [ ] Large deposits work correctly
- [ ] Full withdraw works correctly

#### Events ✅
- [ ] `LiquidityProvided` event emitted correctly
- [ ] `LiquidityRemoved` event emitted correctly
- [ ] Event parameters match transaction details
- [ ] Events visible in `/events` page

#### Storage Updates ✅
- [ ] `total_liquidity` updates correctly
- [ ] `liquidity[address]` updates correctly
- [ ] Reserves update correctly
- [ ] User balances update correctly

---

### Troubleshooting Checkpoint 5

#### Issue 1: "Insufficient allowance"
**Solution**: Call `approve()` on Balloons contract first

#### Issue 2: "Insufficient liquidity"
**Solution**: Check user has enough LP tokens to withdraw

#### Issue 3: "Transaction reverted"
**Solution**:
- Check function implementation in dex.cairo
- Verify events are added to enum Event
- Check reserves are sufficient

#### Issue 4: "Wrong LP token amount"
**Solution**:
- Verify LP calculation logic
- Check total_liquidity is correct
- Verify reserves are correct

#### Issue 5: "Events not showing"
**Solution**:
- Verify events are added to enum Event trong dex.cairo
- Check Events tab filters correctly

---

### Quick Test Commands

```bash
# Check current liquidity
curl -X POST http://localhost:5050/rpc \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "method": "starknet_call",
    "params": {
      "request": {
        "contract_address": "[DEX_ADDRESS]",
        "entry_point_selector": "get_total_liquidity",
        "calldata": []
      },
      "block_id": "latest"
    },
    "id": 1
  }'
```

---

## 🐛 Common Issues & Troubleshooting

### Issue 1: Contract not found
**Triệu chứng**: "Contract Dex not found in ABI"

**Giải pháp**:
1. Check đã deploy chưa: `ls packages/snfoundry/deployments/`
2. Redeploy: `make deploy`
3. Restart frontend: `Ctrl+C` → `make start`

### Issue 2: Function returns 0
**Triệu chứng**: All functions return 0

**Nguyên nhân**: Function chưa được implement hoặc không được gọi

**Giải pháp**:
1. Check implementation trong dex.cairo
2. Verify đã uncomment code trong deploy.ts
3. Redeploy contracts

### Issue 3: Balance không đúng
**Triệu chứng**: Balance = 0 hoặc khác kỳ vọng

**Giải pháp**:
1. Check đã approve chưa (cho Checkpoint 2)
2. Check transferScript có chạy không
3. Verify deployer address đúng
4. Redeploy: `make deploy`

### Issue 4: Event not found (Checkpoint 4+)
**Triệu chứng**: "Event StrkToTokenSwap not found"

**Nguyên nhân**: Events chưa được add vào enum Event

**Giải pháp**:
1. Tránh vào page `/events` cho đến khi hoàn thành Checkpoint 4
2. Hoặc implement Checkpoint 4 để add events

### Issue 5: Transaction fails
**Triệu chứng**: Transaction rejected hoặc fail

**Giải pháp**:
1. Check devnet đang chạy
2. Check wallet có đủ STRK để trả gas
3. Check đã approve token chưa (nếu cần)
4. Restart devnet: `Ctrl+C` → `make start-devnet` → `make deploy`

---

## 📊 Testing Checklist Tổng Hợp

### Checkpoint 1: Balloons ✅
- [ ] Deployer balance = 1000 BAL
- [ ] Frontend wallet balance = 10 BAL
- [ ] Total supply = 1000 BAL
- [ ] Name = "Balloons", Symbol = "BAL"

### Checkpoint 2: Init ✅
- [ ] Total liquidity = 5
- [ ] Deployer liquidity = 5
- [ ] DEX BAL balance = 5
- [ ] DEX STRK balance = 5
- [ ] Ratio 1:1 maintained

### Checkpoint 3: Price ✅
- [ ] Small swap calculates correctly
- [ ] Unbalanced ratio works
- [ ] Large swap shows slippage
- [ ] Fee 0.3% applied
- [ ] No reverts on valid inputs

### Checkpoint 4: Trading ✅
- [ ] Can swap STRK → BAL
- [ ] Can swap BAL → STRK
- [ ] Reserves update correctly
- [ ] Balances update correctly
- [ ] Events emitted
- [ ] Approve mechanism works
- [ ] Edge cases handled
- [ ] Price consistency verified

### Checkpoint 5: Liquidity ✅
- [ ] Can add liquidity
- [ ] Can remove liquidity
- [ ] LP tokens calculated correctly
- [ ] Multiple LPs work
- [ ] Events emitted
- [ ] Deposit function works
- [ ] Withdraw function works
- [ ] LP token calculations correct

---

## 🎯 Quick Testing Commands

```bash
# Full reset and test
make start-devnet    # Terminal 1
make deploy          # Terminal 2 (wait for devnet)
make start           # Terminal 3 (wait for deploy)

# Open frontend
open http://localhost:3000

# Quick redeploy (no reset)
yarn deploy:no-reset

# View deployment info
cat packages/snfoundry/deployments/devnet_latest.json | jq

# View deployer balance (từ terminal)
# (cần starknet CLI installed)
starknet call --address <BALLOONS_ADDRESS> --abi <ABI> --function balance_of --inputs <DEPLOYER_ADDRESS>
```

---

## 📝 Notes

- **Decimals**: Tất cả tokens đều dùng 18 decimals
  - 1 token = 1 * 10^18 = 1000000000000000000
  
- **Addresses**: 
  - Devnet deployer: `0x64b48806902a367c8598f4f95c305e8c1a1acba5f082d294a43793113115691`
  - STRK: `0x4718F5A0FC34CC1AF16A1CDEE98FFB20C31F5CD61D6AB07201858F4287C938D`

- **Conversion**:
  ```
  1 token = 1,000,000,000,000,000,000 (18 zeros)
  0.996 token = 996,000,000,000,000,000
  ```

- **Fee calculation**: 0.3% = 997/1000 = giữ 99.7%

---

**Ngày tạo**: 2025-10-02  
**Last updated**: 2025-10-02  
**Status**: Checkpoint 1-5 ✅, Core DEX functionality complete! 🎉

