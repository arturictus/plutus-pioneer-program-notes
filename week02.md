# Week 2

## Loading components in empty script

```haskell
{-# LANGUAGE DataKinds           #-}
{-# LANGUAGE FlexibleContexts    #-}
{-# LANGUAGE NoImplicitPrelude   #-}
{-# LANGUAGE ScopedTypeVariables #-}
{-# LANGUAGE TemplateHaskell     #-}
{-# LANGUAGE TypeApplications    #-}
{-# LANGUAGE TypeFamilies        #-}
{-# LANGUAGE TypeOperators       #-}

module Week02.Gift where

import           Control.Monad       hiding (fmap)
import           Data.Map            as Map
import           Data.Text           (Text)
import           Data.Void           (Void)
import           Plutus.Contract     hiding (when)
import           PlutusTx            (Data (..))
import qualified PlutusTx
import           PlutusTx.Prelude    hiding (Semigroup(..), unless)
import           Ledger              hiding (singleton)
import           Ledger.Constraints  as Constraints
import qualified Ledger.Scripts      as Scripts
import           Ledger.Ada          as Ada
import           Playground.Contract (printJson, printSchemas, ensureKnownCurrencies, stage)
import           Playground.TH       (mkKnownCurrencies, mkSchemaDefinitions)
import           Playground.Types    (KnownCurrency (..))
import           Prelude             (Semigroup (..))
import           Text.Printf         (printf)
```

## Validator

```haskell
validator datum, redeemer, context = ()
```

validator receives
datum, redeemer and context all of them are type `Data`

```haskell
module PlutusTx.Data (Data (..)) where
```

The validator return `unit :: ()` wich means nothing. The validator stops execution by throwing and `ERROR`.

```haskell
{-# INLINABLE mkValidator #-}
mkValidator :: Data -> Data -> Data -> ()
-- simplest validator ever
mkValidator _ _ _ = ()
validator :: Validator
validator = mkValidatorScript $$(PlutusTx.compile [|| mkValidator ||])
```

## Address validator

```haskell
valHash :: Ledger.ValidatorHash
```
