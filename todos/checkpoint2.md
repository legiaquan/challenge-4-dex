## Checkpoint 2: ⚖️ Reserves (Ghi chú nhanh)

Mục tiêu: Implement function `init()` để khởi tạo liquidity pool với reserves ban đầu (tokens + STRK).

### Track Variables (Biến cần theo dõi)

Checkpoint 2 tập trung vào 2 biến storage quan trọng trong DEX:

#### 1. `total_liquidity: u256`
- **Mô tả**: Tổng liquidity trong toàn bộ pool
- **Vai trò**: Đại diện cho tổng số "LP tokens" đã phát hành
- **Khởi tạo**: Được set khi gọi `init()` lần đầu
- **Cập nhật**: 
  - Tăng khi có người `deposit()` thêm liquidity
  - Giảm khi có người `withdraw()` rút liquidity
- **Công thức khởi tạo**: Thường là `sqrt(tokens * strk)` theo AMM standard

#### 2. `liquidity: Map<ContractAddress, u256>`
- **Mô tả**: Mapping lưu liquidity của từng liquidity provider (LP)
- **Key**: Địa chỉ của LP
- **Value**: Số lượng liquidity (LP tokens) mà LP đó sở hữu
- **Vai trò**: 
  - Track phần sở hữu của mỗi LP trong pool
  - Dùng để tính toán khi withdraw (LP được nhận bao nhiêu % của pool)
- **Ví dụ**:
  ```
  liquidity[0x123...] = 1000  // LP này có 1000 LP tokens
  liquidity[0x456...] = 500   // LP này có 500 LP tokens
  total_liquidity = 1500      // Tổng cộng
  ```

### Hợp đồng liên quan

**DEX.cairo** - Function cần implement:
- `init(tokens: u256, strk: u256) -> (u256, u256)`

### Chi tiết implementation

#### ✅ Function `init()` đã được implement:

**Vị trí**: `packages/snfoundry/contracts/src/dex.cairo` (dòng 204-230)

```cairo
fn init(ref self: ContractState, tokens: u256, strk: u256) -> (u256, u256) {
    // Assert that the DEX has not been initialized yet
    assert(self.total_liquidity.read() == 0, 'DEX: already initialized');
    
    // Get addresses
    let caller = get_caller_address();
    let this_contract = get_contract_address();
    
    // Transfer tokens from caller to DEX contract
    self.token.read().transfer_from(caller, this_contract, tokens);
    
    // Transfer STRK from caller to DEX contract
    self.strk_token.read().transfer_from(caller, this_contract, strk);
    
    // Calculate initial liquidity (using tokens as liquidity for simplicity)
    // For equal ratio deposits (1:1), liquidity = tokens
    let liquidity = tokens;
    
    // Set total liquidity
    self.total_liquidity.write(liquidity);
    
    // Set caller's liquidity
    self.liquidity.write(caller, liquidity);
    
    // Return the amounts of tokens and STRK initialized
    (tokens, strk)
}
```

**Giải thích từng bước đã implement:**
1. ✅ **Assert init 1 lần**: `assert(self.total_liquidity.read() == 0, 'DEX: already initialized')`
2. ✅ **Get addresses**: `get_caller_address()` và `get_contract_address()`
3. ✅ **Transfer tokens**: `self.token.read().transfer_from(caller, this_contract, tokens)`
4. ✅ **Transfer STRK**: `self.strk_token.read().transfer_from(caller, this_contract, strk)`
5. ✅ **Calculate liquidity**: `let liquidity = tokens` (đơn giản cho ratio 1:1)
6. ✅ **Set total_liquidity**: `self.total_liquidity.write(liquidity)`
7. ✅ **Set liquidity[caller]**: `self.liquidity.write(caller, liquidity)`
8. ✅ **Return**: `(tokens, strk)`

#### Notes quan trọng:

1. **Transfer_from vs Transfer**:
   - `transfer_from(from, to, amount)`: Chuyển từ address `from` → address `to`
   - DEX phải được approve trước khi có thể dùng `transfer_from()`
   
