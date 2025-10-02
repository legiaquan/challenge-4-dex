## Checkpoint 4: 🤝 Trading (Ghi chú nhanh)

Mục tiêu: Implement 2 trading functions để swap tokens và thêm events vào contract.

---

## 🎯 Yêu cầu chính

### 1. **Implement 2 swap functions:**
- `strk_to_token()` - Swap STRK → $BAL
- `token_to_strk()` - Swap $BAL → STRK

### 2. **Add swap events vào enum Event:**
- `StrkToTokenSwap` - Event khi swap STRK → $BAL
- `TokenToStrkSwap` - Event khi swap $BAL → STRK

---

## 📝 Chi tiết Implementation

### Part 1: Add Events vào enum Event

**Vị trí**: `packages/snfoundry/contracts/src/dex.cairo` (dòng 128-136)

**Hiện tại**:
```cairo
// Todo Checkpoint 4:  Define the events.
#[event]
#[derive(Drop, starknet::Event)]
enum Event {
    #[flat]
    OwnableEvent: OwnableComponent::Event,
    LiquidityProvided: LiquidityProvided,
    LiquidityRemoved: LiquidityRemoved,
    // ❌ THIẾU: StrkToTokenSwap
    // ❌ THIẾU: TokenToStrkSwap
}
```

**Cần sửa thành**:
```cairo
// Todo Checkpoint 4:  Define the events.
#[event]
#[derive(Drop, starknet::Event)]
enum Event {
    #[flat]
    OwnableEvent: OwnableComponent::Event,
    LiquidityProvided: LiquidityProvided,
    LiquidityRemoved: LiquidityRemoved,
    StrkToTokenSwap: StrkToTokenSwap,      // ✅ ADD THIS
    TokenToStrkSwap: TokenToStrkSwap,      // ✅ ADD THIS
}
```

**Lý do**: 
- Event struct đã được định nghĩa (dòng 138-152)
- Nhưng chưa được thêm vào enum Event
- Frontend đang try to listen events này → gây lỗi "Event not found"

---

### Part 2: Implement `strk_to_token()` function

**Vị trí**: `packages/snfoundry/contracts/src/dex.cairo` (dòng 282-295)

**Signature**:
```cairo
fn strk_to_token(ref self: ContractState, strk_input: u256) -> u256
```

**Logic flow**:

#### Step 1: Get addresses
```cairo
let caller = get_caller_address();
let this_contract = get_contract_address();
```

#### Step 2: Get reserves (BEFORE transfer)
```cairo
// x_reserves = STRK reserve (before new STRK added)
let strk_reserves = self.strk_token.read().balance_of(this_contract);

// y_reserves = BAL reserve
let token_reserves = self.token.read().balance_of(this_contract);
```

**⚠️ Important**: Get reserves BEFORE transfer để tính đúng price!

#### Step 3: Transfer STRK from caller to contract
```cairo
self.strk_token.read().transfer_from(caller, this_contract, strk_input);
```

#### Step 4: Calculate token output using price()
```cairo
let token_output = self.price(strk_input, strk_reserves, token_reserves);
```

#### Step 5: Validate output > 0
```cairo
assert(token_output > 0, 'Insufficient output amount');
```

#### Step 6: Transfer tokens to caller
```cairo
self.token.read().transfer(caller, token_output);
```

#### Step 7: Emit event
```cairo
self.emit(StrkToTokenSwap {
    swapper: caller,
    token_output: token_output,
    strk_input: strk_input,
});
```

#### Step 8: Return token output
```cairo
token_output
```

**Complete function**:
```cairo
fn strk_to_token(ref self: ContractState, strk_input: u256) -> u256 {
    // 1. Get addresses
    let caller = get_caller_address();
    let this_contract = get_contract_address();
    
    // 2. Get reserves BEFORE transfer
    let strk_reserves = self.strk_token.read().balance_of(this_contract);
    let token_reserves = self.token.read().balance_of(this_contract);
    
    // 3. Transfer STRK from caller to contract
    self.strk_token.read().transfer_from(caller, this_contract, strk_input);
    
    // 4. Calculate token output
    let token_output = self.price(strk_input, strk_reserves, token_reserves);
    
    // 5. Validate output
    assert(token_output > 0, 'Insufficient output amount');
    
    // 6. Transfer tokens to caller
    self.token.read().transfer(caller, token_output);
    
    // 7. Emit event
    self.emit(StrkToTokenSwap {
        swapper: caller,
        token_output: token_output,
        strk_input: strk_input,
    });
    
    // 8. Return
    token_output
}
```

