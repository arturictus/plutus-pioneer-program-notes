# Week 6

---

`cabal build` does not work

steps to make it work:

- run `nix-shell`
- `cd` to week06 folder

---

## Oracle Core

_code/week06/src/Week06/Oracle/Core.hs_

### Onchain

**Oracle parameter:**

```haskell
data Oracle = Oracle
    { oSymbol   :: !CurrencySymbol -- NFT token symbol probing legitimacy of the oracle
    , oOperator :: !PubKeyHash -- pub key of the creator allowing access to update
    , oFee      :: !Integer -- fees for anyone using the oracle
    , oAsset    :: !AssetClass -- asset quote (for pair: ADA/USDT)
    } deriving (Show, Generic, FromJSON, ToJSON, Prelude.Eq, Prelude.Ord)
```

- **Extract the datum value from the txOut**

```haskell
{-# INLINABLE oracleValue #-}
oracleValue :: TxOut -> (DatumHash -> Maybe Datum) -> Maybe Integer
oracleValue o f = do
    dh      <- txOutDatum o -- [dh: datum hash] extract datum, could fail if no datum in the txOut
    Datum d <- f dh -- apply fun to extract the data
    PlutusTx.fromData d -- convert datum to Integer
```

**oracle validator:**

- `getContinuingOutputs`: returns all the outputs that go to the same address as the script I'm validating.

```haskell

```

### Offchain

#### startOracle

```haskell
data OracleParams = OracleParams
    { opFees   :: !Integer
    , opSymbol :: !CurrencySymbol -- of the NFT
    , opToken  :: !TokenName
    } deriving (Show, Generic, FromJSON, ToJSON)
```

- `forgeContract`:
  forges the amount we want given the a publickey and a list of names, returns a `OneShotCurrency` contract that will create the tokens

```haskell
(forgeContract pkh [(oracleTokenName, 1)] :: Contract w s CurrencyError OneShotCurrency)
```

returns:

```haskell
Contract w s
-- w = arbitrary writer
-- s = arbitrary schema as long as we have `HasBlockchainActions`
```

expected return:

```haskell
startOracle :: forall w s. HasBlockchainActions s => OracleParams -> Contract w s Text Oracle
```

workaround to convert to expected error:

```haskell
mapError (pack . show) (forgeContract pkh [(oracleTokenName, 1)] :: Contract w s CurrencyError OneShotCurrency)
```

#### updateOracle

## Swap

#### RUN app

1rst shell:

```
docker-compose -f code/week06/docker-compose.yml run nix-shell
cd /app
cabal build
cabal run oracle-pab
```

2ond shell:

```
docker exec -it [CONTAINER ID] nix-shell
cd /app
cabal run oracle-client
```

3rd shell, swap contract

```
docker exec -it [CONTAINER ID] nix-shell
cd /app
cabal run swap-client -- 2
```

4rd shell, swap contract

```
docker exec -it [CONTAINER ID] nix-shell
cd /app
cabal run swap-client -- 3
```
