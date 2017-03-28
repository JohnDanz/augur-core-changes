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
data roundTwo[<eventId>](
  roundTwo,
  originalVotePeriod,
  originalOutcome,
  originalEthicality,
  final,
  bondPoster,
  bondReturned,
  bondPaid,
  refund,
  disputedOverEthics
)

data forking[<eventId>](
  bondPoster,
  bondAmount,
  forkedOverEthicality,
  bondPaid,
  originalBranch,
  moved
)

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
data Branches[<branch>](
  currentVotePeriod,
  periodLength,
  markets[<index>],
  numMarkets,
  fxpMinTradingFee,
  balance[<period>][<currency>],
  creationDate,
  oracleOnly,
  parentPeriod,
  baseReporters,
  forkPeriod,
  eventForkedOver,
  parent,
  contract[<currency>],
  numCurrencies,
  currencies[<index>](
    rate,
    rateContract,
    contract
  ),
  currencyToIndex[<currency>],
  mostRecentChild,
  currencyActive[<currency>],
  forkTime
)

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
data branch[<branch>](
  period[<period>](
    denominator,
    penalized[<address>](
      event[<event>],
      num,
      notEnoughReportsPenalized
    ),
    feesCollected[<currency>][<address>],
    repCollected[<address>],
    feeFirst,
    periodBalance
  ),
  penalizedUpTo[<address>],
  baseReportersLastPeriod
)

data refunds[<address/event>]

data slashed[<branch>][<votePeriod>](
  reporter[<address>]
)
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

```
! getFeesCollected(branch, address, period):
```
Changed:
`getFeesCollected(branch, address, period, currency):`
In order to determine wether the fees have been collected for a specific `address` on a the indicated `branch` and during the selected `period`, you now need to specify which `currency` you are attempting to check. This has been added because of the addition of `currency` throughout Augur.

```
! setFeesCollected(branch, address, period):
```
Changed:
`setFeesCollected(branch, address, period, currency):`
Much like the above getter method was changed to require a `currency`, the same is true for the setter. Now we must pass in the `currency` you wish to set as collected for the specified `address`, given a `branch` and `period`.

```
+ decreaseDenominator(branch, period, amount):
```
`decreaseDenominator` lowers the denominator used in calculating fees by a specified `amount` for a specific `branch` and `period`.

```
+ getRepCollected(branch, address, period):
```
`getRepCollected` was added to indicate wether a specific `address` has had it's `REP` collected for a selected `branch` and `period`.

```
+ setRepCollected(branch, address, period):
```
`setRepCollected` was added as a way to indicate that `REP` has been collected for a specific `address` and a selected `branch` and `period`.

## src/data_api/events.se

### Data Structure of events Contract:
```
data Events[<event>](
  branch,
  expirationDate,
  outcome,
  fxpMinValue,
  fxpMaxValue,
  numOutcomes,
  markets[<index>],
  numMarkets,
  threshold,
  mode,
  uncaughtOutcome,
  ethical,
  originalExp,
  rejected,
  rejectedPeriod,
  bond,
  forked,
  forkOver,
  forkOutcome,
  forkEthicality,
  resolutionSource[<index>],
  resolutionSourceLength,
  pushedUp,
  reportersPaidSoFarForEvent,
  resolutionAddress,
  extraBond,
  firstPreliminaryOutcome,
  challenged,
  resolveBondPoster,
  earlyResolutionBond,
  creationTime
)

data past24Hours[<period>]
```
Changes in the event contract's data structure include the change from `minValue, maxValue` to `fxpMinValue, fxpMaxValue` to further indicate that the minimum value and maximum value should be fixed point. `reportersPaidSoFarForEvent` contains the number of reporters who have been paid so far for a particular event. `resolutionAddress` is the address used to resolve an event first. `extraBond` contains the bond amount used to challenge the initial resolution. `firstPreliminaryOutcome` contains the outcome reported by the `resolutionAddress`. `challenged` contains a boolean of whether the event has been challenged already or not. `resolveBondPoster` is the address that posted the `REP` bond for the first resolution period. `earlyResolutionBond` contains the bond amount paid for an early resolution of a specified event. Finally `creationTime` was added to hold the timestamp of when a specified event was created.

