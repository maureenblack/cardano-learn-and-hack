# Smart Contract Specification for Mama Mbongo's DeFi Platform

## Overview
This document outlines the theoretical Plutus smart contract implementation for the micro-lending DeFi platform.

## Contract Architecture

### 1. Lending Pool Contract

**Purpose:** Manages the collective pool of ADA from lenders

**State:**
```haskell
data PoolDatum = PoolDatum
    { totalFunds :: Integer      -- Total ADA in pool (in lovelace)
    , availableFunds :: Integer  -- Available for lending
    , totalLenders :: Integer    -- Number of active lenders
    , interestRate :: Integer    -- Interest rate (basis points)
    }
```

**Actions:**
- `Deposit` - Add ADA to the lending pool
- `Withdraw` - Remove ADA from the pool (with restrictions)
- `UpdateInterestRate` - Governance action to adjust rates

### 2. Loan Contract

**Purpose:** Manages individual loan agreements

**State:**
```haskell
data LoanDatum = LoanDatum
    { borrower :: PubKeyHash     -- Borrower's address
    , principal :: Integer       -- Loan amount in lovelace
    , interestRate :: Integer    -- Interest rate (basis points)
    , startTime :: POSIXTime     -- Loan start timestamp
    , duration :: Integer        -- Loan duration in days
    , status :: LoanStatus       -- Current loan status
    }

data LoanStatus = Active | Repaid | Defaulted
```

**Actions:**
- `RequestLoan` - Create new loan request
- `ApproveLoan` - Approve and fund loan
- `RepayLoan` - Repay loan with interest
- `Liquidate` - Handle defaulted loans

## Validation Logic

### Deposit Validation
```haskell
validateDeposit :: PoolDatum -> Integer -> ScriptContext -> Bool
validateDeposit oldDatum depositAmount ctx =
    let newDatum = oldDatum { 
          totalFunds = totalFunds oldDatum + depositAmount,
          availableFunds = availableFunds oldDatum + depositAmount
        }
    in traceIfFalse "Invalid deposit amount" (depositAmount > 0) &&
       traceIfFalse "Pool datum not updated correctly" (checkDatumUpdate newDatum)
```

### Loan Request Validation
```haskell
validateLoanRequest :: PoolDatum -> LoanDatum -> ScriptContext -> Bool
validateLoanRequest poolDatum loanDatum ctx =
    let loanAmount = principal loanDatum
    in traceIfFalse "Insufficient pool funds" (availableFunds poolDatum >= loanAmount) &&
       traceIfFalse "Invalid loan amount" (loanAmount > 0 && loanAmount <= maxLoanAmount) &&
       traceIfFalse "Invalid duration" (duration loanDatum `elem` allowedDurations)
```

### Repayment Validation
```haskell
validateRepayment :: LoanDatum -> Integer -> ScriptContext -> Bool
validateRepayment loanDatum repaymentAmount ctx =
    let expectedRepayment = calculateRepayment loanDatum
    in traceIfFalse "Incorrect repayment amount" (repaymentAmount >= expectedRepayment) &&
       traceIfFalse "Loan already repaid" (status loanDatum == Active)
```

## Interest Calculation

```haskell
calculateRepayment :: LoanDatum -> Integer
calculateRepayment loan =
    let principalAmount = principal loan
        rate = interestRate loan
        interest = (principalAmount * rate) `div` 10000  -- basis points
    in principalAmount + interest
```

## Security Considerations

### 1. Collateral Management
- Require collateral for loans above certain threshold
- Implement liquidation mechanisms for defaulted loans
- Support multiple collateral types (ADA, native tokens)

### 2. Access Control
- Only pool participants can withdraw funds
- Borrower-specific loan management
- Admin functions for emergency situations

### 3. Time-based Logic
- Proper handling of loan durations
- Grace periods for repayments
- Automatic status updates based on time

## Deployment Parameters

```haskell
data DeploymentParams = DeploymentParams
    { minLoanAmount :: Integer      -- Minimum loan: 1 ADA
    , maxLoanAmount :: Integer      -- Maximum loan: 100 ADA
    , baseInterestRate :: Integer   -- Base rate: 500 (5%)
    , allowedDurations :: [Integer] -- [30, 60, 90] days
    , collateralRatio :: Integer    -- 150% collateral requirement
    }
```

## Transaction Examples

### Deposit Transaction
```
Inputs:
  - User's UTxO with ADA
  - Pool UTxO with current state

Outputs:
  - Updated Pool UTxO with increased funds
  - User's change UTxO

Datums:
  - Updated PoolDatum with new totals
```

### Loan Request Transaction
```
Inputs:
  - Pool UTxO with available funds
  - Borrower's collateral UTxO

Outputs:
  - Updated Pool UTxO with reduced available funds
  - New Loan UTxO with loan details
  - Borrower's UTxO with loan amount
  - Locked collateral UTxO

Datums:
  - Updated PoolDatum
  - New LoanDatum
```

## Testing Strategy

1. **Unit Tests**
   - Individual validator functions
   - Interest calculations
   - State transitions

2. **Integration Tests**
   - Full transaction flows
   - Multi-user scenarios
   - Edge cases and error conditions

3. **Property Tests**
   - Pool invariants (total funds consistency)
   - Interest calculation properties
   - Time-based logic verification

## Governance

Future enhancements could include:
- DAO governance for interest rates
- Voting mechanisms for platform upgrades
- Treasury management for platform fees
- Reputation system for borrowers

## Compliance Considerations

- KYC/AML integration points
- Regulatory reporting capabilities
- Geographic restrictions
- Maximum lending limits per jurisdiction
