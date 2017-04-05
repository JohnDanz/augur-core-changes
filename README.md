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
  * topics.se
  - trades.se (became orders.se)
  + orders.se (was trades.se)
src/functions/
  - buy&sellShares.se (became bidAndAsk.se)
  + bidAndAsk.se (was buy&sellShares.se)
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

## src/data_api/compositeGetters.se

### Data Structure of compositeGetters Contract:

compositeGetters doesn't have it's own data structure, it's also been moved from the `functions` folder to the `data_api` folder.

### compositeGetters method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! getOrderBook(marketID, offset, numTradesToLoad):
```
Changed `getOrderBook(marketID, offset, numOrdersToLoad):` to use `numOrdersToLoad` instead of `numTradesToLoad`, however the function remains functionally unchanged.


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

event logOutcome(event:indexed, outcome)
```
Changes in the event contract's data structure include the change from `minValue, maxValue` to `fxpMinValue, fxpMaxValue` to further indicate that the minimum value and maximum value should be fixed point. `reportersPaidSoFarForEvent` contains the number of reporters who have been paid so far for a particular event. `resolutionAddress` is the address used to resolve an event first. `extraBond` contains the bond amount used to challenge the initial resolution. `firstPreliminaryOutcome` contains the outcome reported by the `resolutionAddress`. `challenged` contains a boolean of whether the event has been challenged already or not. `resolveBondPoster` is the address that posted the `REP` bond for the first resolution period. `earlyResolutionBond` contains the bond amount paid for an early resolution of a specified event. Finally `creationTime` was added to hold the timestamp of when a specified event was created.

Finally, one log event is defined in this contract called `logOutcome`. It stores an `event` and `outcome`. This log is fired off by the `getOutcome` method if the message sender calling `getOutcome` isn't the owner of the `event` or on the whitelist.

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
Changed `initializeMarket(market, events: arr, tradingPeriod, fxpTradingFee, branch, tag1, tag2, tag3, fxpcumulativeScale, numOutcomes, extraInfo: str, gasSubsidy, fxpCreationFee, lastExpDate, shareContracts: arr):`. `marketID` has been changed to just `market`. `tradingFee`, `cumScale`, and `creationFee` have all been renamed to `fxpTradingFee`, `fxpcumulativeScale`, and `fxpCreationFee` respectively to indicate that they are all fixed point values. `makerFees` has been removed. Finally `shareContracts` array was added, this array contains the addresses for erc20 share tokens for each outcome in the market.

```
! addTrade(market, trade_id, last_id):
```
Changed `addOrder(market, orderID):`. `addTrade` has been renamed to `addOrder` and has had it's params updated. `trade_id` has been changed to `orderID` and `last_id` is no longer required so the param has been dropped completely.

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

## src/data_api/reporting.se

### Data Structure of reporting Contract:
```
data Reporting[<branch>](
  reputation[<index>](
    repValue,
    reporterID
  ),
  numberReporters,
  repIDtoIndex[<address>],
  totalRep,
  dormantRep[<index>](
    repValue,
    reporterID
  ),
  activeRep,
  fork,
  reportedOnNonFinalRoundTwoEvent
)

data whitelists[<contract>](
  addresses[<address>],
  taken
)
```
The only addition here is `reportedOnNonFinalRoundTwoEvent` added to the `Reporting[<branch>]` data structure. When a person reports on a round 2 event before it was in the second round [i.e. the first reporting backstop], then `reportedOnNonFinalRoundTwoEvent` will be set to the eventID of the event that was reported on. A reporter cannot convert their rep to dormant or send rep until they've finished the resolution process for that round 2 event. Once the round 2 event is final then `reportedOnNonFinalRoundTwoEvent` should be set to 0. Otherwise the data structure for reporting has remained the same.


### reporting method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! adjustActiveRep(branch, amount):
```
Changed `adjustActiveRep(branch, fxpAmount):` to use `fxpAmount` instead of `amount` to indicate that this value should be fixed point.

```
! setInitialReporters(parent, branchID):
```
Changed `setInitialReporters(branch):` to no longer require a `parent` param and `branchID` has been renamed to simply `branch`.

```
! addReporter(branch, sender, amount, dormant, repToBonderOrBranch):
```
Changed `addReporter(branch, sender, fxpAmount, fxpDormant, fxpRepToBonderOrBranch):` has had `amount`, `dormant`, and `repToBonderOrBranch` renamed to `fxpAmount`, `fxpDormant`, and `fxpRepToBonderOrBranch` respectively in order to indicate they are fixed point values.

```
! addRep(branch, index, value):
! subtractRep(branch, index, value):
! setRep(branch, index, newRep):
! addDormantRep(branch, index, value):
! subtractDormantRep(branch, index, value):
```
Changed:
```
addRep(branch, index, fxpValue):
subtractRep(branch, index, fxpValue):
setRep(branch, index, fxpNewRep):
addDormantRep(branch, index, fxpValue):
subtractDormantRep(branch, index, fxpValue):
```
The 5 methods above have renamed the `value` or `newRep` params to `fxpValue` or `fxpNewRep` to indicate that they should be a fixed point value.

```
! setSaleDistribution(addresses: arr, balances: arr, branchID):
```
Changed `setSaleDistribution(addresses: arr, balances: arr, branch):` changed `branchID` to simply `branch`.

```
+ getReportedOnNonFinalRoundTwoEvent(branch):
```
`getReportedOnNonFinalRoundTwoEvent` was added to return the value contained in `reportedOnNonFinalRoundTwoEvent` given a specified `branch`. This will result in an `eventID` or `0` being returned.

```
+ setReportedOnNonFinalRoundTwoEvent(branch, event):
```
`setReportedOnNonFinalRoundTwoEvent` has been added to set the `reportedOnNonFinalRoundTwoEvent` value to the `event` passed on the specified `branch`.

```
+ claimInitialRep():
```
`claimInitialRep` is used to claim initial `REP` for the sender from the `repContract`. This `REP` will be dormant until activated.

## src/data_api/reportingThreshold.se

### Data Structure of reportingThreshold Contract:

`reportingThreshold` has no data structure of it's own. It contains all the methods required to deal with reporting thresholds.


### reportingThreshold methods:
*This is a new contract, as such all methods are new methods.*

```
calculateReportingThreshold(branch, event, period, sender):
```
`calculateReportingThreshold` is used to determine the reporting threshold for a specified `event` and `sender` in a specific `branch` and `period`. This function returns the calculated reporting threshold, which is used to determine if a reporter should report on a specific `event` or not.

```
getEventsToReportOn(branch, period, sender, start, end):
```
`getEventsToReportOn` is used to get a list of events for a reporter to report on given an index range for the pool of events to select from. This method determines what reports a specific `sender` should report on in a given `branch` and `period`. This takes a `start` and `end` index to limit the potential events that might be selected for a reporter. Any event where the `sender`'s SHA3 hash of their `address` + the `eventID` normalized to 1 is below the report threshold for the specific event will be added to the list of events to report on for the `sender`.

```
getEventCanReportOn(branch, period, reporter, event):
```
`getEventCanReportOn` is used to determine if a `reporter` is eligible to report on a specific `event` in a given `branch` and `period`. returns `1` if they can, `0` if they cannot.

```
setReportingThreshold(event):
```
`setReportingThreshold` is used to change to change the threshold of a given `event` to the maximum threshold. In the rare possibility that less than 3 reporters get randomly selected to report on a market in a given period, on the last day, we can change the SHA3 threshold using this function. This would be called from the UI.

```
calculateNumberOfEventsAReporterHasToReportOnAtMinimum(branch, reporter, period):
```
`calculateNumberOfEventsAReporterHasToReportOnAtMinimum` is used to return the minimum number of events a `reporter` needs to report on for a given `branch` and `period`. For example, if the `reporter` is eligible to report on 4 events, the minimum amount of events the `reporter` must report on is 2.

```
findLazyReportersAndLeechers(branch, votePeriod, reporterStart, reporterEnd, eventStart, eventEnd):
```
`findLazyReportersAndLeechers` is used to penalize people who had at least 1 REP active but didn't report on the minimum number of reports. It loops through the list of active reporters, given a range from `reporterStart` to `reporterEnd` for a specific `branch` and `votePeriod`. It checks reporters minimum number of events to report on given a range from `eventStart` to `eventEnd`. Both `reporterEnd` and `eventEnd` will default to the total number of reporters or events respectively if they are passed as 0. If the reporters have reported on less than the minimum we find an example of an event they could have reported on but didn't and then return two arrays, one with address of reporters who need to be penalized and another of the example event addresses they could have reported on.

## src/data_api/orders.se

### Data Structure of orders Contract:
```
data orderCommits[<address>](
  hash,
  block
)