---

### Part 3: Implement `token_to_strk()` function

**Vị trí**: `packages/snfoundry/contracts/src/dex.cairo` (dòng 297-310)

**Signature**:
```cairo
fn token_to_strk(ref self: ContractState, token_input: u256) -> u256
```

**Logic flow**: Tương tự `strk_to_token()` nhưng ngược lại

#### Step 1: Get addresses
```cairo
let caller = get_caller_address();
let this_contract = get_contract_address();
```

#### Step 2: Get reserves (BEFORE transfer)
```cairo
// x_reserves = BAL reserve (before new tokens added)
let token_reserves = self.token.read().balance_of(this_contract);

// y_reserves = STRK reserve
let strk_reserves = self.strk_token.read().balance_of(this_contract);
```

**⚠️ Note**: Thứ tự reserves ngược với strk_to_token()!

#### Step 3: Transfer tokens from caller to contract
```cairo
self.token.read().transfer_from(caller, this_contract, token_input);
```

**⚠️ Remember**: User phải approve tokens trước!

#### Step 4: Calculate STRK output using price()
```cairo
let strk_output = self.price(token_input, token_reserves, strk_reserves);
```

#### Step 5: Validate output > 0
```cairo
assert(strk_output > 0, 'Insufficient output amount');
```

#### Step 6: Transfer STRK to caller
```cairo
self.strk_token.read().transfer(caller, strk_output);
```

#### Step 7: Emit event
```cairo
self.emit(TokenToStrkSwap {
    swapper: caller,
    tokens_input: token_input,
    strk_output: strk_output,
});
```

#### Step 8: Return STRK output
```cairo
strk_output
```

**Complete function**:
```cairo
fn token_to_strk(ref self: ContractState, token_input: u256) -> u256 {
    // 1. Get addresses
    let caller = get_caller_address();
    let this_contract = get_contract_address();
    
    // 2. Get reserves BEFORE transfer
    let token_reserves = self.token.read().balance_of(this_contract);
    let strk_reserves = self.strk_token.read().balance_of(this_contract);
    
    // 3. Transfer tokens from caller to contract
    self.token.read().transfer_from(caller, this_contract, token_input);
    
    // 4. Calculate STRK output
    let strk_output = self.price(token_input, token_reserves, strk_reserves);
    
    // 5. Validate output
    assert(strk_output > 0, 'Insufficient output amount');
    
    // 6. Transfer STRK to caller
    self.strk_token.read().transfer(caller, strk_output);
    
    // 7. Emit event
    self.emit(TokenToStrkSwap {
        swapper: caller,
        tokens_input: token_input,
        strk_output: strk_output,
    });
    
    // 8. Return
    strk_output
}
```

---

## 🔍 Key Concepts

### 1. **Transfer_from vs Transfer**

**transfer_from(from, to, amount)**:
- Chuyển tokens TỪ address `from` ĐẾN address `to`
- Cần approve trước
- Dùng khi DEX pull tokens từ user

**transfer(to, amount)**:
- Chuyển tokens từ caller (contract) đến address `to`
- Không cần approve (contract tự transfer)
- Dùng khi DEX send tokens cho user

### 2. **Thứ tự quan trọng**

```cairo
// ✅ ĐÚNG: Get reserves TRƯỚC khi transfer
let reserves = token.balance_of(contract);  // Old balance
transfer_from(user, contract, input);       // Add new tokens
let output = price(input, reserves, ...);   // Calculate với old balance

// ❌ SAI: Get reserves SAU khi transfer
transfer_from(user, contract, input);       // Add new tokens first
let reserves = token.balance_of(contract);  // New balance (wrong!)
let output = price(input, reserves, ...);   // Wrong calculation!
```

**Lý do**: Function `price()` tính dựa trên reserves TRƯỚC swap, không phải sau!

### 3. **Reserves mapping**

**strk_to_token()**:
```
x_input = STRK (input)
x_reserves = STRK reserve
y_reserves = BAL reserve
y_output = BAL (output)
```

**token_to_strk()**:
```
x_input = BAL (input)
x_reserves = BAL reserve
y_reserves = STRK reserve
y_output = STRK (output)
```

### 4. **Approve flow**

Trước khi swap, user phải approve:

