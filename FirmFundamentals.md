
# Get firm fundamentals

## Introduction

This process is for obtaining firm fundamentals from vendor Financial Modeling
Prep henceforth referred to as FMP.

The computation is based on the wonderful article written by [Camilo
Cisera](https://codingandfun.com/piotroski-f-score/). Camilo has gone over the
API used and computation but has used an undocumented API call that does not
exist in FMP documentation anymore.

## API Calls

~~We will be using the following API calls to obtain the firm financial data:~~

| ~~Statement~~              | ~~API endpoint~~                                                                                                |
|------------------------|-------------------------------------------------------------------------------------------------------------|
| Income Statement       | https://financialmodelingprep.com/api/v3/income-statement/{firm}?limit=13&apikey=demo&period=quarter        |
| Balancesheet Statement | https://financialmodelingprep.com/api/v3/balance-sheet-statement/{firm}?limit=13&apikey=demo&period=quarter |
| Cashflow Statment      | https://financialmodelingprep.com/api/v3/cash-flow-statement/{firm}?limit=13&apikey=demo&period=quarter     |

{~~firm} will be replaced with the ticker that we are going to request. We will
limit to 13 quarters of data as we will be using only two years of data (8
quarters - getting a year and a quarter of additional data for no valid reason
at this time).~~

## Obstacles

~~During the proof-of-concept phase, it was observed that FMP limits concurrent
calls. Also, the FMP limits the number of calls one can make, and the account
used has a limit of 250 calls a day.~~

~~To obtain the fundamentals the application needs to make 3 calls per
institution. Based on observation it has been decided that we will limit to
**two** firms for each invocation of the application, i.e. the application will
make six API calls each time it is invoked. If we are going to invoke the
application 75 times a day, then we can obtain the fundamentals of 150 firms a
day.~~

~~The business logic needs us to compute the fundamentals for firms that are
included in the S&P500 and NASDAQ-100 and there are many overlapping industries.
The combined list is usually around 525 to 550 and even during busy earning
seasons of quarter ends we can catch-up within a week.~~

## Revision

Ideally, I should have removed sections that were found obsolete and replaced
them with the current thought process. Instead, this document, at least for now,
maintains the old thought process (marked that it's not valid anymore) and
current process.

## API Calls

As indicated earlier Camilo had used an undocumented API call which

1.  The vendor might stop providing data or worse continue the API but provide
    obsolete values.

2.  FMP does not provide data for more than 5 quarters for the signed up rank.
    Though the endpoint Camilo provided has all data we need it might be turned
    off and our calculations cannot be performed with 5 quarters filing details.

Hence it was decided to switch vendor and it has been decided to use data from
[Alpha Vantage](https://www.alphavantage.co/). Though vendor has different
limitations on the number of API calls one can make their commitment to refresh
data on the same day a company reports its latest earnings and financials is
promising and delayed acquisition of filing data will not be required.

API calls used to obtain firm financial data:

| Statement               | API Endpoint                                                                                                                                                                |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Income Statement        | [https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol={firm}&apikey=demo](https://www.alphavantage.co/query?function=INCOME_STATEMENT&symbol=IBM&apikey=demo) |
| Balance sheet Statement | [https://www.alphavantage.co/query?function=BALANCE_SHEET&symbol={firm}&apikey=demo](https://www.alphavantage.co/query?function=BALANCE_SHEET&symbol=IBM&apikey=demo)       |
| Cash flow Statement     | [https://www.alphavantage.co/query?function=CASH_FLOW&symbol={firm}&apikey=demo](https://www.alphavantage.co/query?function=CASH_FLOW&symbol=IBM&apikey=demo)               |

{firm} will be replaced with the ticker we are going to request. All queries to
Alpha Vantage go through a single endpoint *query* and actual query
diversification is made using API Parameter **function.**

## Obstacles

Alpha Vantage has the following limits:

-   Each API key can make only up to 5 API requests per minute.

-   The service limit will limit the API key used only up to 500 times a day.

To obtain fundamentals the application needs to make 3 calls per institution. To
ensure we do not hit limits it has been decided to limit data fetch to 60 firms
a day. This will ensure the application will update all covered firms (index
firms) within 10 days after its filing has been done.

## Procedure

-   The application on startup will read the collection 'EarningsCalendar' for
    documents that have the field 'EarningsRetrieved' marked false and the field
    'Date' earlier than the actual run day.

-   The application will select two firms that are included in the collection
    ‘SlickChartFirmNames’; this collection is refreshed periodically. It lists
    all firms that are included in S&P 500 and NASDAQ100.

-   Call Alpha Vantage API to obtain income, balance sheet, and cash flow
    statements. Calls to Alpha Vantage will be done synchronously rather than
    asynchronous. This will limit the number of simultaneous connections the
    application will make to Alpha Vantage.

-   Values obtained from Alpha Vantage will be stored in the database in
    collections ‘BalanceSheet’, ‘CashFlow’, and ‘IncomeStatement’.

    -   Any previous entries for the firms obtained will be purged.

-   ‘EarningsCalendar’ will be updated; the field ‘EarningsRetrieved’ will be
    updated from false to true for the firms that have been processed.

-   Any record that is older than a one-quarter year will be purged.

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTA4ODQ0MjQ5OV19
-->