data orders[<order>](
  id,
  type,
  market,
  amount,
  price,
  owner,
  block,
  outcome,
  sharesEscrowed,
  moneyEscrowed
)
```
`trades.se` has become `orders.se` and gone through some changes in the new contracts. `tradeCommits` has been changed to `orderCommits`. It's still indexed by a trader address and contains the transaction hash `hash` and block number `block` for each order committed. `trades` has become `orders` which is indexed by an `order` ID. The two new values added to `orders` are `sharesEscrowed` and `moneyEscrowed` which represent the amount of shares or money held to be used to fulfill bids and asks.

### orders method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! makeTradeHash(max_value, max_amount, trade_ids: arr):
```
Changed `makeOrderHash(market, outcome, direction):` to use different values to generate the orderHash, which is now a SHA3 hash generated by a length 4 array containing the `market`, `outcome`, `direction`, and `sender`. `market` is the market ID, `outcome` is the outcome we have traded on, and `direction` is wether it's a buy or sell.

```
! commitTrade(hash):
```
Renamed `commitOrder(hash):`.

```
! getID(tradeID):
```
Changed `getID(orderID):` to rename the param `orderID` instead of `tradeID`.

```
! saveTrade(trade_id, type, market, amount, price, sender, outcome):
```
Changed `saveOrder(orderID, type, market, amount, price, sender, outcome, money, shares):` has been renamed to `saveOrder`. It takes all the same args but adds two more to reflect the updated data structure. `money` and `shares` are also required so they can be put in escrow.

```
! get_trade(id):
```
Renamed `getOrder(id):`.

```
! get_amount(id):
```
Renamed `getAmount(id):`.

```
! get_price(id):
```
Renamed `getPrice(id):`.

```
! remove_trade(id):
```
Renamed `removeOrder(id):`.

```
! fill_trade(id, fill):
```
Changed `fillOrder(orderID, fill, money, shares):` was renamed to `fillOrder`. renamed the `id` param to `orderID`. Added `money` and `shares` as new arguments to be able to know how much `money` or `shares` to fill from the escrowed money and shares.


```
- zeroHash():

- getBestBidID(market, outcome):

- getBestAskID(market, outcome):

- getTradeOwner(id):

- get_trade_block(id):

- update_trade(id, price):

- getSender():
```

## src/data_api/topics.se

`Topics` actually hasn't changed at all between the current version of master and develop, I just included it here to make it clear I didn't forget to inspect `topics`, but that it just hasn't changed.

## src/functions/bidAndAsk.se

### Data Structure of bidAndAsk Contract:
```
event logAddTx(
  market:indexed,
  sender:indexed,
  type,
  fxpPrice,
  fxpAmount,
  outcome,
  orderID,
  moneyEscrowed,
  sharesEscrowed
)

event logCancel(
  market:indexed,
  sender:indexed,
  fxpPrice,
  fxpAmount,
  orderID,
  outcome,
  type,
  money,
  shares
)

event buyAndSellSharesLogReturn(
  returnValue
)
```
There is no data structures for the `bidAndAsk` contract but there are events defined that are emitted by some of the methods in `bidAndAsk`. It appears that for the most part, what used to be `buy&sellShares.se` has been renamed to `bidAndAsk.se` and some methods have been removed and have been replaced with a more all encompassing `placeOrder` method instead of individual `buy`, `sell`, and `shortAsk` methods.

`logAddTx` is fired off when `placeOrder` is called and it is successful. `logCancel` is fired off when a user decides to `cancel` an order. `buyAndSellSharesLogReturn` is fired off to indicate wether the attempt to place or cancel an order was successful or not. If you place an order, and it's successful it will log the orderID as the `returnValue`. If it's a failure, it will log `0` which is a failure. If you attempt to cancel an order, and it's successful then the returnValue for the log will be `1` which is a success.