### events method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*
```
! initializeEvent(ID, branch, expirationDate, minValue, maxValue, numOutcomes, resolution: str):
```
Changed:
`initializeEvent(ID, branch, expirationDate, fxpMinValue, fxpMaxValue, numOutcomes, resolution: str, resolutionAddress, resolveBondPoster):`
The changes here come from the switch from plain `minValue` and `maxValue` to `fxpMinValue` and `fxpMaxValue` respectively. You also have two new fields expected, `resolutionAddress` which is optionally an address you want to have report to resolve the market. This is done if you want to have only 1 source for reporting, if this is not defined then this event will default to regular reporting. `resolveBondPoster` is the address of a person who posted the resolution bond, generally this will be the person who created the event.

```
+ getCreationTime(event):
```
`getCreationTime` was added to get the timestamp of when the specified `event` was created.

```
+ setCreationTime(event):
```
`setCreationTime` was added to set the timestamp of when the specified `event` is created.

```
+ getResolveBondPoster(event):
```
`getResolveBondPoster` has been added to return the address of the account who posted the resolution bond for a specific `event`.

```
+ getChallenged(event):
```
`getChallenged` returns wether the specified `event` has been challenged or not.

```
+ setChallenged(event):
```
`setChallenged` sets the specific `event` to challenged.

```
+ getFirstPreliminaryOutcome(event):
```
`getFirstPreliminaryOutcome` returns the outcome submitted by the `resolutionAddress`, which is stored as firstPreliminaryOutcome, for a specific `event`.

```
+ setFirstPreliminaryOutcome(event, outcome):
```
`setFirstPreliminaryOutcome` was added to set the `firstPreliminaryOutcome` which is the `outcome` submitted by the `resolutionAddress`. It takes in a specified `event` and the `outcome` we intend to set.

```
+ getReportersPaidSoFar(event):
```
`getReportersPaidSoFar` was added to return the count of the number of people who have reported so far on the specified `event`.

```
+ addReportersPaidSoFar(event):
```
`addReportersPaidSoFar` has been added to increment the count of the reporters paid so far for a specified `event`.

```
+ getResolutionAddress(event):
```
`getResolutionAddress` was added to return the `resolutionAddress` if one exists for a specified `event`.

```
+ getEventResolution(event):
```
`getEventResolution` was created to return the resolution string for a specified `event`. This appears to be a renamed version of the removed method `getResolution`.

```
+ getExtraBond(event):
```
`getExtraBond` returns the bond amount used to challenge the initial resolution for a given `event`.

```
+ setExtraBond(event, extraBond):
```
`setExtraBond` is used to set the challenge bond amount `extraBond` for a specified `event`.

```
+ getEarlyResolutionBond(event):
```
`getEarlyResolutionBond` is used to retrieve the early resolution bond amount for a specified `event`.

```
+ setEarlyResolutionBond(event, bond):
```
`setEarlyResolutionBond` is used to set the early resolution `bond` amount for a certain `event`.

```
- getResolution(event):

- getEthical(event):

- getBranch(event):
```

## src/data_api/expiringEvents.se

