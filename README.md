# file structure in augur-core/develop
```
Key : description
+   : added file that didn't exist in master
-   : removed file that used to exist in master
->  : existing file that was moved to a new location
<-  : a file that was removed from this location and has been moved
*   : a file that is exists in this location for both master and develop
```
```
src/
  * repContract.se
src/data_api/
  * backstops.se
  * branches.se
  <- cash.se
  -> compositeGetters.se
  * consensusData.se
  * events.se
  * expiringEvents.se
  + float.se
  * fxpFunctions.se
  * info.se
  * markets.se
  + mutex.se
  + periodStage.se
  * refund.se
  * register.se
  * reporting.se
  + reportingThreshold.se
  - topics.se
  * trades.se
src/functions/
  - buy&sellShares.se
  + bidAndAsk.se
  -> cash.se
  + claimMarketProceeds.se
  * closeMarket.se
  * collectFees.se
  * completeSets.se
  * consensus.se
  * createBranch.se
  * createMarket.se
  + eventHelpers.se
  * eventResolution.se
  * faucets.se
  * forking.se
  * forkPenalize.se
  * logReturn.se
  * makeReports.se
  - payout.se
  + offChainTrades.se
  + oneWinningOutcomePayouts.se
  * penalizationCatchup.se
  * penalizeNotEnoughReports.se
  * proportionCorrect.se
  + redistributeRep.se
  + repChange.se
  * roundTwo.se
  * roundTwoPenalize.se
  * sendReputation.se
  + shareTokens.se
  * slashRep.se
  * trade.se
  + tradeAvailableOrders.se
  + twoWinningOutcomePayouts.se
  + wallet.se
```

## src/data_api/backstops.se
```
Key : description
!   : Modified function
-   : removed function
+   : added function
->  : function moved FROM another contract
<-  : function moved TO another contract
```
*Any function not explicitly mentioned is unchanged from it's current master iteration.*

#### Data Structure of backstops Contract:
```
data roundTwo[<eventId>](roundTwo, originalVotePeriod, originalOutcome,
originalEthicality, final, bondPoster, bondReturned, bondPaid,
refund, disputedOverEthics)

data forking[<eventId>](bondPoster, bondAmount, forkedOverEthicality, bondPaid, originalBranch, moved)

data resolved[<branch>][<forkPeriod>]
```

The roundTwo array now contains a new value, `disputedOverEthics`.   `disputedOverEthics` is a boolean indicating if the `event` was disputed over it's ethicality.

#### backstops method changes, additions, and removals:
```
+ getDisputedOverEthics(event):
```
getDisputedOverEthics returns the disputedOverEthics bool for an event
```
+ setDisputedOverEthics(event):
```
setDisputedOverEthics sets the passed in event's disputedOverEthics bool to 1

## src/data_api/branches.se
```
Key : description
!   : Modified function
-   : removed function
+   : added function
->  : function moved FROM another contract
<-  : function moved TO another contract
```
*Any function not explicitly mentioned is unchanged from it's current master iteration.*

#### Data Structure of branches Contract:
```
data Branches[<branch>](currentVotePeriod, periodLength, markets[<index>],
numMarkets, fxpMinTradingFee, balance[<period>][<currency>], creationDate, oracleOnly, parentPeriod, baseReporters, forkPeriod, eventForkedOver, parent, contract[<currency>], numCurrencies, currencies[<index>](rate, rateContract, contract), currencyToIndex[<currency>], mostRecentChild, currencyActive[<currency>], forkTime)

data branchList[<index>]
data branchListCount
```
With the update to the contracts planned in develop, we are switching to using "currency". When you see `currency` in the contracts that value is an address for the token allowed to be used on this branch. The balance array has changed to become multi-dimensional and now takes in both a period and the currency to work with balance since we have a new concept of multiple currencies allowed. So it's not enough to just ask for the balance in a period, you need to know what denomination of currency we want to get the balance of as an example. `minTradingFee` has become `fxpMinTradingFee`, which indicates that this value should be in fix point. The `contract[<currency>]` array was added and holds the `wallet` addresses for the indexed `currency` address. `numCurrencies` is a simple count of the number of currencies currently allowed on the `branch`.

The `currencies[<index>](rate, rateContract, contract)` array was added and is a simple 0 indexed list. Inside each index you will find 3 values, the `contract` which is the `currency` address, the `rate` which is a fixed exchange rate and the `rateContract` which is a contract with rates for the currency to be exchanged with `Eth` denominated in `Wei`. `currencyToIndex[<currency>]` is a reverse mapping of currencies to their indices. `mostRecentChild` is the most recent child of a `branch`, `currencyActive[<currency>]` contains booleans indicating wether a currency is allowed to be used to create a new market or event. `forkTime` was also added as a timestamp for when the `branch` was forked.

#### branches method changes, additions, and removals:
*note: when a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*
```
- initDefaultBranch():

! setInitialBalance(branch, period, balance):
```
setInitialBalance Changed:
`setInitialBalance(branch, period, balance, currency)`
`setInitialBalance` now also takes in `currency` as well in order to indicate
which currency we are setting the balance of.

```
! getInitialBalance(branch, period):
```
getInitialBalance Changed:
`getInitialBalance(branch, period, currency)`
`getInitialBalance` also now takes `currency` in order to determine which currency you want to get the initial balance of.
```
- getMarketsInBranch(branch):

- getSomeMarketsInBranch(branch, initial, last):

! initializeBranch(ID, currentVotePeriod, periodLength, minTradingFee, oracleOnly, parentPeriod, parent):
```
initializeBranch Changed:
`initializeBranch(ID, currentVotePeriod, periodLength, fxpMinTradingFee,
 oracleOnly, parentPeriod, parent, contract, wallet, mostRecentChild):`
A few things have changed with `initializeBranch`, `minTradingFee` has become the more explicit `fxpMinTradingFee` and should be passed as a fixed point value. `contract` was added and is expecting a `currency` address as it's value, `wallet` was added and is expected to be the `wallet` address used to hold the `currency` passed as `contract`. `mostRecentChild` is the most recent child of the `branch`.

```
+ getForkTime(branch)
```
`getForkTime` was added to return the fork timestamp which is set when `setForkPeriod` is called. If the branch hasn't been forked then this will return 0.


*Please ignore everything below this line as not part of the change log, simply some notes for upcoming updates to the change log.*
# Ignore the below please.
updateNumCurrencies(branch, num):
addCurrency(branch, currency, rate, rateContract):
disableCurrency(branch, currency):
reactivateCurrency(branch, currency)
replaceCurrency(branch, oldCurrencyIndex, newCurrency, newRate,
removeLastCurrency(branch):
updateCurrencyRate(branch, currency, rate, rateContract):
getCurrencyRate(branch, currency):
getCurrency(branch, index):
getCurrencyByContract(branch, currency):
getWallet(branch, currency):
getNumCurrencies(branch):
getCurrencyActive(branch, currency):
setInitialBalance(branch, period, balance, currency):
getInitialBalance(branch, period, currency):
getMarketIDsInBranch(branch, initial, last):
getBranchesStartingAt(index):
getMostRecentChild(ID):
setMostRecentChild(parent, child):
initializeBranch(ID, currentVotePeriod, periodLength, fxpMinTradingFee, oracleOnly, parentPeriod, parent, contract, wallet, mostRecentChild):
getForkTime(branch):