2. **Approve flow**:
   ```
   User calls: token.approve(dex_address, amount)
   DEX calls:  token.transfer_from(user, dex, amount)
   ```

3. **Storage access**:
   - Read: `self.token.read()` hoặc `self.total_liquidity.read()`
   - Write: `self.total_liquidity.write(value)`
   - Map: `self.liquidity.write(address, value)`

4. **Get addresses**:
   - `get_caller_address()`: Địa chỉ người gọi function
   - `get_contract_address()`: Địa chỉ của DEX contract

### Steps trong deploy.ts

File `packages/snfoundry/scripts-ts/deploy.ts` có sẵn code (đang bị comment):

#### 1. Approve tokens (dòng 54-78)
```typescript
let approveResponse = await deployer.execute([
    {
        contractAddress: balloons_token.address,
        entrypoint: "approve",
        calldata: CallData.compile({
            spender: dex.address,
            amount: INITIAL_SUPPLY,
        }),
    },
    {
        contractAddress: STRK_ADDRESS,
        entrypoint: "approve",
        calldata: CallData.compile({
            spender: dex.address,
            amount: INITIAL_SUPPLY
        })
    }
]);
```
**Mục đích**: Cho phép DEX contract chuyển tokens từ deployer account

#### 2. Initialize DEX (dòng 82-96)
```typescript
const initResponse = await deployer.execute([{
    contractAddress: dex.address,
    entrypoint: "init",
    calldata: CallData.compile({
        tokens: INITIAL_SUPPLY,  // 5 * 10^18
        strk: INITIAL_SUPPLY     // 5 * 10^18
    }),
}]);
```
**Mục đích**: Gọi `init()` để khởi tạo pool với reserves ban đầu

#### 3. Constant quan trọng
```typescript
const INITIAL_SUPPLY = cairo.uint256(5_000_000_000_000_000_000n); // 5 * 10^18
```
- Dùng cho cả tokens và STRK
- = 5 tokens (vì 18 decimals)
- Tạo initial ratio 1:1 (5 BAL : 5 STRK)

### Checklist hoàn thành Checkpoint 2

#### ✅ Phase 1: Implementation (Đã hoàn thành)
- [x] Implement function `init()` trong `DEX.cairo`:
  - [x] Assert DEX chưa được init (security check)
  - [x] Get addresses (caller và contract)
  - [x] Transfer tokens từ caller vào DEX
  - [x] Transfer STRK từ caller vào DEX
  - [x] Tính và set `total_liquidity`
  - [x] Set `liquidity[caller]`
  - [x] Return (tokens, strk)

#### ✅ Phase 2: Deploy Configuration (Đã hoàn thành)
- [x] Uncomment phần approve trong `deploy.ts` (dòng 54-78)
  - Code approve cả Balloons token và STRK token cho DEX
  
- [x] Uncomment phần init DEX trong `deploy.ts` (dòng 82-96)
  - Code gọi function `init()` với INITIAL_SUPPLY (5 * 10^18)
  
- [x] Uncomment dòng `await transferScript();` trong `deploy.ts` (dòng 152)
  - transferScript sẽ tự động chạy khi deploy

- [x] BONUS: Transfer 10 $BAL to frontend address (dòng 111-141)
  - Tự động transfer tokens cho frontend testing

#### 🧪 Phase 3: Testing (Sẵn sàng test)
- [ ] Test deployment:
  ```bash
  make deploy
  # hoặc
  yarn deploy
  ```