### Data Structure of expiringEvents Contract:
```
data periodEventInfo[<branch>][<period>](
  events[<index>],
  eventToIndex[<event>],
  requiredEvents[<event>],
  committed[<event>],
  subsidy[<event>],
  eventWeight[<event>],
  lesserReportNum[<event>],
  numberEvents,
  roundTwoNumEvents,
  numReqEvents,
  numberRemoved,
  numEventsToReportOn,
  feeValue,
  afterFork
)

data reporterPeriodInfo[<branch>][<period>](
  beforeRep[<address>],
  afterRep[<address>],
  periodDormantRep[<address>],
  reportHash[<address>][<event>],
  saltyEncryptedHash[<address>][<event>],
  report[<address>][<event>],
  ethics[<address>][<event>],
  numReportsSubmitted[<event>],
  periodRepWeight[<address>],
  numberOfActiveReporters,
  reporters[<index>]
)

data modeItems[<period>][<event>](
  reportValue[<report>],
  currentMode,
  currentModeItems
)
```
`expiringEvents`'s data structure has changed to a more simple structure overall. Previously there was a total of six data structures, but this has been shaved down to just three. The first is `periodEventInfo` which takes in a `branch` and `period`. `events[<index>]` and `eventToIndex[<event>]` are both arrays, `events[<index>]` contains a mapping of an index to event ID, where as `eventToIndex[<event>]` contains a mapping of an event ID to it's index. `requiredEvents[<event>]` contains a boolean to determine wether the event specified is required to be reported on. `committed[<event>]` keeps a count per event of how many reports have been committed so far for that event. `subsidy[<event>]` contains the amount of money used to payback the person who did the work to estimate the number of reporters needed for a specific `event`. `eventWeight[<event>]` contains event weight for a specific `event`. event weight is the number of reporters on an event in round 1 or the total rep reported on an event in backstop 1 or fork event.

`lesserReportNum[<event>]` contains the number of reports you should have for a specified event. `numberEvents, roundTwoNumEvents, numReqEvents, numberRemoved` and `numEventsToReportOn` are all simple counts for the various things they are named for. The names seem descriptive enough to understand what each of those fields contain. `feeValue` returns the total fees for all markets on this `branch` expiring in this `period` denominated in `Wei`. `afterFork` contains the number of events created for a fork or 2 periods after the fork provided those events were created after the fork.

The next data structure is `reporterPeriodInfo[<branch>][<period>]` which contains the reporter information for a specific `period`. `beforeRep[<address>]` and `afterRep[<address>]` both take in a reporter's address and return the amount of active rep for that reporter either before any modifications to `REP` for the period or after all modifications are complete. `periodDormantRep[<address>]` contains the amount of dormant `REP` for a specified account `address`. `reportHash[<address>][<event>]` contains the `reportHash` for a specific `event` submitted by a specific `address`. `saltyEncryptedHash[<address>][<event>]` holds the `saltyEncryptedHash` for a specific `address` and `event`. `report[<address>][<event>]` contains the actual reports for a specified reporter `address` and `event`. `ethics[<address>][<event>]` contains the ethicality of each report given a specified reporter `address` and `event`. `numReportsSubmitted[<event>]` is map of counts of the number of reports submitted for a specified `event` ID. `periodRepWeight[<address>]` contains weighting used used in the calculation of how many events a specific reporter `address` needs to report on. `numberOfActiveReporters` is the number of active Reporters for a specific `branch` and `period`. `reporters[<index>]` is a zero indexed array that maps to a reporter address. Finally we have `modeItems[<period>][<event>]` which is essentially unchanged except it's moved to a camelCase style instead of an_underscore_style of naming.

### expiringEvents method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! refundCost(to, value):
```
Changed: `refundCost(to, branch, period, event):` to require `branch`, `period`, and `event` for the event we plan to refund for the cost of calculating the required number of reporters. `value` has been removed from the params as the amount to refund is stored in `periodEventInfo[branch][period].subsidy[event]`.

```
! getRequired(event):
```
Changed: `getRequired(event, period, branch):` to also require a `branch` and `period` in order to return wether a specified `event` is required to be reported on.

```
! getEvents(branch, expDateIndex):

! getEventsRange(branch, expDateIndex, start, end):

! getNumEventsToReportOn(branch, expDateIndex):

! getNumberEvents(branch, expDateIndex):

! getEvent(branch, expDateIndex, eventIndex):

! getReportHash(branch, expDateIndex, reporter, event):