```typescript
// Frontend: Approve tokens
await token.approve(dex_address, amount);

// Sau đó mới swap
await dex.token_to_strk(amount);
```

Không cần approve STRK vì STRK được transfer trong transaction call.

---

## 📊 Example Scenarios

### Scenario 1: Swap 1 STRK → BAL

**Initial state**:
```
Pool: 5 STRK / 5 BAL
User balance: 100 STRK, 0 BAL
```

**User calls**:
```cairo
dex.strk_to_token(1 * 10^18)  // Swap 1 STRK
```

**Execution**:
```
1. Get reserves: strk_reserves = 5 * 10^18, token_reserves = 5 * 10^18
2. Transfer 1 STRK from user to DEX
3. Calculate: token_output = price(1e18, 5e18, 5e18) ≈ 0.996 * 10^18
4. Transfer 0.996 BAL to user
5. Emit StrkToTokenSwap event
6. Return 0.996 * 10^18
```

**Final state**:
```
Pool: 6 STRK / 4.004 BAL (ratio changed)
User balance: 99 STRK, 0.996 BAL
```

### Scenario 2: Swap 1 BAL → STRK

**Initial state**:
```
Pool: 6 STRK / 4 BAL (after scenario 1)
User balance: 10 BAL, 0 STRK
```

**User calls**:
```typescript
// 1. Approve first!
await balloons.approve(dex_address, 1 * 10^18)

// 2. Then swap
await dex.token_to_strk(1 * 10^18)
```

**Execution**:
```
1. Get reserves: token_reserves = 4 * 10^18, strk_reserves = 6 * 10^18
2. Transfer 1 BAL from user to DEX
3. Calculate: strk_output = price(1e18, 4e18, 6e18) ≈ 1.493 * 10^18
4. Transfer 1.493 STRK to user
5. Emit TokenToStrkSwap event
6. Return 1.493 * 10^18
```

**Final state**:
```
Pool: 4.507 STRK / 5 BAL
User balance: 9 BAL, 1.493 STRK
```

---

## ✅ Checklist hoàn thành Checkpoint 4

### ✅ Phase 1: Events (Fix lỗi frontend) - HOÀN THÀNH
- [x] Add `StrkToTokenSwap` vào enum Event
- [x] Add `TokenToStrkSwap` vào enum Event
- [x] Verify no linter errors

### ✅ Phase 2: Swap Functions Implementation - HOÀN THÀNH
- [x] Implement `strk_to_token()`:
  - [x] Get addresses (caller, contract)
  - [x] Get reserves BEFORE transfer
  - [x] Transfer STRK from caller to contract
  - [x] Calculate token output using price()
  - [x] Validate output > 0
  - [x] Transfer tokens to caller
  - [x] Emit StrkToTokenSwap event
  - [x] Return token output

- [x] Implement `token_to_strk()`:
  - [x] Get addresses (caller, contract)
  - [x] Get reserves BEFORE transfer
  - [x] Transfer tokens from caller to contract
  - [x] Calculate STRK output using price()
  - [x] Validate output > 0
  - [x] Transfer STRK to caller
  - [x] Emit TokenToStrkSwap event
  - [x] Return STRK output

### 🧪 Phase 3: Testing (Sẵn sàng test)
- [ ] Deploy contract: `make deploy`
- [ ] Test strk_to_token():
  - [ ] Approve STRK (if needed)
  - [ ] Call function with various amounts
  - [ ] Verify output amounts
  - [ ] Check reserves changed
  - [ ] Check balances updated
  - [ ] Verify event emitted

- [ ] Test token_to_strk():
  - [ ] Approve $BAL tokens
  - [ ] Call function with various amounts
  - [ ] Verify output amounts
  - [ ] Check reserves changed
  - [ ] Check balances updated
  - [ ] Verify event emitted

- [ ] Test edge cases:
  - [ ] Large swap (high slippage)
  - [ ] Multiple swaps in a row
  - [ ] Insufficient balance/allowance

- [ ] Check Events page works (no more "Event not found" error)

---

## 🚨 Common Mistakes & Security

### ❌ Mistake 1: Get reserves AFTER transfer
```cairo
// WRONG!
self.strk_token.read().transfer_from(caller, this_contract, strk_input);
let strk_reserves = self.strk_token.read().balance_of(this_contract); // Wrong!
```

**Solution**: Always get reserves BEFORE transfer.

