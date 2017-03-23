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
Key : description
!   : Modified function
-   : removed function
+   : added function
->  : function moved FROM another contract
<-  : function moved TO another contract

Any function not explicitly mentioned is unchanged from it's current master iteration.

#### DataStructure of Contract:
data roundTwo[](roundTwo, originalVotePeriod, originalOutcome, originalEthicality, final, bondPoster, bondReturned, bondPaid, refund,  
*disputedOverEthics*)
  disputedOverEthics was added, it's a boolean.

data forking[](bondPoster, bondAmount, forkedOverEthicality, bondPaid, originalBranch, moved)

data resolved[][]

+ getDisputedOverEthics(event):
    returns the disputedOverEthics bool for event
+ setDisputedOverEthics(event):
    sets the passed in event's disputedOverEthics bool to 1

## src/data_api/branches.se
Key : description
!   : Modified function
-   : removed function
+   : added function
->  : function moved FROM another contract
<-  : function moved TO another contract

Any function not explicitly mentioned is unchanged from it's current master iteration.

- initDefaultBranch():
! setInitialBalance(branch, period, balance):
  Changed: setInitialBalance(branch, period, balance, currency) Where currency is the address of the erc20 token you want to set the initial balance of in the specified branch and period.
! getInitialBalance(branch, period):
  Changed: getInitialBalance(branch, period, currency)
    currency is the erc20 token address of the token you plan to get the intial balance of.
- getMarketsInBranch(branch):
- getSomeMarketsInBranch(branch, initial, last):
! initializeBranch(ID, currentVotePeriod, periodLength, minTradingFee, oracleOnly, parentPeriod, parent):
  Changed: initializeBranch(ID, currentVotePeriod, periodLength, fxpMinTradingFee, oracleOnly, parentPeriod, parent, contract, wallet, mostRecentChild):

+ getForkTime(branch)
    given a BranchID, this will return the timestamp for when this branch was forked. if it hasn't been forked it returns 0.

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