! setReportHash(branch, expDateIndex, reporter, reportHash, event):
```
Changed: The above functions have remained functionally the same, the only difference is that instead of `expDateIndex` the param has been renamed to simply `period`. Argument wise, the values remain the same.

```
! getEventIndex(period, eventID):
```
Changed: `getEventIndex(branch, period, event):` to also required a `branch` and renamed the param `eventID` to simply `event`.

```
! addEvent(branch, futurePeriod, eventID, subsidy):
```
Changed: `addEvent(branch, futurePeriod, event, subsidy, currency, wallet, afterFork):` to require more params because of the `currency` changes. `eventID` has been renamed to simply `event`, `currency` is the address of the `currency` we plan to use for this `event`. `wallet` is the wallet address of the wallet intended to contain the `currency` for this event, and `afterFork` is passed as 0 or 1, depending on if this event is being added after a fork or not.

```
+ getSaltyEncryptedHash(branch, period, reporter, event):
```
`getSaltyEncryptedHash` was added to return the encrypted hash for a specified `branch`, `period`, and `event` that was submitted by `reporter`.

```
+ setSaltyEncryptedHash(branch, period, reporter, saltyEncryptedHash, event):
```
`setSaltyEncryptedHash` was added to set the encrypted hash, `saltyEncryptedHash`, for a specific `reporter` given a `branch`, `period`, and `event`.

```
+ getPeriodRepWeight(branch, votePeriod, sender):
```
`getPeriodRepWeight` was added to return the weight used to calculate how many events a reporter, `sender`, should report on for a specified `branch` and `votePeriod`.

```
+ setPeriodRepWeight(branch, votePeriod, sender, value):
```
`setPeriodRepWeight` was added to set the REP weight for a specific reporter, `sender`, to a `value` given a `branch` and `votePeriod`.

```
+ getNumReportsSubmitted(branch, votePeriod, sender):
```
`getNumReportsSubmitted` returns the number of reports submitted by `sender` in a specified `branch` and `votePeriod`.

```
+ getEventWeight(branch, votePeriod, event):
```
`getEventWeight` returns either the number of reports for a specific `event` on a `branch` and `votePeriod` if the `event` is a round 1 event. If it's in backstop 1 or a fork event it will return the total REP reported on the event.

```
+ getReportsCommitted(branch, period, event):
```
`getReportsCommitted` returns the amount of reports committed for a specific `event` in a `branch` and `period`.

```
+ getFeeValue(branch, expIndex):
```
`getFeeValue` returns the value of all fees, for all markets that have events that will be expiring in a specific `branch` and period `expIndex`.

```
+ adjustPeriodFeeValue(branch, expIndex, amount):
```
`adjustPeriodFeeValue` is used to modify the value of all the fees for all markets that have events expiring in a specific `branch` and period `expIndex` by a specific `amount`.

```
+ setEventWeight(branch, votePeriod, event, num):
```
`setEventWeight` is used to set the event weight to a specific value `num` for a given `branch`, `votePeriod`, and `event`.

```
+ countReportAsSubmitted(branch, votePeriod, event, sender, weight):
```
`countReportAsSubmitted` is used to increment an event's weight, and update the number of reports submitted. `branch`, `votePeriod`, and `event` are used to target the specific `event`, `weight` is the weight value to be added to the specific `event` and `sender` is used to target the reporter to increment it's number of reports submitted count.

```
+ addReportToReportsSubmitted(branch, period, user):
```
`addReportToReportsSubmitted` is used to increment the count of reports submitted for a specific reporter `user` in a `branch` and `period`.

```
+ getActiveReporters(branch, period, from, to):
```
`getActiveReporters` returns an array of active reporter addresses for a specific `branch` and `period` given a start, `from`, and end, `to`, index.

```
+ getNumActiveReporters(branch, period):
```
`getNumActiveReporters` returns the number of active reporters for a given `branch` and `period`.

```
+ getAfterFork(branch, votePeriod):
```
`getAfterFork` returns the number of events created for a fork period or 2 periods after a fork provided that the events were created after the fork. It takes a specific `branch` and `votePeriod` to return the number.


```
- getEncryptedReport(branch, expDateIndex, reporter, event):

