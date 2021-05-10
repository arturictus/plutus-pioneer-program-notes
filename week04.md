# Week 4

## Momads explantaion

Review files:

- Maybe.hs
- Either.hs
- Writer.hs
- Monad.hs

## Emulator

Internal plutus definitions:

- (runEmulatorTrace)[https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Trace/Emulator.hs#L226-L235]
- (EmulatorConfig)[https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Wallet/Emulator/Stream.hs#L135-L138]
- (InitialChainState)[https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Wallet/Emulator/Stream.hs#L140]
- (InitialDistribution)[https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Contract/Trace.hs#L115]
- (defautlDist)[https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Contract/Trace.hs#L248]

---

Run Emulator on repl with better output:

(runEmulatorTraceIO)[https://github.com/input-output-hk/plutus/blob/master/plutus-contract/src/Plutus/Trace/Emulator.hs#L269-L272]

Repl:

```haskell
import Plutus.Trace.Emulator
import Plutus.Contract.Trace

defaultDist
-- => fromList [(Wallet 1,Value (Map [(,Map [("",100000000)])])),(Wallet 2, ...
defaultDistFor [Wallet 1]
-- => fromList [(Wallet 1,Value (Map [(,Map [("",100000000)])]))]
:t runEmulatorTrace
-- runEmulatorTrace
--   :: EmulatorConfig
--      -> EmulatorTrace ()
--      -> ([Wallet.Emulator.MultiAgent.EmulatorEvent], Maybe EmulatorErr,
--          Wallet.Emulator.MultiAgent.EmulatorState)
:i EmulatorConfig
-- type EmulatorConfig :: *
-- data EmulatorConfig
--   = EmulatorConfig {_initialChainState :: Wallet.Emulator.Stream.InitialChainState}
--   	-- Defined in ‘Wallet.Emulator.Stream’
-- instance Eq EmulatorConfig -- Defined in ‘Wallet.Emulator.Stream’
-- instance Show EmulatorConfig -- Defined in ‘Wallet.Emulator.Stream’

import Wallet.Emulator.Stream

:i InitialChainState
-- type InitialChainState :: *
-- type InitialChainState =
--   Either InitialDistribution Ledger.Blockchain.Block
--   	-- Defined in ‘Wallet.Emulator.Stream’

:t EmulatorConfig $ Left defaultDist
-- EmulatorConfig $ Left defaultDist :: EmulatorConfig

runEmulatorTrace (EmulatorConfig $ Left defaultDist) $ return ()
-- lot of data
-- ... txValidRange = Interval {ivFrom = LowerBound NegInf True, ivTo = UpperBound PosInf True}, txForgeScripts = fromList [], txSignatures = fromList [], txData = fromList []}, ciTxId = af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8}]})]}}, _signingProcess = SigningProcess <...>})], _emulatorLog = []})

runEmulatorTraceIO $ return ()
-- Slot 00001: TxnValidate af5e6d25b5ecb26185289a03d50786b7ac4425b21849143ed7e18bcd70dc4db8
-- Slot 00001: SlotAdd Slot 1
-- Slot 00002: SlotAdd Slot 2
-- Final balances
-- Wallet 1:
--     {, ""}: 100000000
-- Wallet 2:
--     {, ""}: 100000000
-- Wallet 3:
--     {, ""}: 100000000
-- Wallet 4:
--     ...


```