### bidAndAsk method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! cancel(trade_id):
```
Changed `cancel(orderID):` to use `orderID` instead of `trade_id` as the param. This function is used to cancel an order before it's been filled. This will refund the shares or money in escrow to the message sender. It also creates a `logCancel` log event to show what order was canceled. When it completes successfully it will fire off another event to be logged: `buyAndSellSharesLogReturn`. The `buyAndSellSharesLogReturn` log will have it's `returnValue` set to `1` (Success) if the order was successfully canceled or `0` (Failure) if the order couldn't be canceled.

```
+ placeOrder(type, fxpAmount, fxpPrice, market, outcome):
```
`placeOrder` is a new method that is used for all orders. It takes a `type` of order, bid (`1`) or ask (`2`). Next it takes the amount of shares in fixed point `fxpAmount`. Then the price per share in fixed point `fxpPrice`. Which `market` and on what `outcome` we are placing this order are the final arguments. As with `cancel` above, there are logs generated from calling `placeOrder`. If the order is placed successfully we should see a `logAddTx` log saved with the order details. We will also see a `buyAndSellSharesLogReturn` which will have it's `returnValue` set to the `orderID` of the successfully placed order. If the order is not placed successfully there will be no `logAddTx` created and `buyAndSellSharesLogReturn` log will have it's `returnValue` set to `0` (Failure).


The below are methods that used to exist in `buy&sellShares.se` but don't exist in `bidAndAsk.se`. I've included them to make it clear that these functions don't exist anymore in the new contracts.
```
- shortAsk(amount, price, market, outcome, minimumTradeSize, tradeGroupID):

- buy(amount, price, market, outcome, minimumTradeSize, tradeGroupID):

- sell(amount, price, market, outcome, minimumTradeSize, isShortAsk, tradeGroupID):
```

## src/functions/cash.se

### Data Structure of cash Contract:
```
data accounts[2**160<address>](
  balance,
  spenders[2**160<address>](
    maxValue
  )
)

data totalSupply

data name

data symbol

data decimals

event Transfer(
  from:indexed,
  to:indexed,
  value
)

event Approval(
  owner:indexed,
  spender:indexed,
  value
)
```
The `cash` contract has moved from the `data_api` folder to the `functions` folder. The data structure and events are all completely different then the previous implementation of `cash` currently in master. We have a new data array called `accounts` which is indexed by addresses. Each address will have a `balance` and another array called `spenders` which is also indexed by addresses, in this case an address authorized to spend for you. Inside of the `spenders` array contains a single value, `maxValue` which is the maximum value a `spender` can spend from the owner of the account's balance.

We then have 4 pieces of data: `totalSupply`, 'name', 'symbol', and 'decimals'. `totalSupply` is the total supply of cash in the `Cash` contract. `name` is the name of the currency used in the cash contract which is currently set to `"Cash"` during `init()`. `symbol` is the symbol used as a shorthand for the name, like a $ sign is for US Dollars, currently set to `"CASH"` during `init()`. `decimals` is the number of decimal places used for a unit of REP, currently set to `18` during `init()`.

Two events are also potentially emitted to save logs from this contract, they are `Transfer` and `Approval`. I'll discuss where they are emitted in the method section below.

### cash method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! balance(address):
```
Renamed `balanceOf(address: address):`.

```
+ transfer(to: address, value: uint256):
```
`transfer` is used to transfer a `value` of funds from the `msg.sender`'s account and send it to the address `to`. If the sender doesn't have enough funds or the value sent in is negative, this will fail. Successfully transferring funds will emit a `Transfer` event to save a log of the transfer.

```
+ transferFrom(from: address, to: address, value: uint256):
```
`transferFrom` is very similar to `transfer`, the only difference is we explicitly send in a `from` address instead of using `msg.sender` as the account to withdraw from.

```
+ approve(spender: address, value: uint256):
```
`approve` has been added to to authorize a `spender` address to spend up to `value` of the `msg.sender`'s account balance.


```
+ allowance(owner: address, spender: address):
```
`allowance` has been added to check the amount a `spender` address is allowed to spend from a `owner` address' account balance.

```
+ totalSupply():
```
`totalSupply` added to return the value of `totalSupply`, which is the total supply of all Ether in this contract.

```
+ getName():
```
`getName` added to return the value of `name`, as of now set to `"Cash"` when `init()` is called.

```
+ getDecimals():
```
`getDecimals` added to return the value of `decimals`, which is set to `18` during `init()`.

```
+ getSymbol():
```
`getSymbol` added to return the value of `symbol`, which is set to `"CASH"` during `init()`.

```
+ commitSuicide():
```
`commitSuicide` will delete and remove the contract from the blockchain and return all ether in this contract to the `msg.sender`. `msg.sender` must be equal to the suicide address or this will fail.

```
- initiateOwner(account):
- send(recver, value):
- sendFrom(recver, value, from):
- addCash(ID, amount):
- subtractCash(ID, amount):
```

## src/functions/claimProceeds.se

### Data Structure of claimProceeds Contract:

There is no data structure for the `claimProceeds` contract.

### claimProceeds method:

```
+ claimProceeds(market):
```
`claimProceeds` is used to claim the trading profits per share after a `market` is resolved. If the `market` is not resolved this will fail.


## src/functions/closeMarket.se

### Data Structure of closeMarket Contract:
```
event closeMarketLogReturn(
  returnValue
)
```
`closeMarket` has changed a bit from it's current iteration in master. It used to contain a `closeMarket` event, and a `closeMarket_logReturn` event, however `closeMarket` is no longer an event and `closeMarket_logReturn` has been renamed to `closeMarketLogReturn`.

### closeMarket method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! closeMarket(branch, market, sender):
```
Changed `closeMarket(market, sender):` to no longer require a `branch` param. `closeMarket` is still used to get winning outcomes of events related to a `market` and then resolves the `market` using the winning outcomes of the events.

```
+ closeChallengeBond(event):
```
`closeChallengeBond` was added to close out a challenge bond for a specific `event`. If the event is not challenged this will fail. If the event is found to have the same outcome then the bond is lost, if the event is found to have a different outcome then the bond is returned + half of the validity bond is added as payment for a successful challenge.  

```
- claimProceeds(branch, market):
```

## src/functions/collectFees.se

### Data Structure of collectFees Contract:
`collectFees` has no data structure of it's own. It also no longer has an event called `collectedFees` defined.

### collectFees method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! collectFees(branch, sender):
```
Changed to `collectFees(branch, sender, currency):`. In addition to needing a `branch` and `sender` to collect fees for, we also now need to indicate which `currency` we would like to collect. This transfers the fees from the `branch` wallet for the specific `currency` to the `sender`. Possible return values are `1`, for success, `0` for fees already collected, `-1` for penalization not done, `-2` if we aren't in the 2nd half of reporting period, `-3` if rep hasn't been collected yet, and `2` if there was no error but the sender didn't report last period.

