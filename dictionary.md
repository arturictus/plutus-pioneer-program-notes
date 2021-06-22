# Dictionary

- get tx info:

```haskell
info = scriptContextTxInfo ctx
```

- signed by?

```haskell
txSignedBy info $ [PubKeyHash]
```

- get value of asset in output

```haskell
ownOutput :: TxOut
    ownOutput = case getContinuingOutputs ctx of
        [o] -> o
        _   -> traceError "expected exactly one oracle output"

assetClassValueOf (txOutValue ownOutput) (AssetClass ([tokenSymbol], [tokenName])
```

- extract specific datum, example with integer

```haskell
{-# INLINABLE oracleValue #-} -- inlinable for validator
oracleValue :: TxOut -> (DatumHash -> Maybe Datum) -> Maybe Integer
oracleValue o f = do
    dh      <- txOutDatum o
    Datum d <- f dh
    PlutusTx.fromData d

```

- get the public key executing the transaction

```haskell
getPkh = do
    pkh <- pubKeyHash <$> Contract.ownPubKey
```

- depositing into the script

```haskell
offerSwap :: forall w s. HasBlockchainActions s => Oracle -> Integer -> Contract w s Text ()
offerSwap oracle amt = do
    pkh <- pubKeyHash <$> Contract.ownPubKey
    let tx = Constraints.mustPayToTheScript pkh $ Ada.lovelaceValueOf amt
    ledgerTx <- submitTxConstraints (swapInst oracle) tx -- see we are adding the parameters for the validator [oracle]
    awaitTxConfirmed $ txId ledgerTx
    logInfo @String $ "offered " ++ show amt ++ " lovelace for swap"
```
