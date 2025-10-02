## Checkpoint 1: The Structure (Ghi chú nhanh)

Mục tiêu: Nắm cấu trúc dự án, xác định các hợp đồng cốt lõi và biết cách kiểm tra chúng trên giao diện Debug Contracts.

### Hợp đồng liên quan
- **DEX.cairo**: Hợp đồng DEX chính – kết nối tới ERC20.
- **Balloons.cairo**: ERC20 mẫu – mint 1000 $BAL cho deployer.

Vị trí cụ thể của file Cairo nằm trong workspace `packages/snfoundry/contracts/src/` (và có thể tham chiếu trong scripts deploy). Bạn có thể mở thư mục này để xem các file `.cairo` tương ứng.

### Chi tiết chức năng và lưu ý

#### **Balloons.cairo** (ERC20 Token)
- **Chức năng**: Token ERC20 với tên "Balloons" và symbol "BAL"
- **Decimals**: 18 (giống ETH/STRK)
- **Initial Supply**: Cần implement trong constructor để mint 1000 $BAL cho deployer
- **Lưu ý**: 
  - Hiện tại constructor chưa mint token (line 119 chưa có code mint)
  - Cần hoàn thiện constructor để mint initial_supply cho recipient
  - Token này sẽ được dùng trong DEX để swap với STRK

#### **DEX.cairo** (Decentralized Exchange)
- **Chức năng**: AMM (Automated Market Maker) cho cặp BAL ↔ STRK
- **Các function chính**:
  - `init()`: Khởi tạo DEX với reserves ban đầu (cần implement)
  - `price()`: Tính giá theo công thức AMM (cần implement)
  - `strk_to_token()` / `token_to_strk()`: Swap tokens (cần implement)
  - `deposit()` / `withdraw()`: Thêm/rút liquidity (cần implement)
  - `get_liquidity()`: Xem liquidity của user (cần implement)
- **Storage quan trọng**:
  - `total_liquidity`: Tổng liquidity trong pool
  - `liquidity`: Map address → amount của mỗi LP
  - `strk_token`: Reference đến STRK token contract
  - `token`: Reference đến Balloons token contract
- **Lưu ý**:
  - Tất cả function chính đều trả về 0 (chưa implement logic)
  - Có sẵn events cho swap và liquidity operations
  - Có Ownable pattern để quản lý ownership
  - Constants `TokensPerStrk = 100` (có thể dùng cho initial ratio)

### Chạy môi trường local để quan sát
Bạn có thể dùng Makefile (đã tích hợp chuỗi lệnh) cho nhanh:

1) Khởi động Devnet (Docker):
```bash
make start-devnet
```
2) Deploy contracts lên devnet:
```bash
make deploy
```
3) Start frontend (Next.js):
```bash
make start
```

Mở giao diện: `http://localhost:3000` → vào tab `Debug Contracts` để kiểm tra contract, storage, events.

Ghi chú:
- Có thể redeploy nhiều lần để test logic: `make deploy` (tương đương `yarn deploy`).
- Nếu muốn giữ trạng thái, dùng `yarn deploy:no-reset` (xem thêm trong `package.json`).

### Checklist hoàn thành Checkpoint 1
- [x] Xác định file `DEX.cairo` và `Balloons.cairo` trong repo:
  - `packages/snfoundry/contracts/src/dex.cairo`
  - `packages/snfoundry/contracts/src/Balloons.cairo`
- [x] Chạy devnet và deploy contracts lên local (đã dùng `make start-devnet` và `make deploy`).
- [x] Mở UI, truy cập tab `Debug Contracts` và xem thông tin contract.
- [x] Redeploy lại để quen quy trình (nếu cần) bằng `make deploy`/`yarn deploy`.

### Lệnh tham khảo nhanh (không dùng Makefile)
- Chạy devnet: `yarn chain`
- Deploy: `yarn deploy`
- Start frontend: `yarn start`

Tip: Sau khi mọi thứ chạy, thử refresh trang `Debug Contracts` và đối chiếu địa chỉ contract mới mỗi lần deploy.

### Kết quả Checkpoint 1 (devnet)
- Địa chỉ đã deploy gần nhất (theo `packages/snfoundry/deployments/devnet_latest.json`):
  - Balloons (ERC20):
    - classHash: `0x67eca4468cc7a9dda9b0f55aeb6b647fd579da457590bc418ce200443456622`
    - address: `0x6c911d721d0ff58b766c5a3c59d096fe88f3cffc0d5d826272270b901f0c069`
  - Dex:
    - classHash: `0x35654d3fb450ed3f6556d5c4b61f29ca06e898f2e9a4f5e7ee3148f86a4d554`
    - address: `0x1850e3b8e58242b3112ed1d714a22af0f270d5e223390bd21ca6866ddfc409f`

- Đã mở UI tại `http://localhost:3000` và truy cập tab `Debug Contracts` để xác minh contract và storage.
- Checkpoint 1 hoàn tất: nắm được cấu trúc, chạy devnet, deploy và xem contract trên UI.