- setEncryptedReport(branch, expDateIndex, reporter, report, salt, ethics, events):

- getReportersPaidSoFar(branch, event):

- addReportersPaidSoFar(branch, event):

- getPeriodRepConstant(branch, votePeriod, sender):

- setPeriodRepConstant(branch, votePeriod, sender, value):

- getRepEvent(branch, votePeriod, event):

- getNumReportsEvent(branch, votePeriod, eventID):

- getNumReportsActual(branch, votePeriod, sender):

- getShareValue(branch, expIndex):

- adjustPeriodShareValueOutstanding(branch, expIndex, amount):

- addRepEvent(branch, votePeriod, event, amount):

- setNumReportsEvent(branch, votePeriod, eventID, num):

- addReportToEvent(branch, votePeriod, eventID, sender):
```

## src/data_api/info.se

### Data Structure of info Contract:
```
data Info[<branch/event/market ID>](
  description[2048],
  descriptionLength,
  creator,
  creationFee,
  wallet,
  currency
)
```
`info`'s data structure has only changed slightly. It still is indexed by an `ID` (this can be a branch ID, an event ID, or a market ID since these all share the same types of metadata). The new fields are `wallet` and `currency` which were added to accommodate the changes in the contracts to using the `currency` system.

### info method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! setInfo(ID, description: str, creator, fee):
```
Changed: `setInfo(ID, description: str, creator, fxpFee, currency, wallet):` so that it would also take in a `currency` address and `wallet` address for the entered `currency`. Another change is `fee` has become `fxpFee` to indicate that the value is fixed point.

```
+ getCurrency(ID):
```
`getCurrency(ID):` was added to return the `currency` used for the specified branch, event, or market `ID`.

```
+ getWallet(ID):
```
`getWallet(ID):` was added to get the `wallet` used for the specified branch, event, or market `ID`.

```
+ setCurrencyAndWallet(ID, currency, wallet):
```
`setCurrencyAndWallet(ID, currency, wallet):` was added to set the `currency` and `wallet` used for the specified branch, event, or market `ID`.

## src/data_api/markets.se

### Data Structure of markets Contract:
```
data Markets[<market>](
  events[<index>],
  lenEvents,
  sharesPurchased[<index(starts at 1)>],
  participants[<address>](
    shares[<outcome>]
  ),
  winningOutcomes[<index>],
  cumulativeScale,
  numOutcomes,
  tradingPeriod,
  fxpTradingFee,
  branch,
  volume,
  pushingForward,
  bondsMan,
  originalPeriod,
  orderIDs[<order>](
    id,
    nextID,
    prevID
  ),
  lastOrder,
  totalOrders,
  tag1,
  tag2,
  tag3,
  extraInfo[<index>],
  extraInfoLen,
  sharesValue,
  gasSubsidy,
  fees,
  lastExpDate,
  prices[<outcome>],
  shareContracts[<outcome>]
)

data marketsHash[<branch>]
```
The `markets` contract's data structure has had some moderate changes. `makerFees`, `creationBlock`, and `creationTime` are no longer part of the Markets array data structure. `tradingFee` has been renamed to `fxpTradingFee` to indicate this is a fixed point value. `trade_ids` has been renamed to `orderIDs` and the data contained within has been renamed from `next_id` and `prev_id` to `nextID` and `prevID` respectively. Other renamed fields include `last_trade` and `total_trades` which have been converted to `lastOrder` and `totalOrders` respectively. `shareContracts[<outcome>]` has been added to store the erc20 token contract address for the shares related to each outcome in a market.

Finally a new data structure was added with the following signature `marketsHash[<branch>]`. Given a `branch`, this array will contain a composite hash of all the markets on the specified `branch`.

