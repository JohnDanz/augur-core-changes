# file structure in augur-core/develop
```
Key : description
*   : a file that is exists in this location for both master and develop
+   : added file that didn't exist in master
-   : removed file that used to exist in master
->  : existing file that was moved to a new location
<-  : a file that was removed from this location and still exists
      but has been moved
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
  <- compositeGetters.se
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

### Data Structure of backstops Contract:
```
data roundTwo[<eventId>](roundTwo, originalVotePeriod, originalOutcome,
originalEthicality, final, bondPoster, bondReturned, bondPaid,
refund, disputedOverEthics)

data forking[<eventId>](bondPoster, bondAmount, forkedOverEthicality, bondPaid, originalBranch, moved)

data resolved[<branch>][<forkPeriod>]
```

The roundTwo array now contains a new value, `disputedOverEthics`.   `disputedOverEthics` is a boolean indicating if the `event` was disputed over it's ethicality.

### backstops method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration.*
```
+ getDisputedOverEthics(event):
```
`getDisputedOverEthics` returns the `disputedOverEthics` bool for an event
```
+ setDisputedOverEthics(event):
```
`setDisputedOverEthics` sets the passed in event's `disputedOverEthics` bool to 1

## src/data_api/branches.se

### Data Structure of branches Contract:
```
data Branches[<branch>](currentVotePeriod, periodLength, markets[<index>],
numMarkets, fxpMinTradingFee, balance[<period>][<currency>], creationDate, oracleOnly, parentPeriod, baseReporters, forkPeriod, eventForkedOver, parent, contract[<currency>], numCurrencies, currencies[<index>](rate, rateContract, contract), currencyToIndex[<currency>], mostRecentChild, currencyActive[<currency>], forkTime)

data branchList[<index>]
data branchListCount
```
With the update to the contracts planned in develop, we are switching to using "currency". When you see `currency` in the contracts that value is an address for the token allowed to be used on this branch. The balance array has changed to become multi-dimensional and now takes in both a period and the currency to work with balance since we have a new concept of multiple currencies allowed. So it's not enough to just ask for the balance in a period, you need to know what denomination of currency we want to get the balance of as an example. `minTradingFee` has become `fxpMinTradingFee`, which indicates that this value should be in fix point. The `contract[<currency>]` array was added and holds the `wallet` addresses for the indexed `currency` address. `numCurrencies` is a simple count of the number of currencies currently allowed on the `branch`.

The `currencies[<index>](rate, rateContract, contract)` array was added and is a simple 0 indexed list. Inside each index you will find 3 values, the `contract` which is the `currency` address, the `rate` which is a fixed point exchange rate and the `rateContract` which is a contract with rates for the currency to be exchanged with `Eth` denominated in `Wei`. The `rate` will take precendence over the `rateContract` and only one is required per `currency`. `currencyToIndex[<currency>]` is a reverse mapping of currencies to their indices. `mostRecentChild` is the most recent child of a `branch`, `currencyActive[<currency>]` contains booleans indicating wether a currency is allowed to be used to create a new market or event. `forkTime` was also added as a timestamp for when the `branch` was forked.

### branches method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! setInitialBalance(branch, period, balance):
```
Changed:
`setInitialBalance(branch, period, balance, currency)`
`setInitialBalance` now also takes in `currency` as well in order to indicate
which currency we are setting the balance of.

```
! getInitialBalance(branch, period):
```
Changed:
`getInitialBalance(branch, period, currency)`
`getInitialBalance` also now takes `currency` in order to determine which currency you want to get the initial balance of.
```
! initializeBranch(ID, currentVotePeriod, periodLength, minTradingFee, oracleOnly, parentPeriod, parent):
```
Changed:
`initializeBranch(ID, currentVotePeriod, periodLength, fxpMinTradingFee,
 oracleOnly, parentPeriod, parent, contract, wallet, mostRecentChild):`
A few things have changed with `initializeBranch`, `minTradingFee` has become the more explicit `fxpMinTradingFee` and should be passed as a fixed point value. `contract` was added and is expecting a `currency` address as it's value, `wallet` was added and is expected to be the `wallet` address used to hold the `currency` passed as `contract`. `mostRecentChild` is the most recent child of the `branch`.

```
+ getForkTime(branch):
```
`getForkTime` was added to return the fork timestamp which is set when `setForkPeriod` is called. If the branch hasn't been forked then this will return 0.

```
+ updateNumCurrencies(branch, num):
```
`updateNumCurrencies` was added to update the number of currencies on the `branch`.

