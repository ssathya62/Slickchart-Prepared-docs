
# Get firm fundamentals

## Introduction

This process is for obtaining firm fundamentals from vendor Financial Modeling Prep henceforth refered to as FMP.

  

The computation is based on the wonderful article written by [Camilo Cisera](https://codingandfun.com/piotroski-f-score/). Camilo has gone over the API used and computation but has used an undocumented API call that doesn't exist in FMP documentation anymore.

We will be using the following API calls to obtain the firm financial data:

  
| Statement |API endpoint  |
|--|--|
| Income Statement |https://financialmodelingprep.com/api/v3/income-statement/{firm}?limit=13&apikey=demo&period=quarter  |
|Balancesheet Statement|https://financialmodelingprep.com/api/v3/balance-sheet-statement/{firm}?limit=13&apikey=demo&period=quarter|
|Cashflow Statment|https://financialmodelingprep.com/api/v3/cash-flow-statement/{firm}?limit=13&apikey=demo&period=quarter|


  

{firm} will be replaced with the ticker that we are going to request. We will limit to 13 quarters of data as we will be using only two years of data (8 quarters - getting a year and a quarter of additional data for no valid reason at this time).

  

The application on startup will read the collection 'EarningsCalendar' for documents that have the field 'EarningsRetrieved' marked false and the field 'Date' earlier than the actual run day. While performing proof-of-concept it was observed that FMP limits concurrent calls.
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg0OTEzNDE4NSwtNDAzNDU2NTM0LDE4OD
IwMjI5NjVdfQ==
-->