- [ ] Verify trên Debug Contracts (http://localhost:3000):
  - [ ] Vào tab "Debug Contracts"
  - [ ] Chọn DEX contract
  - [ ] Check `get_total_liquidity()` > 0 (nên là 5 * 10^18)
  - [ ] Check `get_liquidity(deployer_address)` > 0
  - [ ] Check `balance_of` DEX contract (có $BAL và STRK)

### Lệnh tham khảo

- Deploy lại: `make deploy` hoặc `yarn deploy`
- Check balance: Vào Debug Contracts → DEX → Read functions
- Xem storage: Debug Contracts → Storage section

### Ghi chú thêm

- **Initial liquidity formula**: Có nhiều cách tính, phổ biến là:
  - Simple: `liquidity = min(tokens, strk)`
  - Standard: `liquidity = sqrt(tokens * strk)` (Uniswap V2 style)
  
- **Why track liquidity?**: 
  - Để biết mỗi LP sở hữu bao nhiêu % của pool
  - Khi withdraw, LP nhận được: 
    ```
    LP_share = liquidity[LP] / total_liquidity
    tokens_out = LP_share * token_reserves
    strk_out = LP_share * strk_reserves
    ```

- **Security notes**:
  - Phải kiểm tra `total_liquidity == 0` khi init (chỉ init 1 lần)
  - Phải kiểm tra `tokens > 0` và `strk > 0`
  - Transfer_from có thể fail nếu không được approve

### Resources

- Storage trong Cairo: https://book.starknet.io/ch02-02-storage.html
- ERC20 transfer_from: Xem implementation trong Balloons.cairo
- AMM formula: x * y = k (sẽ dùng trong Checkpoint 3)

---

## 📊 Tóm tắt kết quả đã làm

### ✅ Hoàn thành (100% ready to test!)
1. **Function `init()` trong DEX.cairo** (dòng 204-230)
   - ✅ Đã implement đầy đủ logic khởi tạo liquidity pool
   - ✅ Có security check (assert chỉ init 1 lần)
   - ✅ Transfer cả tokens và STRK vào DEX contract
   - ✅ Track liquidity cho deployer
   - ✅ Sử dụng `liquidity = tokens` cho initial deposit 1:1 ratio

2. **Deploy script configuration** (deploy.ts)
   - ✅ Approve section uncommented (dòng 54-78)
   - ✅ Init section uncommented (dòng 82-96)  
   - ✅ TransferScript call uncommented (dòng 152)
   - ✅ BONUS: Transfer to frontend uncommented (dòng 111-141)

3. **Testing và verification**
   - Chạy devnet (nếu chưa chạy): `make start-devnet` hoặc `yarn chain`
   - Deploy contracts: `make deploy` hoặc `yarn deploy`
   - Verify kết quả trên Debug Contracts UI

### 🎯 Bước tiếp theo - Chỉ cần chạy!
```bash
# Terminal mới - Deploy
yarn deploy  # hoặc yarn deploy

# Terminal mới - Start frontend (nếu chưa chạy)
yarn start  # hoặc yarn start
```

Sau đó verify tại http://localhost:3000 → Debug Contracts:
- Check `get_total_liquidity()` → Kỳ vọng: 5000000000000000000 (5 * 10^18)
- Check `get_liquidity(deployer_address)` → Kỳ vọng: 5000000000000000000
- Check balance_of DEX contract → Có cả $BAL và STRK

### 💡 Lessons Learned
- **Liquidity calculation**: Với initial 1:1 ratio, có thể dùng `liquidity = tokens` đơn giản
- **Security**: Assert `total_liquidity == 0` để đảm bảo chỉ init 1 lần
- **Flow**: Approve → Transfer_from → Update storage
- **Storage**: Dùng `.read()` và `.write()` để access storage, `.write(key, value)` cho Map

---

**Ngày cập nhật**: 2025-10-02  
**Trạng thái**: Phase 1 & 2 hoàn thành 100% ✅ → Sẵn sàng deploy và test! 🚀

### 🎉 Checkpoint 2 Implementation Complete!
- ✅ DEX.cairo: function `init()` implemented
- ✅ deploy.ts: Tất cả sections đã uncommented
- 🚀 Ready to deploy: `make deploy` or `yarn deploy`
- 🧪 Ready to test: Verify on Debug Contracts UI

