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

## 🧪 Checkpoint 4: Trading Functions (Coming Soon)

### Functions cần test:
- `strk_to_token()` - Swap STRK → BAL
- `token_to_strk()` - Swap BAL → STRK

### Prerequisites:
- Checkpoint 3 hoàn thành
- Events đã được add vào enum Event

### Test scenarios:
1. Swap STRK → BAL
2. Swap BAL → STRK
3. Verify reserves thay đổi
4. Verify user balances
5. Check swap events

**Note**: Sẽ update chi tiết khi implement Checkpoint 4.

---

## 💧 Checkpoint 5: Liquidity Functions (Coming Soon)

### Functions cần test:
- `deposit()` - Add liquidity
- `withdraw()` - Remove liquidity
- `get_deposit_token_amount()` - Calculate token amount needed

### Test scenarios:
1. Add liquidity
2. Remove liquidity
3. Multiple LPs
4. LP token calculations

**Note**: Sẽ update chi tiết khi implement Checkpoint 5.

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

### Checkpoint 4: Trading 🔄
- [ ] Can swap STRK → BAL
- [ ] Can swap BAL → STRK
- [ ] Reserves update correctly
- [ ] Balances update correctly
- [ ] Events emitted
- [ ] (Sẽ update khi implement)

### Checkpoint 5: Liquidity 🔄
- [ ] Can add liquidity
- [ ] Can remove liquidity
- [ ] LP tokens calculated correctly
- [ ] Multiple LPs work
- [ ] Events emitted
- [ ] (Sẽ update khi implement)

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
**Status**: Checkpoint 1-3 ✅, Checkpoint 4-5 coming soon 🔄