```
+ collectRep(branch, sender):
```
`collectRep` was added to collect the REP at the end of a period. It requires a `branch` and `sender`. Possible return values are `1`, for success, `0` for fees already collected, `-1` for penalization not done, `-2` if we aren't in the 2nd half of reporting period, and `2` if there was no error but the sender didn't report last period.

## src/functions/completeSets.se

### Data Structure of completeSets Contract:
```
event completeSetsLogReturn(
  sender: indexed,
  market: indexed,
  type: indexed,
  returnValue,
  numOutcomes
)
```
`completeSets` has no data structure of it's own. It does however have an event, `completeSetsLogReturn` which creates a log with information about the `sender`, `market`, `type` of trade, the `returnValue`, and the number of outcomes bought or sold `numOutcomes`.

### completeSets method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! buyCompleteSets(market, amount):
```
Changed to `buyCompleteSets(market, fxpAmount):`. `amount` has been changed to `fxpAmount` to indicate this is a fixed point value. `buyCompleteSets` is used to buy the `fxpAmount` of shares for each outcome in a `market`. Successfully handling a `buyCompleteSets` will emit a `completeSetsLogReturn` event which creates a log of the transaction. The returnValues are `1` if successful, `0` if the market doesn't exist. `-1` is returned if this is a oracle only branch and therefor trading is disabled. `-2` is returned if the sender doesn't have enough money and `-3` is returned if the sender didn't enter a large enough `fxpAmount`, like if `fxpAmount` was less than or equal to `0`.

```
! sellCompleteSets(market, amount):
```
Changed to `sellCompleteSets(market, fxpAmount):`. Like `buyCompleteSets`, we have renamed the `amount` param to `fxpAmount` to indicate this is a fixed point value. `sellCompleteSets` is used to sell the `fxpAmount` of shares for each outcome in a `market` assuming that we own a position on each outcome less than or equal to the `fxpAmount`. Successfully handling a `sellCompleteSets` will emit a `completeSetsLogReturn` event which creates a log of the transaction. The returnValues are `1` if successful, `-1` if the user doesn't have enough shares to sell, and `-2` if the user entered too small of a `fxpAmount` value. An example of too small of a `fxpAmount` would be a number less than or equal to `0`.

## src/functions/consensus.se

### Data Structure of consensus Contract:
```
event penalize(
  user:indexed,
  outcome,
  oldrep,
  repchange,
  newafterrep,
  p,
  reportValue
)
event consensusLogReturn(
  returnValue
)
```

There is no data structure in the consensus contract but there are two events. One of them is `consensusLogReturn` which simply contains a `returnValue` and is unchanged from it's master iteration aside from a name change from `consensus_logReturn` to `consensusLogReturn`. `penalize` keeps its name the same but it's params have changed. Instead of sender, we have `user` which is set to the `msg.sender`. We no longer record a `branch`, `event`, `penalizedUpTo`, or `timestamp`. `outcome`, `oldrep` which is just the `beforeRep` value for `msg.sender`, `repchange`, `p` (proportionCorrect), and `reportValue` are all still recorded as before. `newafterrep` has been added and is the `afterRep` value for the `msg.sender`.  

### consensus method changes, additions, and removals:

Both `penalizeWrong(branch, event):` and `incrementPeriodAfterReporting(branch):` are unchanged and remain the only exposed methods in the consensus contract. The `penalizeWrong` method is the only method that emits events to create logs.

## src/functions/createBranch.se

### Data Structure of createBranch Contract:
```
data ethContract
```
The only data structure in `createBranch` is the `ethContract` address. This is set during `init()` and should be set to the `ETH` sub currency contract address.

### createBranch method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! createSubbranch(description:str, periodLength, parent, minTradingFee, oracleOnly):
```
Changed to `createSubbranch(description:str, periodLength, parent, fxpMinTradingFee, oracleOnly, mostRecentChildBranch):`. `minTradingFee` has become `fxpMinTradingFee` in order to indicate this is a fixed point value. `mostRecentChildBranch` has been added as a param and is expected to be the most recent child branch address, or 0 for user created branches. This function returns the branch ID created as it's return value if it's successful. Possible errors are `-1` for bad input or non existent parent branch, `-2` is thrown if the branch already exists.

```
+ init():
```
`init` sets the `ethContract` to the `ETH` sub currency address.

## src/functions/createMarket.se

### Data Structure of createMarket Contract:
```
event marketCreated(
  market
)

event tradingFeeUpdated(
  market,
  fxpTradingFee
)
```
The `createMarket` contract has no data structure but keeps it's two event logs that exist in master, `marketCreated` and `tradingFeeUpdated`. Both events have been simplified to include less data in the log that is written when they are triggered. For `createMarket` we only are storing the `marketID` in the log created when previously we would record the `sender`, the `marketID`, the `topic`, the `branch`, the `creationFee`, the `eventBond`, and the creation `timestamp`. `tradingFeeUpdated` has also lost a few values it used to record such as `sender`, `branch`, and `timestamp`. `tradingFee` has been changed to `fxpTradingFee` to indicate this is a fixed point value. `marketID` is now simply `market`.


