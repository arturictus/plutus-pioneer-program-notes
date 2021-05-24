# Week 5

### Value and Native Tokens

(Value definition)[https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Value.hs#L210-L214]
(Ada)[https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Ada.hs#L58-L63]

repl:

```haskell

import Plutus.V1.Ledger.Value
import Plutus.V1.Ledger.Ada
:set -XOverloadedStrings
adaSymbol -- empty bitstring
adaToken -- empty bitstring

lovelaceValueOf 123
-- => Value (Map [(,Map [("",123)])]) // observe how symbol and tokenName are empty

-- combine multiple values
lovelaceValueOf 123 <> lovelaceValueOf 10
-- => Value (Map [(,Map [("",133)])])

-- Build Value for different tokens
:t singleton
-- => singleton :: CurrencySymbol -> TokenName -> Integer -> Value
singleton "a8ff" "ABC" 7
-- => Value (Map [(a8ff,Map [("ABC",7)])])



-- combine values of different tokens
singleton "a8ff" "ABC" 7 <> lovelaceValueOf 42 <> singleton "a8ff" "XYZ" 100
-- => Value (Map [(,Map [("",42)]),(a8ff,Map [("ABC",7),("XYZ",100)])])


-- helper to get values
let v = singleton "a8ff" "ABC" 7 <> lovelaceValueOf 42 <> singleton "a8ff" "XYZ" 100
:t valueOf
-- => valueOf :: Value -> CurrencySymbol -> TokenName -> Integer
valueOf v "" "" -- ADA
-- => 42
 valueOf v "a8ff" "ABC"
-- => 7
 valueOf v "a8ff" "XYZ"
-- => 100
 valueOf v "a8ff" "abc"
-- => 0

:t flattenValue
-- => flattenValue :: Value -> [(CurrencySymbol, TokenName, Integer)]
flattenValue v
-- => [(a8ff,"ABC",7),(a8ff,"XYZ",100),(,"",42)]
```

### Validation Policy | Minting Script

Review context Datum

(Context)[https://github.com/input-output-hk/plutus/blob/master/plutus-ledger-api/src/Plutus/V1/Ledger/Contexts.hs#L88-L116]
