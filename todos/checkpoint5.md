# 🌊 Checkpoint 5: Liquidity Functions

## 📋 Tổng quan

Checkpoint 5 implement các functions để users có thể thêm và rút liquidity từ DEX pool. Đây là phần cuối cùng của core DEX functionality.

## 🎯 Mục tiêu

- **Deposit**: Bất kỳ ai cũng có thể thêm liquidity vào pool
- **Withdraw**: Redeem LP tokens để lấy lại STRK + BAL
- **LP Tokens**: Track liquidity của mỗi address
- **Impermanent Loss**: LP value có thể tăng (do fees) hoặc giảm (do price change)

## 🛠️ Functions đã implement

### 1. `get_deposit_token_amount(strk_amount: u256) -> u256`
**Mục đích**: Tính token amount cần deposit dựa trên current ratio

**Logic**:
```cairo
token_amount = (strk_amount * token_reserves) / strk_reserves
```

**Ví dụ**: Nếu reserves = 5 STRK : 5 BAL, deposit 1 STRK cần 1 BAL

### 2. `deposit(strk_amount: u256) -> u256`
**Mục đích**: Thêm liquidity vào pool và mint LP tokens

**Steps**:
1. Calculate token amount needed
2. Transfer STRK from caller to contract
3. Transfer tokens from caller to contract (cần approve trước)
4. Calculate LP tokens to mint
5. Update total_liquidity và liquidity[caller]
6. Emit LiquidityProvided event
7. Return LP tokens minted

**LP Token Calculation**:
- **First deposit**: LP = strk_amount (simplified cho 1:1 ratio)
- **Subsequent deposits**: LP = (strk_amount * total_liquidity) / strk_reserves

### 3. `withdraw(amount: u256) -> (u256, u256)`
**Mục đích**: Rút liquidity từ pool và burn LP tokens

**Steps**:
1. Validate caller has enough liquidity
2. Calculate STRK và token amounts to return
3. Transfer STRK to caller
4. Transfer tokens to caller
5. Update total_liquidity và liquidity[caller]
6. Emit LiquidityRemoved event
7. Return (strk_amount, token_amount)

**Calculation**:
```cairo
strk_amount = (amount * strk_reserves) / current_total_liquidity
token_amount = (amount * token_reserves) / current_total_liquidity
```

### 4. `get_liquidity(lp_address: ContractAddress) -> u256`
**Mục đích**: Xem liquidity của một address

**Returns**: `liquidity[address]`

### 5. `get_total_liquidity() -> u256`
**Mục đích**: Xem tổng liquidity trong pool

**Returns**: `total_liquidity`

## ⚠️ Important Notes

### **Approve Required**
User phải approve BAL tokens trước khi deposit:
```typescript
// Frontend sẽ tự động handle
await balloonsContract.approve(dexAddress, tokenAmount);
await dexContract.deposit(strkAmount);
```

### **LP Token Mechanics**
- **LP Tokens**: Represent ownership của pool
- **First Deposit**: LP = strk_amount (simplified)
- **Subsequent Deposits**: Proportional to existing liquidity
- **Withdraw**: Burn LP tokens, get proportional share

### **Events**
- `LiquidityProvided`: Khi user deposit
- `LiquidityRemoved`: Khi user withdraw

### **Impermanent Loss**
- LP value có thể thay đổi do price movements
- Fees có thể compensate cho loss
- User cần hiểu rủi ro

## 🧪 Testing Scenarios

### **Test Case 1: First Deposit**
- **Input**: Deposit 1 STRK + 1 BAL
- **Expected**: LP = 1, total_liquidity = 1
- **Verify**: Reserves = 6 STRK : 6 BAL

### **Test Case 2: Subsequent Deposit**
- **Input**: Deposit 1 STRK + 1 BAL (after first deposit)
- **Expected**: LP proportional to existing liquidity
- **Verify**: Reserves = 7 STRK : 7 BAL

### **Test Case 3: Partial Withdraw**
- **Input**: Withdraw 0.5 LP tokens
- **Expected**: Get proportional STRK + BAL
- **Verify**: Reserves decreased proportionally

### **Test Case 4: Full Withdraw**
- **Input**: Withdraw all LP tokens
- **Expected**: Get all deposited tokens back
- **Verify**: User liquidity = 0

### **Test Case 5: Multiple Users**
- **Input**: User A deposit, User B deposit
- **Expected**: Each gets proportional LP tokens
- **Verify**: Both can withdraw independently

## 🔍 Key Implementation Details

### **Storage Updates**
```cairo
// Update total liquidity
self.total_liquidity.write(self.total_liquidity.read() + liquidity_minted);

// Update user liquidity
self.liquidity.write(caller, self.liquidity.read(caller) + liquidity_minted);
```

### **Event Emission**
```cairo
self.emit(LiquidityProvided {
    liquidity_provider: caller,
    liquidity_minted: liquidity_minted,
    strk_input: strk_amount,
    tokens_input: token_amount,
});
```

### **Validation**
```cairo
// Check sufficient liquidity for withdraw
assert(caller_liquidity >= amount, 'Insufficient liquidity');
```

## 🎯 Frontend Integration

### **DEX Tab Features**
- **Deposit Section**: Input STRK amount → calculate token amount needed
- **Withdraw Section**: Input LP amount → calculate STRK + BAL output
- **LP Balance Display**: Show user's current LP tokens
- **Total Liquidity Display**: Show pool's total liquidity

### **User Flow**
1. **Deposit**:
   - User inputs STRK amount
   - Frontend shows required BAL amount
   - User approves BAL tokens
   - Execute deposit transaction
   - Show LP tokens received

2. **Withdraw**:
   - User inputs LP amount to withdraw
   - Frontend shows STRK + BAL output
   - Execute withdraw transaction
   - Show tokens received

## ✅ Checklist Checkpoint 5

### **Implementation** ✅
- [x] `get_deposit_token_amount()` - Calculate token amount needed
- [x] `deposit()` - Add liquidity và mint LP tokens
- [x] `withdraw()` - Remove liquidity và burn LP tokens
- [x] `get_liquidity()` - Get user's LP tokens
- [x] `get_total_liquidity()` - Get total pool liquidity

### **Events** ✅
- [x] `LiquidityProvided` event emitted
- [x] `LiquidityRemoved` event emitted
- [x] Event parameters correct

### **Logic** ✅
- [x] LP token calculation correct
- [x] Proportional withdraw calculation
- [x] Storage updates correct
- [x] Validation logic implemented

### **Edge Cases** ✅
- [x] First deposit handling
- [x] Insufficient liquidity validation
- [x] Zero amount handling
- [x] Multiple users support

## 🚀 Next Steps

1. **Test trên frontend** - Dùng DEX tab để test deposit/withdraw
2. **Verify events** - Check Events tab cho liquidity events
3. **Test edge cases** - Multiple users, large amounts, etc.
4. **Checkpoint 6** - UI improvements và visualization

---

**Status**: ✅ Checkpoint 5 hoàn thành  
**Date**: 2025-10-02  
**Functions**: 5/5 implemented  
**Events**: 2/2 implemented
