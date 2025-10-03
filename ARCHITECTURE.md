# 🏗️ DEX Architecture Documentation

## 🎯 **Tổng quan**

Dự án này xây dựng một Decentralized Exchange (DEX) trên Starknet blockchain, sử dụng Automated Market Maker (AMM) model với cặp token BAL ↔ STRK.

## 🏗️ **Kiến trúc hệ thống**

### **Core Components**

#### **1. Smart Contracts (Cairo)**
- **`DEX.cairo`**: Main DEX contract với AMM logic
- **`Balloons.cairo`**: ERC20 token contract (BAL)

#### **2. Frontend (Next.js)**
- **UI Interface**: Swap tokens, view balances, visualize slippage
- **Wallet Integration**: Argent X, Braavos
- **Real-time Updates**: Contract interactions

#### **3. Development Environment**
- **Starknet Devnet**: Local blockchain development
- **Docker**: Containerized devnet
- **Scaffold-Stark**: Development framework

## 🔄 **Smart Contract Architecture**

### **DEX Contract Functions**

```mermaid
graph TD
    A[DEX Contract] --> B[Initialization]
    A --> C[Trading Functions]
    A --> D[Liquidity Functions]
    A --> E[Utility Functions]
    
    B --> B1[init - Initialize DEX]
    B --> B2[set_token - Set token contract]
    
    C --> C1[strk_to_token - Swap STRK → BAL]
    C --> C2[token_to_strk - Swap BAL → STRK]
    C --> C3[price - Calculate token price]
    
    D --> D1[deposit - Add liquidity]
    D --> D2[withdraw - Remove liquidity]
    D --> D3[get_liquidity - Get user LP tokens]
    
    E --> E1[get_total_liquidity - Total pool liquidity]
    E --> E2[get_deposit_token_amount - Calculate token amount]
```

### **Token Flow**

```mermaid
sequenceDiagram
    participant U as User
    participant D as DEX Contract
    participant B as Balloons Contract
    participant S as STRK Contract
    
    Note over U,S: Swap STRK → BAL
    U->>D: strk_to_token(amount)
    D->>S: transfer_from(user, dex, amount)
    D->>B: transfer(user, calculated_amount)
    D-->>U: BAL tokens
    
    Note over U,S: Add Liquidity
    U->>B: approve(DEX, token_amount)
    U->>D: deposit(strk_amount)
    D->>S: transfer_from(user, dex, strk_amount)
    D->>B: transfer_from(user, dex, token_amount)
    D->>D: mint LP tokens to user
```

## 🌐 **Network Architecture**

```mermaid
graph TB
    subgraph "Development Environment"
        A[Local Devnet<br/>Docker Container]
        B[Smart Contracts<br/>DEX.cairo, Balloons.cairo]
        C[Frontend<br/>Next.js App]
    end
    
    subgraph "Testnet Environment"
        D[Sepolia Testnet<br/>Starknet]
        E[Deployed Contracts<br/>Production-like]
        F[Public Frontend<br/>Testnet UI]
    end
    
    subgraph "User Interface"
        G[Argent X Wallet]
        H[Braavos Wallet]
        I[Web Interface<br/>localhost:3000]
    end
    
    A --> B
    B --> C
    C --> G
    C --> H
    C --> I
    
    D --> E
    E --> F
    F --> G
    F --> H
```

## 📊 **AMM (Automated Market Maker) Logic**

### **Constant Product Formula**

```
x * y = k
```

Where:
- `x` = STRK reserves
- `y` = BAL token reserves  
- `k` = Constant product

### **Price Calculation**

```mermaid
graph LR
    A[Input Amount] --> B[Calculate Price Impact]
    B --> C[Apply Slippage]
    C --> D[Output Amount]
    
    B --> B1[Current Price = y/x]
    B --> B2[New Price = (y - Δy)/(x + Δx)]
    B --> B3[Slippage = (New Price - Current Price) / Current Price]
```

### **Liquidity Pool Mechanics**

```mermaid
graph TD
    A[Liquidity Provider] --> B[Deposit STRK + BAL]
    B --> C[Mint LP Tokens]
    C --> D[Share of Pool]
    
    E[Swap Fee] --> F[Accumulate in Pool]
    F --> G[Increase LP Token Value]
    
    H[Price Change] --> I[Impermanent Loss]
    I --> J[Decrease LP Token Value]
```

## 🛠️ **Development Workflow**

### **Local Development**

```mermaid
graph TD
    A[Start Devnet] --> B[Docker Compose Up]
    B --> C[Deploy Contracts]
    C --> D[Start Frontend]
    D --> E[Test on localhost:3000]
    
    F[Make Changes] --> G[Recompile Contracts]
    G --> H[Redeploy Contracts]
    H --> I[Test Changes]
```