### ❌ Mistake 2: Forget to validate output
```cairo
// WRONG! No validation
let token_output = self.price(...);
self.token.read().transfer(caller, token_output); // Could be 0!
```

**Solution**: Add assert to validate output > 0.

### ❌ Mistake 3: Wrong reserves order
```cairo
// WRONG! Swapped reserves
let token_output = self.price(strk_input, token_reserves, strk_reserves); // Wrong order!
```

**Solution**: x_reserves phải là input asset, y_reserves là output asset.

### ❌ Mistake 4: Forget to emit events
```cairo
// WRONG! No event
self.token.read().transfer(caller, token_output);
return token_output; // Missing emit!
```

**Solution**: Always emit events để frontend có thể track.

### ❌ Mistake 5: Events not in enum
```cairo
// WRONG! Event struct defined but not in enum
struct StrkToTokenSwap { ... }  // ✅ Defined

enum Event {
    // ❌ NOT included in enum!
}
```

**Solution**: Add events vào enum Event (Part 1).

---

## 🎓 Concept Quiz

**Q1**: Tại sao phải get reserves TRƯỚC khi transfer?
- **A**: Vì function `price()` tính dựa trên reserves trước swap. Nếu get sau transfer, reserves đã bao gồm input → calculation sai.

**Q2**: Sự khác biệt giữa `strk_to_token()` và `token_to_strk()`?
- **A**: 
  - `strk_to_token()`: x = STRK (input), y = BAL (output)
  - `token_to_strk()`: x = BAL (input), y = STRK (output)
  - Thứ tự reserves ngược nhau!

**Q3**: Tại sao cần approve cho `token_to_strk()` nhưng không cho `strk_to_token()`?
- **A**: 
  - `token_to_strk()`: DEX pull BAL tokens từ user → cần approve
  - `strk_to_token()`: STRK được gửi trong transaction call → không cần approve riêng

**Q4**: Event emission có bắt buộc không?
- **A**: Về mặt technical không bắt buộc, nhưng HIGHLY recommended để frontend có thể track transactions, show history, và update UI.

**Q5**: Có cần check balance trước khi swap không?
- **A**: Cairo's `transfer_from` sẽ tự động fail nếu insufficient balance/allowance, nhưng good practice là validate input > 0 và output > 0.

---

## 📚 Resources

- [Uniswap V2 Swap Implementation](https://docs.uniswap.org/contracts/v2/guides/smart-contract-integration/trading-from-a-smart-contract)
- Cairo Events: https://book.starknet.io/ch02-03-events.html
- ERC20 approve/transfer_from: https://eips.ethereum.org/EIPS/eip-20

---

## 📊 Tóm tắt

### Cần làm:
1. **Add 2 events vào enum Event** (fix lỗi frontend)
2. **Implement `strk_to_token()`** (8 steps)
3. **Implement `token_to_strk()`** (8 steps)
4. **Test trên frontend**

### Key points:
- ✅ Get reserves BEFORE transfer
- ✅ Use `price()` để calculate output
- ✅ Emit events
- ✅ Validate output > 0
- ✅ Right reserves order: x = input, y = output

### ✅ Hoàn thành:
1. ✅ Events đã add vào enum (StrkToTokenSwap, TokenToStrkSwap)
2. ✅ Function `strk_to_token()` implemented (8 steps)
3. ✅ Function `token_to_strk()` implemented (8 steps)
4. ✅ No linter errors

### 🚀 Bước tiếp theo:
1. Deploy: `make deploy`
2. Test swap functions trên Debug Contracts
3. Test trên DEX Tab (nếu có UI)
4. Verify Events page không còn lỗi

---

**Ngày tạo**: 2025-10-02  
**Ngày hoàn thành**: 2025-10-02  
**Trạng thái**: Phase 1 & 2 hoàn thành 100% ✅ → Sẵn sàng test! 🚀  
**Dependencies**: Checkpoint 3 (price function) ✅

### 🎉 Checkpoint 4 Implementation Complete!
- ✅ Events: StrkToTokenSwap, TokenToStrkSwap added to enum
- ✅ DEX.cairo: `strk_to_token()` implemented (dòng 293-323)
- ✅ DEX.cairo: `token_to_strk()` implemented (dòng 334-364)
- ✅ No linter errors
- 🚀 Ready to deploy and test
- 🧪 Ready for Checkpoint 5 (Liquidity functions)