### markets method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! addFees(market, amount):
```
Changed `addFees(market, fxpAmount):` to use `fxpAmount` instead of simply `amount` to help indicate this is a fixed point value.

```
! setPrice(market, outcome, price):
```
Changed `setPrice(market, outcome, fxpPrice):` to use `fxpPrice` instead of simply `price` to help indicate this is a fixed point value.

```
! getgasSubsidy(market):
```
Renamed to `getGasSubsidy(market):`.

```
! getCumScale(market):
```
Renamed to `getCumulativeScale(market):`.

```
! getBranchID(market):
```
Renamed to `getBranch(market):`.

```
! modifyShares(marketID, outcome, amount):
```
Changed `modifyShares(market, outcome, fxpAmount):` to use `fxpAmount` instead of just `amount` to indicate this is a fixed point value.

```
! modifySharesValue(marketID, amount):
```
Changed `modifySharesValue(market, fxpAmount):` to use `fxpAmount` instead of just `amount` to indicate this is a fixed point value.

```
! modifyParticipantShares(marketID, trader, outcome, amount, cancel):
```
Changed `modifyParticipantShares(market, trader, outcome, fxpAmount, actualTrade):` to use just `market` instead of `marketID` and `fxpAmount` instead of just `amount` to indicate a fixed point value. `cancel` has been removed and replaced with `actualTrade`. `actualTrade` is a boolean, if it's `1` the trade will be considered a real trade and modify the market's volume, if it's a `0` it will not effect the market volume.

```
! getLastTrade(market):
```
Renamed `getLastOrder(market):`.

```
! get_total_trades(market_id):
```
Renamed `getTotalOrders(marketID):`.

```
! remove_trade_from_market(market_id, trade_id):
```
Renamed `removeOrderFromMarket(marketID, orderID):`. It was also modified to use `marketID` and `orderID` instead of `market_id` and `trade_id` as it's params.

```
! initializeMarket(marketID, events: arr, tradingPeriod, tradingFee, branch, tag1, tag2, tag3, makerFees, cumScale, numOutcomes, extraInfo: str, gasSubsidy, creationFee, lastExpDate):
```
Changed `initializeMarket(market, events: arr, tradingPeriod, fxpTradingFee, branch, tag1, tag2, tag3, fxpcumulativeScale, numOutcomes, extraInfo: str, gasSubsidy, fxpCreationFee, lastExpDate, shareContracts: arr):` in a a few ways. `marketID` has been changed to just `market`. `tradingFee`, `cumScale`, and `creationFee` have all been renamed to `fxpTradingFee`, `fxpcumulativeScale`, and `fxpCreationFee` respectively to indicate that they are all fixed point values. `makerFees` has been removed. Finally `shareContracts` array was added, this array contains the addresses for erc20 share tokens for each outcome in the market.

```
+ getMarketsHash(branch):
```
`getMarketsHash` was added to return the composite market hash of all markets on the specified `branch`.

```
+ addToMarketsHash(branch, newHash):
```
`addToMarketsHash` is used to add a new market hash, `newHash` to the composite markets hash on a specified `branch`. This is done by taking the current composite market hash, and the new hash into an array of length 2, then rehashed using SHA3.

```
+ getMarketShareContracts(market):
```
`getMarketShareContracts` returns an array of share erc20 token contract addresses, one for each outcome in the `market` specified.

```
+ getOrderIDs(marketID):
```
`getOrderIDs` returns an array of orderIDs for a specific `marketID`.

```
+ getPrevID(market, order):
```
`getPrevID` returns the previous `order` to the specified `order` on a specific `market`.


```
- getMakerFees(market):

- setMakerFees(market, makerFees):

- getTopic(market):

- get_trade_ids(market_id, offset, numTradesToLoad):

- getCreationTime(market):

- getCreationBlock(market):
```

# Ignore the below please.
*Please ignore everything below this line as not part of the change log, simply some notes for upcoming updates to the change log.*


!addTrade(market, trade_id, last_id):(became addOrder?)+addOrder(market, orderID):