### **Deployment Process**

```mermaid
graph TD
    A[Prepare for Deployment] --> B[Switch to Sepolia]
    B --> C[Configure Environment]
    C --> D[Fund Account from Faucet]
    D --> E[Deploy Contracts]
    E --> F[Update Frontend Config]
    F --> G[Test on Testnet]
```

## 📁 **Project Structure**

```
challenge-4-dex/
├── packages/
│   ├── snfoundry/           # Smart contracts & deployment
│   │   ├── contracts/
│   │   │   └── src/
│   │   │       ├── DEX.cairo
│   │   │       └── Balloons.cairo
│   │   ├── scripts-ts/      # Deployment scripts
│   │   └── .env             # Environment variables
│   └── nextjs/              # Frontend application
│       ├── app/
│       │   ├── dex/         # DEX interface
│       │   └── events/      # Transaction history
│       ├── contracts/       # Deployed contract addresses
│       └── scaffold.config.ts
├── notes/                   # Checkpoint documentation
├── todos/                   # Testing guides
├── docker-compose.yml       # Devnet configuration
└── Makefile                 # Development commands
```

## 🔧 **Key Commands**

### **Development**
```bash
# Start local devnet
make start-devnet

# Deploy contracts
make deploy

# Start frontend
make start

# Stop devnet
make stop-devnet
```

### **Testing**
```bash
# Run tests
yarn test

# Compile contracts
yarn compile
```

### **Deployment**
```bash
# Deploy to Sepolia
yarn deploy --network sepolia

# Deploy to Mainnet
yarn deploy --network mainnet
```

## 🌍 **Network Configuration**

### **Supported Networks**

| Network | RPC URL | Chain ID | Purpose |
|---------|---------|----------|---------|
| Devnet | http://127.0.0.1:5050 | Local | Development |
| Sepolia | https://starknet-sepolia.public.blastapi.io/rpc/v0_8 | SN_SEPOLIA | Testing |
| Mainnet | https://starknet-mainnet.public.blastapi.io/rpc/v0_8 | SN_MAIN | Production |

### **Environment Variables**

```bash
# Sepolia Configuration
PRIVATE_KEY_SEPOLIA=your_private_key
ACCOUNT_ADDRESS_SEPOLIA=your_account_address
RPC_URL_SEPOLIA=https://starknet-sepolia.public.blastapi.io/rpc/v0_8
```

## 🧪 **Testing Strategy**

### **Contract Testing**
- **Unit Tests**: Individual function testing
- **Integration Tests**: Contract interaction testing
- **Edge Cases**: Boundary condition testing

### **Frontend Testing**
- **UI Components**: React component testing
- **Wallet Integration**: Connection testing
- **Transaction Flow**: End-to-end testing

### **Network Testing**
- **Devnet**: Local development testing
- **Sepolia**: Testnet deployment testing
- **Mainnet**: Production deployment testing

## 📚 **Resources**

### **Documentation**
- [Starknet Documentation](https://docs.starknet.io/)
- [Cairo Book](https://book.cairo-lang.org/)
- [Scaffold-Stark](https://github.com/scaffold-stark/scaffold-stark)

### **Tools**
- [Starkli CLI](https://book.starkli.rs/)
- [Argent X Wallet](https://www.argent.xyz/)
- [Braavos Wallet](https://braavos.app/)

### **Faucets**
- [Starknet Sepolia Faucet](https://starknet-faucet.publicgoods.xyz/)
- [Alchemy Faucet](https://www.alchemy.com/faucets/starknet-sepolia)
- [Blast Faucet](https://blastapi.io/faucets/starknet-sepolia-eth)

## 🎯 **Checkpoints Completed**

- ✅ **Checkpoint 1**: Project structure & contract deployment
- ✅ **Checkpoint 2**: Contract initialization & token setup
- ✅ **Checkpoint 3**: Price calculation & AMM logic
- ✅ **Checkpoint 4**: Trading functions (swap)
- ✅ **Checkpoint 5**: Liquidity functions (deposit/withdraw)
- ✅ **Checkpoint 6**: Frontend UI & slippage visualization
- ✅ **Checkpoint 7**: Sepolia testnet deployment

## 🚀 **Getting Started**

1. **Clone repository**
2. **Install dependencies**: `yarn install`
3. **Start devnet**: `make start-devnet`
4. **Deploy contracts**: `make deploy`
5. **Start frontend**: `make start`
6. **Open browser**: http://localhost:3000

## 📄 **License**

MIT License - See LICENSE file for details.
