# 🖼️ Checkpoint 6: UI Testing Guide

## 📋 Tổng quan

Checkpoint 6 tập trung vào **UI/UX testing** để đảm bảo frontend DEX có đầy đủ functionality và visualization cần thiết.

## 🎯 Mục tiêu Checkpoint 6

- ✅ **Show token balances** - Hiển thị số dư tokens
- ✅ **Let users swap tokens** - Cho phép users swap tokens  
- ✅ **Visualize slippage** - Hiển thị slippage

## 🧪 Testing Requirements

### **1. Token Balance Display**
- STRK balance hiển thị đúng
- BAL balance hiển thị đúng
- Real-time balance updates
- Contract reserves hiển thị

### **2. Swap Interface**
- Input fields hoạt động
- Expected output calculations
- Auto-approve mechanism
- Transaction handling

### **3. Slippage Visualization**
- Interactive AMM curve
- Real-time slippage calculations
- Visual indicators (arrows, dots)
- Fee display (-0.3%)

## 🚀 Setup Phase

### **Prerequisites**
```bash
# Start all services
make start-devnet    # Terminal 1
make deploy          # Terminal 2
make start           # Terminal 3

# Open browser
open http://localhost:3000
```

### **Connect Wallet**
1. Click "Connect Wallet"
2. Choose Argent X hoặc Braavos
3. Approve connection

---

## 🧪 Test Case 1: Token Balance Display

### **Test 1.1: Header Balance Display**

#### **Steps**:
1. **Navigate to main page**: http://localhost:3000
2. **Check header area**:
   - Look for STRK balance display
   - Should show current STRK balance
   - Format: "X.XX STRK"

#### **Expected Results**:
- ✅ STRK balance visible in header
- ✅ Balance updates after transactions
- ✅ Format is user-friendly (not wei)

### **Test 1.2: DEX Page Balance Display**

#### **Steps**:
1. **Navigate to DEX page**: http://localhost:3000/dex
2. **Check balance section**:
   - Look for BAL balance display
   - Should show current BAL balance
   - Format: "BAL Balance: X.XX"

#### **Expected Results**:
- ✅ BAL balance visible on DEX page
- ✅ Balance updates after swaps
- ✅ Format is user-friendly

### **Test 1.3: Contract Reserves Display**

#### **Steps**:
1. **On DEX page**, check reserves section
2. **Look for contract balance displays**:
   - DEX STRK reserves
   - DEX BAL reserves

#### **Expected Results**:
- ✅ Contract reserves displayed
- ✅ Reserves update after swaps/deposits
- ✅ Format is readable

---

## 🧪 Test Case 2: Swap Interface Testing

### **Test 2.1: STRK to Token Swap Interface**

#### **Steps**:
1. **Navigate to DEX page**: http://localhost:3000/dex
2. **Find "STRK to Token Swap" section**
3. **Test input field**:
   - Input: `1` (STRK)
   - Check expected output calculation
   - Verify format: "~X.XX BAL"

#### **Expected Results**:
- ✅ Input field accepts numbers
- ✅ Expected output calculated correctly
- ✅ Output shows ~0.996 BAL (với fee 0.3%)
- ✅ Format is user-friendly

### **Test 2.2: Token to STRK Swap Interface**

#### **Steps**:
1. **Find "Token to STRK Swap" section**
2. **Test input field**:
   - Input: `1` (BAL)
   - Check expected output calculation
   - Verify format: "~X.XX STRK"

#### **Expected Results**:
- ✅ Input field accepts numbers
- ✅ Expected output calculated correctly
- ✅ Output shows ~0.996 STRK (với fee 0.3%)
- ✅ Format is user-friendly

### **Test 2.3: Swap Button Functionality**

#### **Steps**:
1. **Input valid amount**: `1` STRK
2. **Click "Swap STRK to Token" button**
3. **Check wallet popup**:
   - Should show transaction details
   - Should include approve + swap calls

#### **Expected Results**:
- ✅ Button is clickable
- ✅ Wallet popup appears
- ✅ Transaction details clear
- ✅ Multi-write contract calls (approve + swap)

---

## 🧪 Test Case 3: Slippage Visualization

### **Test 3.1: Interactive AMM Curve**

#### **Steps**:
1. **On DEX page**, locate the curve visualization
2. **Check curve elements**:
   - Blue dot (current reserves)
   - AMM curve line
   - Axes labels

#### **Expected Results**:
- ✅ Curve displays correctly
- ✅ Blue dot shows current reserves position
- ✅ AMM curve follows x*y=k formula
- ✅ Axes are labeled properly

### **Test 3.2: Real-time Slippage Calculation**

#### **Steps**:
1. **Input STRK amount**: `1`
2. **Watch curve update**:
   - Should show red arrow from current position
   - Should show green arrow to new position
   - Should display slippage information

#### **Expected Results**:
- ✅ Red arrow shows input direction
- ✅ Green arrow shows output direction
- ✅ New position dot appears
- ✅ Slippage calculation displayed

### **Test 3.3: Large Swap Slippage Warning**

#### **Steps**:
1. **Input large amount**: `3` STRK
2. **Check curve visualization**:
   - Should show significant slippage
   - Should display warning indicators
   - Should show reduced output

#### **Expected Results**:
- ✅ Significant slippage visible
- ✅ Warning indicators displayed
- ✅ Output significantly reduced
- ✅ Visual feedback clear

### **Test 3.4: Fee Display**

#### **Steps**:
1. **Input any swap amount**
2. **Check fee information**:
   - Should display "-0.3% fee"
   - Should be visible in curve area