```
+ addCurrency(branch, currency, rate, rateContract):
```
`addCurrency` was added to add an new `currency` to the `branch`. You only need to pass either `rate` or `rateContract`. As mentioned above, `rate` is a fixed point exchange rate for the currency to `Eth` denominated in `Wei`. A `rateContract` is a contract that contains rates for conversion `currency` to `Eth` denominated in `Wei`.

```
+ disableCurrency(branch, currency):
```
`disableCurrency` was added to disable a `currency` on a specified `branch` from being used to create a `market` or `event` on that `branch`.

```
+ reactivateCurrency(branch, currency):
```
`reactivateCurrency` is much like the above `disableCurrency` except that it enables the `currency` specified to be used to create a new `market` or `event` on the `branch`.

```
+ replaceCurrency(branch, oldCurrencyIndex, newCurrency, newRate, newRateContract):
```
`replaceCurrency` was added to to replace a currently setup `currency` with a updated one. Takes a `branch` and the index `oldCurrencyIndex` of the `currency` we plan to replace. We give a the updated `currency` address as `newCurrency` and we can also pass a new `rate` `newRate` or a new `rateContract` `newRateContract`.

```
+ removeLastCurrency(branch):
```
`removeLastCurrency` is used to remove the most recently added `currency` from a specified `branch`.

```
+ updateCurrencyRate(branch, currency, rate, rateContract):
```
`updateCurrencyRate` was added to update the `rate` or `rateContract` for a specified `currency`.

```
+ getCurrencyRate(branch, currency):
```
`getCurrencyRate` returns the exchange rate for a specified `currency` to `Eth` denominated in `Wei`. This will use the `rate` first, if `rate` isn't defined for the specified `currency` then it will fall back to the `rateContract`.

```
+ getCurrency(branch, index):
```
`getCurrency` was added to return a currency's address given a currency's `index` for a specified `branch`.

```
+ getCurrencyByContract(branch, currency):
```
`getCurrencyByContract` returns the currency's index given a currency's address `currency` and a `branch`.

```
+ getWallet(branch, currency):
```
`getWallet` returns the `wallet` holding the specified `currency` on a certain `branch`.

```
+ getNumCurrencies(branch):
```
`getNumCurrencies` returns the number of currencies on a specific `branch`.

```
+ getCurrencyActive(branch, currency):
```
`getCurrencyActive` returns wether the specified `currency` is active and therefor usable in new markets/events created on the defined `branch`.

```
+ getMarketIDsInBranch(branch, initial, last):
```
`getMarketIDsInBranch` returns an array of marketIDs in a specified `branch`, from the `initial` index to the `last` index. This appears to have replaced the function `getSomeMarketsInBranch` which has been removed.

```
+ getBranchesStartingAt(index)
```
`getBranchesStartingAt` returns the all the branches since the specified `index` including the branch at that index.

```
+ getMostRecentChild(ID):
```
`getMostRecentChild` returns the most recent child of a specified branch `ID`.

```
+ setMostRecentChild(parent, child):
```
`setMostRecentChild` was added to set the `mostRecentChild` value for a branch. It takes in `parent` and `child`, `parent` is the branch that will be the `parent`, `child` is the value set to `mostRecentChild` for that `parent` branch.

```
- initDefaultBranch():

- getMarketsInBranch(branch):

- getSomeMarketsInBranch(branch, initial, last):
```

## src/data_api/consensusData.se

### Data Structure of consensusData Contract:
```
data branch[<branch>](period<period>](denominator, penalized[<address>](event[<event>], num, notEnoughReportsPenalized), feesCollected[<currency>][<address>], repCollected[<address>], feeFirst, periodBalance), penalizedUpTo[<address>], baseReportersLastPeriod)

data refunds[<address/event>]

data slashed[<branch>][<votePeriod>](reporter[<address>])
```
There are only a few changes to the Data Structure of the consensusData contract. One of those changes is `feesCollected[<currency>][<address>]` has become multi-dimensional, it now takes the `currency` as the first index, and an account `address` as the second. This is to facilitate the new use of currencies throughout the augur contracts. Naturally if you want to know about `feesCollected` you will now need to have the user `address` and also the `currency` type to determine what denomination of fees we are looking for. The other addition is `repCollected[<address>]`. `repCollected` has been added to indicate wether the `address` has collected `REP` or not.

### consensusData method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*



# Ignore the below please.
*Please ignore everything below this line as not part of the change log, simply some notes for upcoming updates to the change log.*
