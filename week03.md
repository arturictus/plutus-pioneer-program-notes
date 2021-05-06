# Week 1

**clone repo:**

```
git clone git@github.com:input-output-hk/plutus-pioneer-program.git
```

**cabal build:**

```
cd code /week01
cabal build
```

### Run playground

#### Run in docker (recommended)

```
git clone git@github.com:arturictus/ppp.git
cd ppp
docker-compose -f docker-compose-week03.yml up
```

go to: https://localhost:8009/

### Updated Plutus, Changes in isData.hs

```diff
- mkValidator :: () -> MySillyRedeemer -> ValidatorCtx -> Bool
+ mkValidator :: () -> MySillyRedeemer -> ScriptContext -> Bool
```

```diff
- valHash :: Ledger.ValidatorHash
- valHash = Scripts.validatorHash validator

- scrAddress = ScriptAddress valHash
+ scrAddress = scriptAddress validator
```

### Definition of ScriptContext

https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Contexts.hs#L88-L116

### Interval helpers

https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Interval.hs

examples:

```haskell
import Plutus.V1.Ledger.Slot
import Plutus.V1.Ledger.Interval
-- initializing inst
Slot 3
3 :: Slot
interval (Slot 3) 10
-- member
member 5 $ interval (Slot 3) 10
-- => True
member 11 $ interval (Slot 3) 10
-- => False

-- initialize `to`
member 100 $ to (Slot 100)
-- => True
contains (to $ Slot 100) $ interval 30 50
-- => True
contains (to $ Slot 100) $ interval 30 110
-- => False
overlaps (to $ Slot 100) $ interval 30 110
-- => True
overlaps (to $ Slot 100) $ interval 101 110
-- => False
```

### Get Wallet pubKeyHash

```
cabal repl
```

```haskell
import Wallet.Emulator
import Ledger

pubKeyHash $ walletPubKey $ Wallet 1
-- => 21fe31dfa154a261626bf854046fd2271b7bed4b6abe45aa58877ef47f9721b9
pubKeyHash $ walletPubKey $ Wallet 2
-- => 39f713d0a644253f04529421b9f51b9b08979d08295959c4f3990ee617f5139f
```

### Parameterized script

**Lift class:**

https://github.com/input-output-hk/plutus/blob/master/plutus-tx/src/PlutusTx/Lift.hs

**Diff to parameterized script:**

```diff
34c35
- data VestingDatum = VestingDatum
---
+ data VestingParam = VestingParam
39c40,41
- PlutusTx.unstableMakeIsData ''VestingDatum
---
+ PlutusTx.unstableMakeIsData ''VestingParam
+ PlutusTx.makeLift ''VestingParam
42,43c44,45
- mkValidator :: VestingDatum -> () -> ScriptContext -> Bool
- mkValidator dat () ctx =
---
+ mkValidator :: VestingParam -> () -> () -> ScriptContext -> Bool
+ mkValidator p () () ctx =
51c53
-     checkSig = beneficiary dat `elem` txInfoSignatories info
---
+     checkSig = beneficiary p `elem` txInfoSignatories info
54c56
-     checkDeadline = from (deadline dat) `contains` txInfoValidRange info
---
+     checkDeadline = from (deadline p) `contains` txInfoValidRange info
58c60
-     type instance DatumType Vesting = VestingDatum
---
+     type instance DatumType Vesting = ()
61,63c63,65
- inst :: Scripts.ScriptInstance Vesting
- inst = Scripts.validator @Vesting
-     $$(PlutusTx.compile [|| mkValidator ||])
---
+ inst :: VestingParam -> Scripts.ScriptInstance Vesting
+ inst p = Scripts.validator @Vesting
+     ($$(PlutusTx.compile [|| mkValidator ||]) `PlutusTx.applyCode` PlutusTx.liftCode p)
66c68
-     wrap = Scripts.wrapValidator @VestingDatum @()
---
+     wrap = Scripts.wrapValidator @() @()
68,69c70,71
- validator :: Validator
- validator = Scripts.validatorScript inst
---
+ validator :: VestingParam -> Validator
+ validator = Scripts.validatorScript . inst
71,72c73,74
- scrAddress :: Ledger.Address
- scrAddress = scriptAddress validator
---
+ scrAddress :: VestingParam -> Ledger.Address
+ scrAddress = scriptAddress . validator
83c85
-         .\/ Endpoint "grab" ()
---
+         .\/ Endpoint "grab" Slot
87c89
-     let dat = VestingDatum
---
+     let p  = VestingParam
91,92c93,94
-         tx  = mustPayToTheScript dat $ Ada.lovelaceValueOf $ gpAmount gp
-     ledgerTx <- submitTxConstraints inst tx
---
+         tx = mustPayToTheScript () $ Ada.lovelaceValueOf $ gpAmount gp
+     ledgerTx <- submitTxConstraints (inst p) tx
99,100c101,102
- grab :: forall w s e. (HasBlockchainActions s, AsContractError e) => Contract w s e ()
- grab = do
---
+ grab :: forall w s e. (HasBlockchainActions s, AsContractError e) => Slot -> Contract w s e ()
+ grab d = do
103,105c105,106
-     utxos <- Map.filter (isSuitable pkh now) <$> utxoAt scrAddress
-     if Map.null utxos
-         then logInfo @String $ "no gifts available"
---
+     if now - d
+         then logInfo @String $ "too early"
107,124c108,124
-             let orefs   = fst <$> Map.toList utxos
-                 lookups = Constraints.unspentOutputs utxos  <>
-                           Constraints.otherScript validator
-                 tx :: TxConstraints Void Void
-                 tx      = mconcat [mustSpendScriptOutput oref $ Redeemer $ PlutusTx.toData () | oref <- orefs] <>
-                           mustValidateIn (from now)
-             ledgerTx <- submitTxConstraintsWith @Void lookups tx
-             void $ awaitTxConfirmed $ txId ledgerTx
-             logInfo @String $ "collected gifts"
-   where
-     isSuitable :: PubKeyHash -> Slot -> TxOutTx -> Bool
-     isSuitable pkh now o = case txOutDatumHash $ txOutTxOut o of
-         Nothing -> False
-         Just h  -> case Map.lookup h $ txData $ txOutTxTx o of
-             Nothing        -> False
-             Just (Datum e) -> case PlutusTx.fromData e of
-                 Nothing -> False
-                 Just d  -> beneficiary d == pkh && deadline d <= now
---
+             let p = VestingParam
+                         { beneficiary = pkh
+                         , deadline    = d
+                         }
+             utxos <- utxoAt $ scrAddress p
+             if Map.null utxos
+                 then logInfo @String $ "no gifts available"
+                 else do
+                     let orefs   = fst <$> Map.toList utxos
+                         lookups = Constraints.unspentOutputs utxos      <>
+                                   Constraints.otherScript (validator p)
+                         tx :: TxConstraints Void Void
+                         tx      = mconcat [mustSpendScriptOutput oref $ Redeemer $ PlutusTx.toData () | oref <- orefs] <>
+                                   mustValidateIn (from now)
+                     ledgerTx <- submitTxConstraintsWith @Void lookups tx
+                     void $ awaitTxConfirmed $ txId ledgerTx
+                     logInfo @String $ "collected gifts"
130c130
-     grab' = endpoint @"grab" >>  grab
---
+     grab' = endpoint @"grab" >>= grab
```