### createMarket method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! createEvent(branch, description:str, expDate, minValue, maxValue, numOutcomes, resolution: str):
```
Changed to `createEvent(branch, description:str, expDate, fxpMinValue, fxpMaxValue, numOutcomes, resolution: str, resolutionAddress, currency, forkResolveAddress):`. `createEvent` has a couple of renamed params, `minValue` and `maxValue` became `fxpMinValue` and `fxpMaxValue` to indicate they are fixed point values. We have also added a few more fields, `resolutionAddress`, `currency`, and `forkResolveAddress`. `resolutionAddress` is the address of a specific person set to resolve the market, otherwise this field should be `0` if we plan to use random reporters. `currency` is the contract address for the erc20 token we plan to use for the new event. `forkResolveAddress` is the address of a contract that this newly created even should use to fork. If we want to use the regular forking process then this should be `0`.

Possible errors include: `0` for not enough money to pay fees or the `event` already exists. `-1` if we are pasted the `expDate` passed, the `branch` doesn't exist, `description` is bad or the `resolutionAddress` is bad. `-2` if the max value is less than the min value or the market range is less than 1. `-3` if passed an invalid number of outcomes (<2 or >8). `-4` if an `event` would be created that would expire in the last 48 hours of the current `period`. `-5` if the `currency` isn't an approved `branch` currency.

```
! createMarket(branch, tradingFee, event, tag1, tag2, tag3, makerFees, extraInfo:str):
```
Changed to `createMarket(branch, description: str, fxpTradingFee, event, tag1, tag2, tag3, extraInfo: str, currency):`. `createMarket` has had `tradingFee` renamed to `fxpTradingFee` to indicate this is a fixed point value. `makerFees` has been removed. `description` has been added and should be the market question. `currency` was also added and is the erc20 token address of the currency we plan to allow to be used for trading on this market. `createMarket` will write a `marketCreated` log with the market ID as it's `market` value.

Possible errors include: `-1` for bad input or a non existent `event`. `-2` if the `event` has already expired. `-4` if the `market` already exists. `-5` if the currency isn't an approved branch currency. `-6` if there isn't enough money for the market resolution gas cost. It will throw if there isn't enough money to create the market and place the event in the appropriate reporting period.

```
! updateTradingFee(branch, market, tradingFee, makerFees):
```
Changed to `updateTradingFee(market, fxpTradingFee):`. `updateTradingFee` has dropped the `branch` and `makerFees` as params and updated `tradingFee` to `fxpTradingFee` to indicate this is a fixed point value. `updateTradingFee` will write a `tradingFeeUpdated` log if it's successful with the `marketID` and the new `fxpTradingFee` as it's entries. Possible errors are `-1` if the msg.sender isn't the market creator and `-2` if `fxpTradingFee` is invalid because it is too large or too small.

```
+ putEventIntoReportingPeriod(event):
```
`putEventIntoReportingPeriod` is used to add an event to the soonest regular reporting cycle for events that have a defined `resolutionAddress` but that address didn't resolve the `event` within 3 days of expiration. This is used to fall back to regular reporting.

```
+ challengeInitialResolution(event):
```
`challengeInitialResolution` has been added as a way to challenge the result of an `event` resolved by a `resolutionAddress` within 3 days of the resolution. This method will move the event to the soonest possible reporting period in order to be reported on by the regular reporting cycle. This also triggers the payment of the extraBond.

```
+ resolveEarly(event, outcome):
```
`resolveEarly` was added to resolve an `event` before the scheduled expiration of the `event`. The `event` will be set to whatever `outcome` is passed. This must be called by the `resolutionAddress` or else it will fail.

```
+ addFees(market, fxpAmount):
```
`addFees` is used to add more fees, `fxpAmount`, to a `market`. This is done to effectively help pay for oracle usage directly without having to trade using default contracts.

```
- createSingleEventMarket(branch, description:str, expDate, minValue, maxValue, numOutcomes, resolution:str, tradingFee, tag1, tag2, tag3, makerFees, extraInfo:str):
```

## src/functions/eventResolution.se

### Data Structure of eventResolution Contract:

`eventResolution` doesn't have a data structure or events of it's own.

### eventResolution method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! resolveCategoricalOrScalar(scaled_min, scaled_max, event, market, branch, votingPeriodEvent):
```
Changed to `resolveCategoricalOrScalar(fxpScaledMin, fxpScaledMax, event, market, branch, votingPeriodEvent):`. In the develop version of `resolveCategoricalOrScalar` the params `scaled_min` and `scaled_max` have been renamed to `fxpScaledMin` and `fxpScaledMax` to indicate these should be fixed point values.

## src/functions/faucets.se

Faucets is unchanged.


## src/functions/forking.se

### Data Structure of forking Contract:

`forking` doesn't have a data structure of it's own.

### forking method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! move_event(event):
```
Renamed to `moveEvent(event):`.

```
! resolve_fork(branch):
```
Renamed to `resolveFork(branch):`.

## src/functions/forkPenalize.se

### Data Structure of forkPenalize Contract:

`forkPenalize` doesn't have a data structure of it's own.

### forkPenalize method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
- proportionCorrectForkEvent(event):
```

## src/functions/makeReports.se

### Data Structure of makeReports Contract:
```
event makeReportsLogReturn(
  returnValue
)
```
The `makeReports` contract doesn't have a data structure and now only contains one event for writing a log: `makeReportsLogReturn`. `makeReportsLogReturn` has been renamed from the old event `makeReports_logReturn`. `submittedReportHash` and `submittedReport` events have been removed from this contract.

### makeReports method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! makeHash(salt, report, eventID, sender):
```
Changed to `makeHash(salt, fxpReport, event, sender, ethics):`. `report` has been renamed to `fxpReport` to indicate this is a fixed point value. `eventID` has been renamed to simply `event`. `ethics` has been added to the hash.

```
! submitReportHash(event, reportHash, encryptedReport, encryptedSalt, ethics):
```
Changed to `submitReportHash(event, reportHash, saltyEncryptedHash):`. Now to submit a report hash we only need to pass `event`, `reportHash`, and the `saltyEncryptedHash`. `setSaltyEncryptedHash` is the result of `makeHash`.

```
! submitReport(event, salt, report, ethics):
```
Changed to `submitReport(event, salt, fxpReport, fxpEthics):`. The params `report` and `ethics` have been renamed to `fxpReport` and `fxpEthics`. This was done to indicate they should be fixed point values.

```
! validateReport(eventID, branch, votePeriod, report, forkedOverEthicality, forkedOverThisEvent, roundTwo, balance):
```
Changed to `validateReport(event, branch, votePeriod, fxpReport, forkedOverEthicality, forkedOverThisEvent, weight):`. `eventID` has been renamed to just `event`. `roundTwo` and `balance` have been removed from the arguments. `weight` has been added as an input which is set to `1` in the `submitReport` function.

## src/functions/offChainTrades.se

### Data Structure of offChainTrades Contract:
```
event Order(
  tokenX:address:indexed,
  orderSize,
  tokenY:address:indexed,
  fxpPrice,
  expiration,
  user:address:indexed
)

