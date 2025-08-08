# YamPay Solution

This is an example solution for the YamPay challenge - a beginner-friendly Cardano wallet integration demo.

## Features Implemented

✅ **Basic Wallet Integration**
- Connect to Nami or Eternl wallets
- Display wallet balance in ADA
- Show wallet address (truncated for display)

✅ **Simple Market Interface**
- Display 3 types of yams with prices
- Interactive buy buttons
- Farmer's wallet address display

✅ **Transaction Simulation**
- Simulated transaction flow (for demo purposes)
- Success/error message handling
- Transaction ID generation

✅ **User Experience**
- Responsive design
- Beautiful gradient styling
- Clear status messages
- Disabled buttons until wallet connected

## How to Test

1. **Setup**
   - Open `index.html` in a web browser
   - Make sure you have a Cardano testnet wallet installed (Nami or Eternl)

2. **Get Test ADA**
   - Visit the [Cardano Testnet Faucet](https://docs.cardano.org/cardano-testnet/tools/faucet)
   - Request test ADA for your wallet

3. **Test the Application**
   - Click "Connect Wallet"
   - Authorize the connection in your wallet
   - View your wallet balance and address
   - Try clicking "Buy Now" on any yam

## Technical Implementation

### Wallet Detection
```javascript
if (typeof window.cardano !== 'undefined') {
    cardano = window.cardano;
    // Wallet is available
}
```

### Wallet Connection
```javascript
if (cardano.nami) {
    walletApi = await cardano.nami.enable();
} else if (cardano.eternl) {
    walletApi = await cardano.eternl.enable();
}
```

### Balance Display
```javascript
const balanceValue = await walletApi.getBalance();
const balance = parseInt(balanceValue) / 1000000; // Convert lovelace to ADA
```

## Note on Transactions

This demo includes **simulated transactions** for educational purposes. In a real implementation, you would:

1. Build a proper transaction using Cardano's transaction builder
2. Use `walletApi.submitTx()` to submit the transaction
3. Handle real transaction responses and confirmations

## Success Criteria Met

- ✅ Connect wallet to webpage
- ✅ Show wallet balance correctly  
- ✅ Create test transaction flow
- ✅ Add basic error messages
- ✅ **Bonus:** Nice CSS styling
- ✅ **Bonus:** Thank you messages after purchase
- ✅ **Bonus:** Friendly transaction display

## Files

- `index.html` - Complete single-file solution with HTML, CSS, and JavaScript
- `README.md` - This documentation

## Browser Compatibility

- Modern browsers with ES6+ support
- Requires Cardano wallet extension (Nami, Eternl, etc.)
- Best tested on Chrome/Firefox with wallet extensions
