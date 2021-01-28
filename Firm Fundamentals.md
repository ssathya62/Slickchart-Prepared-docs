
# Get firm fundamentals

## Introduction

This process is for obtaining firm fundamentals from vendor Financial Modeling
Prep henceforth referred to as FMP.

The computation is based on the wonderful article written by [Camilo
Cisera](https://codingandfun.com/piotroski-f-score/). Camilo has gone over the
API used and computation but has used an undocumented API call that does not
exist in FMP documentation anymore.

## API Calls 

We will be using the following API calls to obtain the firm financial data:

| Statement              | API endpoint                                                                                                |
|------------------------|-------------------------------------------------------------------------------------------------------------|
| Income Statement       | https://financialmodelingprep.com/api/v3/income-statement/{firm}?limit=13&apikey=demo&period=quarter        |
| Balancesheet Statement | https://financialmodelingprep.com/api/v3/balance-sheet-statement/{firm}?limit=13&apikey=demo&period=quarter |
| Cashflow Statment      | https://financialmodelingprep.com/api/v3/cash-flow-statement/{firm}?limit=13&apikey=demo&period=quarter     |

{firm} will be replaced with the ticker that we are going to request. We will
limit to 13 quarters of data as we will be using only two years of data (8
quarters - getting a year and a quarter of additional data for no valid reason
at this time).

## Obstacles

During the proof-of-concept phase, it was observed that FMP limits concurrent
calls. Also, the FMP limits the number of calls one can make, and the account
used has a limit of 250 calls a day.

To obtain the fundamentals the application needs to make 3 calls per
institution. Based on observation it has been decided that we will limit to
**two** firms for each invocation of the application, i.e. the application will
make six API calls each time it is invoked. If we are going to invoke the
application 35 times a day, then we can obtain the fundamentals of 70 firms a
day.

The business logic needs us to compute the fundamentals for firms that are
included in the S&P500 and NASDAQ-100 and there are many overlapping industries.
The combined list is usually around 525 to 550 and even during busy earning
seasons of quarter ends we can catch-up quickly.

## Procedure

-   The application on startup will read the collection 'EarningsCalendar' for
    documents that have the field 'EarningsRetrieved' marked false and the field
    'Date' earlier than the actual run day.

-   The application will select two firms that are included in the collection
    ‘SlickChartFirmNames’; this collection is refreshed periodically. It lists
    all firms that are included in S&P 500 and NASDAQ100.

-   Call FMP API to obtain income, balance sheet, and cash flow statements.
    Calls to FMP will be done synchronously rather than asynchronous. This will
    limit the number of simultaneous connections the application will make to
    FMP.

-   Values obtained from FMP will be stored in the database in collections
    ‘BalanceSheet’, ‘CashFlow’, and ‘IncomeStatement’.

    -   Any previous entries for the firms obtained will be purged.

-   ‘EarningsCalendar’ will be updated; the field ‘EarningsRetrieved’ will be
    updated from false to true for the firms that have been processed.

-   Any record that is older than a one-quarter year will be purged.

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTM0MjUxNjkxMSwtODQ5MTM0MTg1LC00MD
M0NTY1MzQsMTg4MjAyMjk2NV19
-->