event Cancel(
  tokenX:address:indexed,
  orderSize,
  tokenY:address:indexed,
  fxpPrice,
  expiration,
  user:address:indexed,
  v,
  r,
  s
)

event Trade(
  maker:address:indexed,
  taker:address,
  tokenX:address:indexed,
  tokenY:address:indexed,
  amount,
  fxpPrice,
  tradeHash
)

data amountFilled[<orderHash>]
```

`offChainTrades.se` has been added to the contracts in `develop`. It has three events and one data structure. `amountFilled` is indexed by the `orderHash` of an off chain order. It contains the amount of shares an order has filled so far. `Order` is an event that writes a log whenever `onChainOrder` is called successfully. It's used to store a record of an off chain order being placed. It stores the address of the token we plan to sell as `tokenY`. The amount of `tokenY` to sell is defined as `orderSize`. The address for the token we plan to buy is `tokenX`. The exchange rate of `tokenY` for each `tokenX` is defined by `fxpPrice`. The `expiration` for the order and the `user` who placed the order are also recorded.

`Cancel` is called when an order is successfully canceled by the `user` who posted the order. `Cancel` records the `tokenY` address, the amount of `tokenY` as `orderSize`, the address for the `tokenX` accepted for `tokenY` and the rate of exchange (`fxpPrice`) one `tokenX` is worth to `tokenY`. It also records `expiration` of the order and the `user` who posted the order originally. This is all similar to `Order` however `Cancel` also requires the `v`, `r`, and `s` fields of the `user` who placed the order in order to verify that the order was sent by `user`.

`Trade` is called when part or all of an order has been filled successfully. Trade records the `maker` address of the person who placed the order, the `taker` address of the person who wants to fill the order, the token address (`tokenX`) used by the `taker` to fill the order, the token address (`tokenY`) of the `maker`'s order to be filled, the `amount` to fill, the rate (`fxpPrice`) at which to fill `tokenY` for each `tokenX`, and the `tradeHash`.

### offChainTrades method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
+ trade(maker: address, v, r, s, tokenX: address, tokenY: address, orderSize, takerAmount, fxpPrice, expiration):
```
`trade` is called to attempt to fill some or all of an order. It takes a `maker` address, which is the person who placed the order on the book. It takes the `v, r, s` values in order to verify the `orderHash` was signed by `maker`. It also requires details about the order such as `tokenX` (token to be bought), `tokenY` (token to be sold), orderSize which indicates the amount of `tokenY` available in the order, the `fxpPrice` exchange rate of `tokenY` to each `tokenX` and the `expiration` of the order. `takerAmount` is the amount of token Y we intend to fill of the order. This will record a `Trade` log if successful and return `1`.

Possible errors are: `0` if the orderHash can't be verified. `-1` if the order has expired. `-2` if the `maker` or `taker` doesn't have enough money or isn't authorized to spend enough money to fill the request. `-3` if the `takerAmount` exceeds the amount left available on the order.

```
+ onChainOrder(tokenX: address, orderSize, tokenY: address, fxpPrice, expiration):
```
`onChainOrder` is called to generate an `Order` log and returns the `orderHash` for the order placed. It requires a `tokenX` to be accepted as payment, the amount of `tokenY` to sell as `orderSize`, the `tokenY` we plan to sell, the rate at which we will sell `tokenY` for one `tokenX` as `fxpPrice` and the `expiration` of this order.

```
+ cancelOrder(tokenX: address, orderSize, tokenY: address, fxpPrice, expiration, v, r, s):
```
`cancelOrder` is used to do exactly that. After placing an order, if the user who placed the order has decided to cancel the order and remove it from the order book before it's been filled then they would call this method. `cancelOrder` requires the `tokenX` we planned to accept, the amount of `tokenY` we plan to remove from the order book as `orderSize`, the `tokenY` we had planned to sell, the rate at which we would have exchanged `tokenY` for one `tokenX` as `fxpPrice`, and the `expiration` of the order. We are also required to pass the `v, r, s` values in order to confirm that the `msg.sender` of this request is the user who placed the order on the book. If successful in canceling the order, a `Cancel` log will be written to record the cancelation and a `1` will be returned. This will throw an error if the `msg.sender` isn't the person who created the order.

```
+ getAmountFilled(orderHash):
```
`getAmountFilled` returns the amount of an order already filled given an `orderHash`. If none of the order has been filled this will return `0`.

## src/functions/oneWinningOutcomePayouts.se

### Data Structure of oneWinningOutcomePayouts Contract:

`oneWinningOutcomePayouts` has no data structure and no events are defined in the contract. This contract is used to pay out markets that have only 1 winning outcome, so binary markets and categorical markets.

### oneWinningOutcomePayouts method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
+ oneOutcome(market, winningOutcome, sender, categoricalPointFive, numOutcomes):
```
`oneOutcome` is used to payout a market with only one winning outcome, like a binary market as an example. It requires a market ID `market`, the winning outcome value `winningOutcome`, the `sender` address who will be getting the payout, `categoricalPointFive` which is a boolean to indicate if this is an indeterminate categorical market in which all outcomes should get a payout evenly, and finally the number of outcomes `numOutcomes`. Returns `1` if successful.

## src/functions/penalizationCatchup.se

### Data Structure of penalizationCatchup Contract:

`penalizationCatchup` has no data structure and no events are defined in the contract. The `penalizationCaughtUp` event is no longer a part of this contract and no such log is written when calling `penalizationCatchup`.

### penalizationCatchup method changes, additions, and removals:

```
penalizationCatchup(branch, sender):
```
`penalizationCatchup` is the only function callable in this contract. It hasn't changed it's signature from the master incarnation.

## src/functions/penalizeNotEnoughReports.se

### Data Structure of penalizeNotEnoughReports Contract:

`penalizeNotEnoughReports` has no data structure and no events are defined in the contract.

### penalizeNotEnoughReports method changes, additions, and removals:

```
proveReporterDidntReportEnough(branch, reporter, eventExample):
```
`proveReporterDidntReportEnough`'s signature is unchanged and still takes a `branch`, `reporter` address, and event ID `eventExample` to check and penalize a `reporter` if they haven't reported enough. This will return 1 if successful. Possible errors include: `0` if the branch or `eventExample` doesn't exist, `-1` if penalization for not reporting is already complete, `-2` if this is called but we aren't in the correct part of the period, `-3` if the `eventExample` isn't in the `branch`, and `-5` if the `reporter` has reported enough and doesn't need to be penalized.

## src/functions/roundTwo.se

### Data Structure of roundTwo Contract:

`roundTwo` has no data structure and no events are defined in the contract.

### roundTwo method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! roundTwoPostBond(branch, event, eventIndex, votePeriod):
```
Changed to `roundTwoPostBond(branch, event, eventIndex, overEthicality):`. We still need `branch`, `event`, and `eventIndex` but instead of `votePeriod` we now require `overEthicality`. `overEthicality` is a boolean indicating wether the the bond is being posted in order to dispute an `event` over it's ethics or not. This function will return `1` if successful. Will throw if `msg.value` is not enough to cover the cost of the resolution or the bond isn't large enough. Possible errors include: `0` for an invalid branch or period, `-1` if the event was pushed forward in which case posting a bond isn't allowed during the event being moved, `-2` if the branch is in middle of a fork, `-3` if there is not enough money in `msg.value`, `-4` if the `event` has no votes.

