# Piotroski F-Score Computation
The Piotroski Score is a discrete score between zero and nine that reflects nine criteria used to determine the strength of a firm's financial position. The Piotroski Score is used to determine the best value stocks, with nine being the best and zero being the worst.
This score is computed using the company’s accounting results in recent periods (years). For every criterion met (noted below), one point is awarded; otherwise, no points are awarded. The points are then added up to determine the best value stocks.

### Basic Outline of Scoring
>* The score is a ranking between zero and nine that incorporates nine factors that speak to a firm’s financial strength.
>* The scale was created by Joseph Piotroski
>* The nine aspects are based on accounting results over many years; a point is awarded each time a standard is met, resulting in an overall score.
>* If a company has a score of eight or nine, it is considered a good value. If a company has a score of between zero and two points, it not a good value.
#Scoring matrix

The score is broken down into the following categories:

1.	Profitability
I.	Positive net income
II.	Positive return on assets in the current year.
III.	Positive operating cash flow in the current year.
IV.	Cash flow from operations being greater than net income.

2.	Leverage, liquidity, and source of funds
I.	Lower ratio of long term debt in the current period, compared to the previous year.
II.	Higher current ratio this year compared to the previous year.
III.	No new shares were issued in the last year.

3.	Operating efficiency
I.	A higher gross margin compared to the previous year
II.	A higher asset turnover ratio compared to the previous year.
### Computation
The values can be computed by the application but a vendor, Simfin, provides the computed values, and when we can leverage external compute why not leverage it? Simfin currently covers around 3,000 firms and does include most of the listed firms included in S&P500 and NASDAQ. But it does miss some of the firms and we need to compute the values. To compute the values we need to obtain the underlying data, balance sheet, cash flow, and income statements from its 10-K and 10-Q filings.  We can get these underlying data from Financial Modeling Prep. 
#### Proposed changes
•	The application is currently trying to obtain the Piotroski score only for the top 100 firms by dollar volume. The proposed change is to obtain the score for all firms; the combined list between the two indexes will be around 530 to 550 securities.
•	Piotroski score is currently obtained every day by the application. The score will change only based on filings; i.e. only once a quarter. Hence the score for each firm will be obtained/computed only 4 times a year.
•	To fulfill the above change we need to obtain the earnings calendar for the firms listed in indexes. This data can be obtained from Finnhub Stock API.
•	Underlying filing data, Balance-sheet, Cash-flow, and Income-statement, can be obtained from Financial Modeling Prep.
#### Implementation Details
##### List of Index Securities
Index securities are currently obtained from an external vendor SlickChar and existing feed and processed results can be leveraged.
##### Earnings Calendar
The earnings calendar will be obtained from Finnhub every day for a window of two months; one month prior and one month looking forward. The values obtained from the vendor will be upserted(insert/updated) to the database. 
Finnhub’s earnings calendar have the following fields:
>* Date: (Reporting date)
>* EPS Actual
>* EPS Estimate
>* Hour: (dmh/amc/bmo)
>* Quarter:
>* Revenue Actual
>* Revenue Estimate
>* Symbol
>* Year

These values will be stored in the database but an additional field processing date will be added while storing into the database. Also, care will be taken that only one record exists per symbol in the database.
##### Clean up
Any record that has Reporting date aged more than 100 calendar days will be purged from the collection. The reasoning is every firm is mandated to do its quarterly filing as long as it is listed in NYSE/NASDAQ and at least provide the next reporting date at least 30 days in advance. Hence the assumption that a reporting date is older than 100 calendar days is a stale record is valid.
#### Obtain Financials
This process will be the one that will obtain indexes firm’s financials. Due to the limitation imposed by the vendor, the process will be limited to process not more than two securities at a time. To satisfy vendor requirements and business needs the application will run around 40 times a day (40 * 2 * 3 = 240 calls). 
The application first checks the earning calendar for all firms that have already reported (reporting date less than run date) and does not have the processing date set. If the firm(s) are included in index securities the application will bring its financials and store the values in the database. 
##### Clean up
Any record that is older than three years can be considered as aged and be purged. Also, any record that is referring to a symbol that is not included in the index securities can be marked as inactive.
#### Compute values
This process is the one that will compute the values. The following fields are proposed to be stored in rating collection:

|Symbol|Vendor Rating  |Vendor fetch date  |Computed Rating  |Compute Date  |Piotroski values as enumerable  |
|--|--|--|--|--|--|


_Code that was developed for POC._