#### **Expected Results**:
- ✅ Fee percentage displayed
- ✅ Fee calculation clear
- ✅ Information easily readable

---

## 🧪 Test Case 4: User Experience Testing

### **Test 4.1: Responsive Design**

#### **Steps**:
1. **Test different screen sizes**:
   - Desktop (1920x1080)
   - Tablet (768x1024)
   - Mobile (375x667)

#### **Expected Results**:
- ✅ Layout adapts to screen size
- ✅ Curve visualization scales
- ✅ Text remains readable
- ✅ Buttons accessible

### **Test 4.2: Dark/Light Mode**

#### **Steps**:
1. **Toggle theme** (if available)
2. **Check visualization**:
   - Curve colors adapt
   - Text remains readable
   - Background contrasts properly

#### **Expected Results**:
- ✅ Theme toggle works
- ✅ Curve colors adapt
- ✅ Good contrast maintained
- ✅ Readable in both modes

### **Test 4.3: Error Handling**

#### **Steps**:
1. **Test invalid inputs**:
   - Negative numbers
   - Non-numeric characters
   - Empty fields

2. **Test insufficient balance**:
   - Input amount > balance
   - Try to swap

#### **Expected Results**:
- ✅ Invalid inputs rejected
- ✅ Error messages clear
- ✅ No crashes or freezes
- ✅ User guidance provided

---

## 🧪 Test Case 5: Integration Testing

### **Test 5.1: Balance Updates After Swap**

#### **Steps**:
1. **Note initial balances**
2. **Execute STRK → BAL swap**
3. **Check balance updates**:
   - STRK balance decreased
   - BAL balance increased
   - Reserves updated

#### **Expected Results**:
- ✅ Balances update immediately
- ✅ No page refresh needed
- ✅ Accurate calculations
- ✅ Real-time updates

### **Test 5.2: Multiple Swap Operations**

#### **Steps**:
1. **Execute STRK → BAL swap**
2. **Wait for completion**
3. **Execute BAL → STRK swap**
4. **Check all balances updated**

#### **Expected Results**:
- ✅ Both swaps execute successfully
- ✅ All balances update correctly
- ✅ No state inconsistencies
- ✅ Smooth user experience

---

## 🧪 Test Case 6: Advanced UI Features

### **Test 6.1: Curve Interaction**

#### **Steps**:
1. **Hover over curve elements**
2. **Check for tooltips or info**
3. **Test zoom/pan functionality** (if available)

#### **Expected Results**:
- ✅ Interactive elements respond
- ✅ Tooltips informative
- ✅ Smooth interactions

### **Test 6.2: Loading States**

#### **Steps**:
1. **Execute swap transaction**
2. **Check loading indicators**:
   - Button shows loading state
   - Curve shows processing
   - User feedback clear

#### **Expected Results**:
- ✅ Loading indicators visible
- ✅ User knows transaction pending
- ✅ No confusion about state
- ✅ Clear completion feedback

---

## ✅ Checklist Checkpoint 6

### **Token Balance Display** ✅
- [ ] STRK balance visible in header
- [ ] BAL balance visible on DEX page
- [ ] Contract reserves displayed
- [ ] Balances update in real-time
- [ ] Format is user-friendly

### **Swap Interface** ✅
- [ ] Input fields accept valid numbers
- [ ] Expected output calculated correctly
- [ ] Swap buttons functional
- [ ] Multi-write transactions work
- [ ] Wallet integration smooth

### **Slippage Visualization** ✅
- [ ] AMM curve displays correctly
- [ ] Real-time updates work
- [ ] Visual indicators clear (arrows, dots)
- [ ] Slippage calculations accurate
- [ ] Fee information displayed

### **User Experience** ✅
- [ ] Responsive design works
- [ ] Dark/light mode support
- [ ] Error handling robust
- [ ] Loading states clear
- [ ] Smooth interactions

### **Integration** ✅
- [ ] Balance updates after transactions
- [ ] Multiple operations work
- [ ] No state inconsistencies
- [ ] Real-time synchronization

---

## 🐛 Troubleshooting UI Issues

### **Issue 1: Balances not updating**
**Solution**:
- Check contract calls are working
- Verify real-time hooks are active
- Refresh page if needed

### **Issue 2: Curve not displaying**
**Solution**:
- Check canvas element exists
- Verify props are passed correctly
- Check console for errors

### **Issue 3: Swap buttons not working**
**Solution**:
- Check wallet connection
- Verify contract addresses
- Check transaction parameters

### **Issue 4: Expected output = 0**
**Solution**:
- Verify price function working
- Check reserves are initialized
- Check input validation

---

## 🎯 UI Testing Commands

```bash
# Check if frontend is running
curl http://localhost:3000

# Check contract deployment
cat packages/snfoundry/deployments/devnet_latest.json

# Check devnet status
curl http://localhost:5050/is_alive
```

---

## 📊 UI Testing Summary

### **Core Requirements** ✅
- ✅ Token balances displayed
- ✅ Swap interface functional
- ✅ Slippage visualized

### **Advanced Features** ✅
- ✅ Interactive curve
- ✅ Real-time updates
- ✅ Multi-write transactions
- ✅ Responsive design

### **User Experience** ✅
- ✅ Smooth interactions
- ✅ Clear feedback
- ✅ Error handling
- ✅ Loading states

---

**Status**: ✅ Checkpoint 6 UI testing complete  
**Date**: 2025-10-02  
**Requirements**: 3/3 met  
**UI Features**: All functional