## src/functions/roundTwoPenalize.se

### Data Structure of roundTwoPenalize Contract:

`roundTwoPenalize` has no data structure and no events are defined in the contract.

### roundTwoPenalize method changes, additions, and removals:

```
penalizeRoundTwoWrong(branch, event):
```
`penalizeRoundTwoWrong` is the only function callable in this contract. It's signature is unchanged.

## src/functions/sendReputation.se

### Data Structure of sendReputation Contract:
```
data amountCanSpend[<address(owner)>][<address(spender)>](
  branch[<branch>]
)

event Transfer(
  from:indexed,
  to:indexed,
  value
)

event Approval(
  owner:indexed,
  spender:indexed,
  branch:indexed,
  value
)
```
`sendReputation` has one data structure defined and two events defined which have had renamed params. `amountCanSpend` is a multi-dimensional array indexed by the address of an `owner` of an account and the address of the approved `spender` for that owned account. Inside is a `branch` array indexed by a branch ID with fixed point values for the amount of dormant `REP` an approved `spender` is allowed to withdraw from the `owner`'s account.

The `Transfer` event writes a `Transfer` log whenever a successful call to `sendRepFrom` or `transferFrom` is executed. The `Transfer` log stores the `from` address for a transfer of `REP`, the `to` address, and the `value` amount of `REP` which should be fixed point.

The `Approval` event writes an `Approval` log whenever a successful call to `approve` is executed. The `Approve` log stores information about which `owner` account approved which `spender` account to withdraw an `value` amount of dormant `REP` from their account on a specific `branch`.

### sendReputation method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! transferFrom(branch, from, recver, value):
```
Changed to `transferFrom(branch, from, receiver, fxpValue):`. The only real change here is the renamed params: `recver` became `receiver` and `value` has become `fxpValue` to help indicate this should be a fixed point value.

```
! approve(branch, spender, value):
```
Changed to `approve(branch, spender, fxpValue):`. The only change is the `value` param has become `fxpValue` to indicate it should be a fixed point value.

```
! allowance(owner, spender):
```
Changed to `allowance(branch, owner, spender):`. This function now takes a `branch` param in addition to the `owner` and `sender`.

```
! convertToDormantRep(branch, value):
```
Changed to `convertToDormantRep(branch, fxpValue):`. The `value` param has been renamed to `fxpValue` to indicate this is a fixed point value.

```
! convertToActiveRep(branch, value):
```
Changed to `convertToActiveRep(branch, fxpValue):`. The `value` param has been renamed to `fxpValue` to indicate this is a fixed point value.

```
+ claimRep():
```
`claimRep` is used to claim a `msg.sender`'s `REP` and add them to the reporting pool.

```
+ sendRepFrom(branch, from, receiver, fxpValue):
```
`sendRepFrom` is used to send a fixed point amount (`fxpValue`) of active `REP` from one address (`from`) to another address (`receiver`) on a certain `branch`.
Returns the `fxpValue` of sent `REP` if successful. Possible errors include: `0` if we are one period behind and need to do penalization, `-1` the `from` account doesn't have any rep, `-2` wrong part of period, `-3` the `to` address doesn't exist, `-4` is couldn't catch up automatically, `-5` not enough rep, fxpValue is too small, or not authorized to send `REP`. Finally we have `-6` if a person reported on a round 2 event before it was in the second round [i.e. the first reporting backstop], they cannot convert their rep to dormant or send rep until they've finished the resolution process for that round 2 event.

```
- sendReputation(branch, recver, value):

- transfer(branch, recver, value):
```

## src/functions/shareTokens.se

### Data Structure of shareTokens Contract:
```
data accounts[2**160<address>](
  balance,
  spenders[2**160<address>](
    maxValue
  )
)

data totalSupply

data name

data symbol

data decimals

event Transfer(
  from:indexed,
  to:indexed,
  value
)

event Approval(
  owner:indexed,
  spender:indexed,
  value
)
```
The `shareTokens` contract is a erc20 token wrapper for shares. There are five data structures and two events defined in the `shareTokens` contract. Four of the data structures are simple and self explanitory, they are `totalSupply`, `name`, `symbol`, and `decimals`. `totalSupply` is the total supply of shares in the contract, `name` is the name of the token, which is `"Shares"` by default, the symbol is set to `"SHARE"` and the `decimals` is the amount of decimal places each Share has, in this case `18`. There is also an `accounts` array indexed by account addresses. Within the `accounts` array is a `balance` of shares for that account, and an array of authorized `spenders` indexed by account addresses. Inside of the `spenders` array is the maximum value (`maxValue`) of how much a spender is authorized to withdraw from the owner of the account.

The `Transfer` event creates a `Transfer` log whenever a successful call to `transfer` or `transferFrom` is made. It records the person who sent the shares ('from'), the person who recieved the shares ('to'), and the amount of shares sent ('value').

The `Approval` event creates an `Approval` log whenever a successful call to `approve` is made. `Approval` records the `owner` of the account, the person authorized to spend (`spender`), and the amount they are authorized to spend (`value`).


### shareTokens method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
+ init():
```
`Init` sets the `name`, `symbol`, and `decimals` data structures to `"Shares"`, `"SHARE"`, and `18` respectively.