```cs
public int GetScore()
		{
			if (balanceSheet == null ||
				balanceSheet.Financials == null ||
				balanceSheet.Financials.Count < LookbackTime ||
				incomeStatement == null ||
				incomeStatement.Financials == null ||
				incomeStatement.Financials.Count < LookbackTime ||
				cashFlowStatement == null ||
				cashFlowStatement.Financials == null ||
				cashFlowStatement.Financials.Count < LookbackTime)
			{
				return 0;
			}
			GetValuesFromBalanceSheet(out decimal longTermDebt, out decimal totalAssets,
				out decimal currentAssets, out decimal currentLiabilities,
				out decimal py_longTermDebt, out decimal py_totalAsseets,
				out decimal py_currentAssets, out decimal py_currentLiabilities,
				out decimal py2_totalAsseets);

			//Income statement
			GetValuesFromIncomeStatement(out decimal curYrRevenue, out decimal prYrRevenue,
				out decimal curYrGrossProfit, out decimal prYrGrossProfit,
				out decimal curYrNetIncome, out decimal pyYrNetIncome,
				out decimal wtAvgOutstandingShares, out decimal wtAvgOutstandingShares_py);

			//Cash-flow
			decimal cyCashFlowOCF = GetValuesFromCashFlowStatment();

			//Now let us compute the values
			//1. Returns on assets
			var roaFS = (curYrNetIncome / ((totalAssets + py_totalAsseets) / 2)) > 0 ? 1 : 0;
			//2. Positive operating cash flow
			var cfoFS = cyCashFlowOCF > 0 ? 1 : 0;
			//3. Higher return on assets than the previous year
			var roaDFS = (curYrNetIncome / ((totalAssets + py_totalAsseets) / 2)) >
				(pyYrNetIncome / ((py_totalAsseets + py2_totalAsseets) / 2)) ? 1 : 0;
			//4. Accruals; leverage, liquidity, and source of funds
			var cfoRoaFS = (cyCashFlowOCF / totalAssets) > (curYrNetIncome / ((totalAssets + py_totalAsseets) / 2)) ? 1 : 0;
			//5. Change in the leverage ratio;
			var ltdFS = longTermDebt < py_longTermDebt ? 1 : 0;
			//6. Change in current ratio
			var crFS = (currentAssets / currentLiabilities) > (py_currentAssets / py_currentLiabilities) ? 1 : 0;
			//7. If no new shares were issued.
			var sharesOutstandng = wtAvgOutstandingShares <= wtAvgOutstandingShares_py ? 1 : 0;
			//8. Change in Gross Margin;
			var gmFS = (curYrGrossProfit / curYrRevenue) > (prYrGrossProfit / prYrRevenue) ? 1 : 0;
			//9. Change in Asset turnout ratio
			var atoFS = (curYrRevenue / ((totalAssets + py_totalAsseets) / 2)) > (prYrRevenue / ((py_totalAsseets + py2_totalAsseets) / 2)) ? 1 : 0;

			var score = roaFS + cfoFS + roaDFS + cfoRoaFS + ltdFS + crFS + sharesOutstandng + gmFS + atoFS;
			return score;
		}



		private void GetValuesFromBalanceSheet(out decimal longTermDebt, out decimal totalAssets,
			out decimal currentAssets, out decimal currentLiabilities,
			out decimal py_longTermDebt, out decimal py_totalAsseets,
			out decimal py_currentAssets, out decimal py_currentLiabilities,
			out decimal py2_totalAsseets)
		{
			var bsCurrentFinancials = balanceSheet.Financials.FirstOrDefault();
			longTermDebt = bsCurrentFinancials.Longtermdebt;
			totalAssets = bsCurrentFinancials.Totalassets;
			currentAssets = bsCurrentFinancials.Totalcurrentassets;
			currentLiabilities = bsCurrentFinancials.Totalcurrentliabilities;

			var bsPriorFinancials = balanceSheet.Financials[OneYearAgo - 1];
			py_longTermDebt = bsPriorFinancials.Longtermdebt;
			py_totalAsseets = bsPriorFinancials.Totalassets;
			py_currentAssets = bsPriorFinancials.Totalcurrentassets;
			py_currentLiabilities = bsPriorFinancials.Totalcurrentliabilities;
			py2_totalAsseets = balanceSheet.Financials[LookbackTime - 1].Totalassets;
		}

		private decimal GetValuesFromCashFlowStatment()
		{
			var cfFinancials = cashFlowStatement.Financials.OrderByDescending(r => r.Date).ToList();
			var cyCashFlowOCF = cfFinancials[0].OperatingCashFlow
				+ cfFinancials[1].OperatingCashFlow
				+ cfFinancials[2].OperatingCashFlow
				+ cfFinancials[3].OperatingCashFlow;
			return cyCashFlowOCF;
		}

		private void GetValuesFromIncomeStatement(out decimal curYrRevenue,
			out decimal prYrRevenue, out decimal curYrGrossProfit,
			out decimal prYrGrossProfit, out decimal curYrNetIncome,
			out decimal pyYrNetIncome, out decimal wtAvgOutstandingShares,
			out decimal wtAvgOutstandingShares_py)
		{
			var isCurrentFinancials = incomeStatement.Financials.FirstOrDefault();
			var finLst = incomeStatement.Financials.OrderByDescending(r => r.Date).ToList();
			curYrRevenue = finLst[0].Revenue + finLst[1].Revenue + finLst[2].Revenue + finLst[3].Revenue;
			prYrRevenue = finLst[4].Revenue + finLst[5].Revenue + finLst[6].Revenue + finLst[7].Revenue;
			curYrGrossProfit = finLst[0].GrossProfit + finLst[1].GrossProfit + finLst[2].GrossProfit + finLst[3].GrossProfit;
			prYrGrossProfit = finLst[4].GrossProfit + finLst[5].GrossProfit + finLst[6].GrossProfit + finLst[7].GrossProfit;
			curYrNetIncome = finLst[0].NetIncome + finLst[1].NetIncome + finLst[2].NetIncome + finLst[3].NetIncome;
			pyYrNetIncome = finLst[4].NetIncome + finLst[5].NetIncome + finLst[6].NetIncome + finLst[7].NetIncome;
			wtAvgOutstandingShares = finLst[0].WeightedAverageShsOut;
			wtAvgOutstandingShares_py = finLst[4].WeightedAverageShsOut;
		}
``` 			

_**The above cannot be copied and used as the API has been modified and the names have changed. The document will be changed eventually.**_

Values will be obtained from the vendor, Simfin, using its API calls. It is to be noted that Simfin, at the time of writing of this document the vendor has released version 2 of its API calls but we will continue using V1 due to issues with the API. 

In the event, the Vendor does not have ratings for equity its rating will be marked as -1. Vendor ratings will be refreshed no earlier than 15 days. 
##### Clean up
Any record that was last computed/refreshed more than 100 days and if has been removed from the index list will be purged. 

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNzQ5OTE4OTBdfQ==
-->