```
+ transfer(to: address, value: uint256):
```
`transfer` will transfer an amount (`value`) of shares from the `msg.sender`'s account to the `to` address specified. This will throw if there is not enough shares to send, the `value` is less than 0, or the `to` address doesn't exist.

```
+ transferFrom(from: address, to: address, value: uint256):
```
`transferFrom` is similar to `transfer` except that we must specify the `from` address instead of using the `msg.sender` address. It can fail for the same reasons as `transfer` as well as if the `from` address doesn't exist.

```
+ approve(spender: address, value: uint256):
```
`approve` is used to authorize a `spender` address to withdraw up to a specific `value` of shares of a `msg.sender`'s account.

```
+ allowance(owner: address, spender: address):
```
`allowance` is used to check how many shares a `spender` is authorized to withdraw from a specific `owner`.

```
+ totalSupply():
```
`totalSupply` returns the value of the `totalSupply` data structure, which is the number of shares total.

```
+ balanceOf(address: address):
```
`balanceOf` is used to get the amount of shares owned by a specific `address`.

```
+ getName():
```
`getName` returns the value of the `name` data structure, which is defaulted to `"Shares"`.

```
+ getDecimals():
```
`getDecimals` returns the value of the `decimals` data structure, which is defaulted to `18`.

```
+ getSymbol():
```
`getSymbol` returns the value of the `symbol` data structure, which is defaulted to `"SHARE"`.

```
+ changeTokens(trader, amount):
```
`changeTokens` is used to add an `amount` of tokens to the balance of a `trader` assuming that their current balance + `amount` isn't less than 0. This function is to be whitelisted.

```
+ modifySupply(amount):
```
`modifySupply` is used to add or subtract from the total supply of shares by a given `amount`. `amount` should be negative to subtract shares, positive to increase shares. This function is to be whitelisted.

## src/functions/slashRep.se

### Data Structure of slashRep Contract:
`slashRep` has no data structure or events of it's own.

### slashRep method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! slashRep(branch, salt, report, reporter, eventID):
```
Changed to `slashRep(branch, salt, report, ethics, reporter, event):`. The params have changed with the new slashRep in development, now in addition to the previous params we require the `ethics` of the event. Also the param `eventID` has been renamed to just `event`. This will return 1 if successful and the `reporter` will have lost their `REP` balance, with half going to the `msg.sender` of the `slashRep` method and the other half going to the `branch`.

Possible errors include: `-1` if it's an invalid votePeriod, `-2` if we are past the date of reveal, `-3` for invalid claims or `reportHash`, users should make sure that the colluding `reporter`'s commit is the same as the params passed to `slashRep`, `-4` if the event isn't in the branch specified, and `-5` if already slashed.

## src/functions/trade.se

### Data Structure of trade Contract:
```
event logPrice(
  market:indexed,
  sender:indexed,
  owner:indexed,
  type,
  price,
  amount,
  timestamp,
  orderID,
  outcome,
  askerSharesFilled,
  askerMoneyFilled,
  bidderSharesFilled,
  bidderMoneyFilled
)

event tradeLogReturn(
  returnValue
)

event tradeLogArrayReturn(
  returnArray: arr
)
```
There is no data structure for the `trade` contract and it now has only three events defined. The `tradeLogReturn` and `tradeLogArrayReturn` methods are the same as in master currently except they have been renamed from `trade_logReturn` and `trade_logArrayReturn` respectively. `log_fill_tx` and `log_short_fill_tx` have been removed and replaced with `logPrice`. `logPrice` records the `market` to the trade was placed on, the `sender` of the request to take an order, the `owner` who is the address who placed the order on the books, type is the `type` of trade (bid/ask), `price` per share, `amount` of shares filled, `timestamp` of the trade, `orderID` of the order, the `outcome` it's trading on, `askerSharesFilled` and `askerMoneyFilled` are the amount of shares and money filled by an asker, `bidderSharesFilled` and `bidderMoneyFilled` are the amount of shares and money filled by a bidder. This log is written when a bid or ask order is successfully filled or partially filled.

### trade method changes, additions, and removals:
```
Key : description
!   : Modified method
-   : removed method
+   : added method
```
*Any function not explicitly mentioned is unchanged from it's current master iteration. When a function is changed, first I will show the old signature that's currently in place in master, then outside of the code preview I will indicate the new signature and why.*

```
! trade(max_value, max_amount, trade_ids:arr, tradeGroupID):
```
Changed to `trade(orderID, amountTakerWants):`. trade is used to pick up or take an order off the order book. It now requires just an `orderID` for the order we intend to take and the `amountTakerWants` is simply the number of shares to take from that order. Possible errors include: `0` if the order doesn't exist, `-1` if the order hash is bad, `-2` if the `amountTakerWants` isn't at least 0.00000001, `-3` if you try to take your own order, `-4` if you have an insufficient balance to complete the request, `-5` if the order hasn't been mined yet (same block).

```
- short_sell(buyer_trade_id, max_amount, tradeGroupID):
```

## src/functions/tradeAvailableOrders.se

### Data Structure of tradeAvailableOrders Contract:
```
event tradeAvailableOrdersLogArrayReturn(
  returnArray: arr
)
```
The `tradeAvailableOrders` contract has no data structure of it's own but does define an event. `tradeAvailableOrdersLogArrayReturn` event writes a log with the `returnArray` when `tradeAvailableOrders` is successfully called.

### tradeAvailableOrders methods:

```
+ tradeAvailableOrders(orderIDs: arr, amountTakerWants, fxpMinimumOrderBalance):
```
`tradeAvailableOrders` is used to fill the first order that fits the params passed to it. Given an array of `orderIDs`, this method will loop through the array and fill the `amountTakerWants` with the first order that is at or above the `fxpMinimumOrderBalance`. So if I have an array of orders and I want to buy 5 shares (`amountTakerWants`), and put the `fxpMinimumOrderBalance` as `5`, then the first order in the Orders array that has at least `5` shares to sell will be filled by my request. If this is successful it will write a `tradeAvailableOrdersLogArrayReturn` log with an array containing the response code (`1` for successful), the unfilled value, and the unfilled amount. If there are no orders in the orderIDs array that meet the critiera then the `tradeAvailableOrdersLogArrayReturn` log will be written with an array containing the value `-6` to indicate that there was no order available that was able to be filled.

# Ignore the below please.
*Please ignore everything below this line as not part of the change log, simply some notes for upcoming updates to the change log.*
