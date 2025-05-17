* 100387171 - ECO-6004B Assessment 2 Do.File
* Project: Do alternative investments provide portfolio insurance during market turmoil?
* Dataset: Monthly prices for A portfolio equally weighted of Gold, Silver, Platinum, as well as the TR/CC and BCOM indexes, S&P500, VIX, US 1M T-bill.


* Import cleaned Excel data (formatted in Excel beforehand)
import excel using "/Users/stathitheo2004/Library/CloudStorage/OneDrive-UniversityofEastAnglia/Life/YEAR 3 UNI/Alternative Investments/Summatives/Summative 2/Datasets/Data.xlsx", sheet("Sheet1") firstrow clear

* Drop any accidentally imported blank rows at the bottom (check: describe/codebook showed 19 missing)
drop if missing(Date)

* Convert Excel date to proper Stata monthly date (very important for time-series ops)
gen month_date = mofd(Date)
format month_date %tm
drop Date
rename month_date Date

* Set time-series structure
tsset Date

* Define local assets
local asset_roots gold silver platinum sp500 trcc bcom
local price_vars gold_close silver_close platinum_close sp500_close trcc_close bcom_close

* Generate log price variables
local i = 1
foreach a of local asset_roots {
    local p : word `i' of `price_vars'
    gen log_`a' = ln(`p')
    local ++i
}

* Generate monthly log returns
foreach a of local asset_roots {
    gen lret_`a' = D.log_`a'
}

* Drop first observation where returns are missing (due to lack of lag)
drop if missing(lret_gold, lret_silver, lret_platinum, lret_sp500, lret_trcc, lret_bcom)

* Define metal log prices and returns
local metal_prices gold_close silver_close platinum_close
local metal_returns lret_gold lret_silver lret_platinum

* Portfolio price: average of metal closing prices
egen portfolio_close = rowmean(`metal_prices')

* Portfolio log return (alternative 1): naive average of component returns
egen lret_portfolio = rowmean(`metal_returns')

* Data is now cleaned, returns are generated, and Date is properly tsset. Can now generate excess returns, construct Crisis dummy, and prepare for regressions
* Generate excess returns (return minus risk-free rate) for the metals and the market index: US1M is expressed in percent form (e.g., 4.25%), so divide by 100 before subtracting:

* Define the assets for which to calculate excess returns
local excess_assets sp500 trcc bcom portfolio

* Generate excess returns over risk-free rate
foreach a of local excess_assets {
    gen exret_`a' = lret_`a' - (US1M / 100)
}

* Crisis Dummy: SP500 crash or high VIX
gen Crisis = (lret_sp500 <= -0.05) | (VIX > 30)
label variable Crisis "Crisis Dummy (VIX > 30 or SP500 <= -5%)"

* Create labels for excess return variables
local ex_assets portfolio trcc bcom sp500
local ex_labels Portfolio_Excess_Return TRCC_Excess_Return BCOM_Excess_Return SP500_Excess_Return

local i = 1
foreach a of local ex_assets {
    local lbl : word `i' of `ex_labels'
    label variable exret_`a' "`=subinstr("`lbl'", "_", " ", .)'"
    local ++i
}

* Normalize price levels to 100 at start (for time-series plots)
local norm_assets sp500 trcc bcom
local norm_prices sp500_close trcc_close bcom_close

local i = 1
foreach a of local norm_assets {
    local p : word `i' of `norm_prices'
    gen norm_`a' = (`p' / `p'[1]) * 100
    local ++i
}

* Cumulative returns and pseudo-price index for the 3-metal portfolio
gen cumlog_portfolio = sum(lret_portfolio)
gen price_portfolio = exp(cumlog_portfolio)
gen norm_portfolio = (price_portfolio / price_portfolio[1]) * 100

* Plot: Normalized Prices Over Time
twoway ///
    (line norm_portfolio Date, lcolor(gold)) ///
    (line norm_trcc Date, lcolor(blue)) ///
    (line norm_bcom Date, lcolor(red)) ///
    (line norm_sp500 Date, lcolor(black)), ///
    legend(label(1 "Portfolio") ///
           label(2 "TRCC") ///
           label(3 "BCOM") ///
           label(4 "S&P500")) ///
    title("Normalized Price Trends") ///
    ytitle("Index (Start = 100)") ///
    xtitle("Date") ///
    scheme(s1color)
	
* Normalize asset prices to 2005m1 base = 100

* Define assets, price vars, scalar names, and normalized var names
local assets sp500 trcc bcom portfolio
local price_vars sp500_close trcc_close bcom_close price_portfolio
local scalars sp500_2005 trcc_2005 bcom_2005 base_portfolio
local normvars norm2005_sp500 norm2005_trcc norm2005_bcom norm2005_portfolio

local i = 1
foreach a of local assets {
    local p : word `i' of `price_vars'
    local s : word `i' of `scalars'
    local n : word `i' of `normvars'

    summarize `p' if Date == tm(2005m1)
    scalar `s' = r(mean)
    gen `n' = (`p' / `s') * 100

    local ++i
}

* Plot: Normalized Prices (2005m1 base)

twoway ///
    (line norm2005_portfolio Date if Date >= tm(2005m1), lcolor(gold)) ///
    (line norm2005_trcc Date if Date >= tm(2005m1), lcolor(blue)) ///
    (line norm2005_bcom Date if Date >= tm(2005m1), lcolor(red)) ///
    (line norm2005_sp500 Date if Date >= tm(2005m1), lcolor(black)), ///
    legend(label(1 "Portfolio") label(2 "TRCC") label(3 "BCOM") label(4 "S&P500")) ///
    title("Normalized Price Trends (Start 2005m1 = 100)") ///
    ytitle("Index (Start = 100)") xtitle("Date") ///
    scheme(s1color)
	
* Regressions: Model 1 (Baseline betas)
local assets portfolio trcc bcom

foreach a of local assets {
    reg exret_`a' exret_sp500
    estimates store `a'1
}

* Regressions: Model 2 (Crisis interaction)
foreach a of local assets {
    reg exret_`a' exret_sp500 Crisis c.exret_sp500#c.Crisis
    estimates store `a'2
}

* Export regression tables to CSV

* Model 1 export
esttab portfolio1 trcc1 bcom1 using Model1_results.csv, ///
    title("Model 1: Portfolio Regressions on S&P500") ///
    se star(* 0.10 ** 0.05 *** 0.01) replace b(%9.3f) se(%9.3f) compress

* Model 2 export
esttab portfolio2 trcc2 bcom2 using Model2_results.csv, ///
    title("Model 2: Crisis Interaction Regressions") ///
    se star(* 0.10 ** 0.05 *** 0.01) replace b(%9.3f) se(%9.3f) compress

	

* Setup: Define assets for performance metrics
local assets portfolio trcc bcom sp500

* Descriptive Stats
summarize lret_portfolio lret_trcc lret_bcom lret_sp500

* Sharpe Ratios
foreach a of local assets {
    summarize exret_`a'
    scalar sharpe_`a' = r(mean) / r(sd)
    display "Sharpe Ratio (`a'):" sharpe_`a'
}


* Sortino Ratios (Excess Return / Downside Risk)
foreach a of local assets {
    gen neg_exret_`a' = cond(exret_`a' < 0, exret_`a', .)

    summarize exret_`a'
    scalar mean_ex_`a' = r(mean)

    summarize neg_exret_`a'
    scalar sortino_`a' = mean_ex_`a' / r(sd)

    display "Sortino Ratio (`a'):" sortino_`a'
}


* Jensen's Alpha (CAPM intercept)
foreach a of local assets {
    reg exret_`a' exret_sp500
    scalar jensen_`a' = _b[_cons]
    display "Jensen's Alpha (`a'):" jensen_`a'
}

* Market Beta (CAPM slope)
foreach a of local assets {
    reg exret_`a' exret_sp500
    scalar beta_`a' = _b[exret_sp500]
    display "Beta (`a'):" beta_`a'
}

* Maximum Drawdown (MDD)
foreach a of local assets {
    gen cum_log_`a' = sum(lret_`a')
    gen cum_`a' = exp(cum_log_`a')
    gen peak_`a' = cum_`a'
    replace peak_`a' = cum_`a'[1]
    
    forvalues i = 2/`=_N' {
        replace peak_`a' = max(peak_`a'[_n-1], cum_`a') in `i'
    }

    gen dd_`a' = (cum_`a' - peak_`a') / peak_`a'
    summarize dd_`a'
    scalar max_dd_`a' = r(min)
    display "Max Drawdown (`a') %:" max_dd_`a' * 100
}

* Drawdown Comparison Test (Portfolio vs. SP500)
ttest dd_portfolio == dd_sp500

* Visual representation of drawdown swings:
twoway (line dd_portfolio Date, lcolor(gold)) ///
       (line dd_trcc Date, lcolor(red)) ///
       (line dd_bcom Date, lcolor(blue)) ///
       (line dd_sp500 Date, lcolor(black)), ///
       title("Drawdown Comparison Across Portfolios") ///
       ytitle("Drawdown") xtitle("Date") ///
       legend(label(1 "3-Metal Portfolio") label(2 "TRCC") label(3 "BCOM") label(4 "S&P500")) ///
       scheme(s1color)

* Skewness & Kurtosis of Log Returns

local assets portfolio trcc bcom sp500

foreach a of local assets {
    summarize lret_`a', detail
    scalar skew_`a' = r(skewness)
    scalar kurt_`a' = r(kurtosis)
    display "Asset: `a'"
    display "Skewness = " skew_`a'
    display "Kurtosis = " kurt_`a'
}


* Set up and prepare for rolling beta and correlations:

ssc install rangestat, replace
sort Date

* Define assets and rolling window lengths (months)
local assets portfolio trcc bcom
local windows 12 60 120

* Rolling regressions and variable generation
foreach a of local assets {
    foreach w of local windows {
        
        * Define rolling window size
        local lag = `w' - 1

        * Run rolling regression of lret_`a' on lret_sp500
        rangestat (reg) lret_`a' lret_sp500, interval(Date -`lag' 0)

        * Generate and rename outputs
        gen beta_`a'_`w'm = b_lret_sp500
        gen alpha_`a'_`w'm = b_cons
        gen se_beta_`a'_`w'm = se_lret_sp500
        gen se_alpha_`a'_`w'm = se_cons

        rename reg_nobs reg_nobs_`a'_`w'm
        rename reg_r2 reg_r2_`a'_`w'm
        rename reg_adj_r2 reg_adj_r2_`a'_`w'm

        * Clean up temp vars
        drop b_* se_*
    }
}

* 12-month rolling beta (all assets)
twoway ///
(line beta_portfolio_12m Date if reg_nobs_portfolio_12m==12, lcolor(gold)) ///
(line beta_trcc_12m Date if reg_nobs_trcc_12m==12, lcolor(grey)) ///
(line beta_bcom_12m Date if reg_nobs_bcom_12m==12, lcolor(brown)), ///
title("12-Month Rolling Betas vs S&P500") ///
ytitle("Beta") xtitle("Date") ///
legend(order(1 "Portfolio" 2 "TRCC" 3 "BCOM")) ///
scheme(s1color)

* 60-month rolling beta
twoway ///
(line beta_portfolio_60m Date if reg_nobs_portfolio_60m==60, lcolor(gold)) ///
(line beta_trcc_60m Date if reg_nobs_trcc_60m==60, lcolor(grey)) ///
(line beta_bcom_60m Date if reg_nobs_bcom_60m==60, lcolor(brown)), ///
title("60-Month Rolling Betas vs S&P500") ///
ytitle("Beta") xtitle("Date") ///
legend(order(1 "Portfolio" 2 "TRCC" 3 "BCOM")) ///
scheme(s1color)

* 120-month rolling beta
twoway ///
(line beta_portfolio_120m Date if reg_nobs_portfolio_120m==120, lcolor(gold)) ///
(line beta_trcc_120m Date if reg_nobs_trcc_120m==120, lcolor(grey)) ///
(line beta_bcom_120m Date if reg_nobs_bcom_120m==120, lcolor(brown)), ///
title("120-Month Rolling Betas vs S&P500") ///
ytitle("Beta") xtitle("Date") ///
legend(order(1 "Portfolio" 2 "TRCC" 3 "BCOM")) ///
scheme(s1color)


* Multi-horizon: Portfolio
twoway ///
(line beta_portfolio_12m Date if reg_nobs_portfolio_12m==12, lcolor(gold)) ///
(line beta_portfolio_60m Date if reg_nobs_portfolio_60m==60, lcolor(blue)) ///
(line beta_portfolio_120m Date if reg_nobs_portfolio_120m==120, lcolor(red)), ///
title("Rolling Betas: Portfolio vs S&P500 (1Y, 5Y, 10Y)") ///
ytitle("Beta") xtitle("Date") ///
legend(order(1 "1-Year" 2 "5-Year" 3 "10-Year")) ///
scheme(s1color)

* TRCC
twoway ///
(line beta_trcc_12m Date if reg_nobs_trcc_12m==12, lcolor(gold)) ///
(line beta_trcc_60m Date if reg_nobs_trcc_60m==60, lcolor(blue)) ///
(line beta_trcc_120m Date if reg_nobs_trcc_120m==120, lcolor(red)), ///
title("Rolling Betas: TRCC vs S&P500 (1Y, 5Y, 10Y)") ///
ytitle("Beta") xtitle("Date") ///
legend(order(1 "1-Year" 2 "5-Year" 3 "10-Year")) ///
scheme(s1color)

* BCOM
twoway ///
(line beta_bcom_12m Date if reg_nobs_bcom_12m==12, lcolor(gold)) ///
(line beta_bcom_60m Date if reg_nobs_bcom_60m==60, lcolor(blue)) ///
(line beta_bcom_120m Date if reg_nobs_bcom_120m==120, lcolor(red)), ///
title("Rolling Betas: BCOM vs S&P500 (1Y, 5Y, 10Y)") ///
ytitle("Beta") xtitle("Date") ///
legend(order(1 "1-Year" 2 "5-Year" 3 "10-Year")) ///
scheme(s1color)


* Rolling Correlations using rangestat

* Define Crisis Bands
gen ylow12m = -2.2
gen yhigh12m = 3

gen ylow60m = -2.2
gen yhigh60m = 1
		  
gen ylow120m = -1
gen yhigh120m = 1

gen ylowport = -1
gen yhighport = 3

gen ylowtrcc = -0.7
gen yhightrcc = 1.5

gen ylowbcom = -0.7
gen yhighbcom = 1.5

* Rolling Correlations: Portfolio vs S&P500 
rangestat (corr) lret_portfolio lret_sp500, interval(Date -11 0)
rename corr_nobs portfolio_sp500_nobs_12m
rename corr_x corr_portfolio_sp500_12m

rangestat (corr) lret_portfolio lret_sp500, interval(Date -59 0)
rename corr_nobs portfolio_sp500_nobs_60m
rename corr_x corr_portfolio_sp500_60m

rangestat (corr) lret_portfolio lret_sp500, interval(Date -119 0)
rename corr_nobs portfolio_sp500_nobs_120m
rename corr_x corr_portfolio_sp500_120m


* Rolling Correlations: TRCC vs S&P500 
rangestat (corr) lret_trcc lret_sp500, interval(Date -11 0)
rename corr_nobs trcc_sp500_nobs_12m
rename corr_x corr_trcc_sp500_12m

rangestat (corr) lret_trcc lret_sp500, interval(Date -59 0)
rename corr_nobs trcc_sp500_nobs_60m
rename corr_x corr_trcc_sp500_60m

rangestat (corr) lret_trcc lret_sp500, interval(Date -119 0)
rename corr_nobs trcc_sp500_nobs_120m
rename corr_x corr_trcc_sp500_120m


* Rolling Correlations: BCOM vs S&P500 
rangestat (corr) lret_bcom lret_sp500, interval(Date -11 0)
rename corr_nobs bcom_sp500_nobs_12m
rename corr_x corr_bcom_sp500_12m

rangestat (corr) lret_bcom lret_sp500, interval(Date -59 0)
rename corr_nobs bcom_sp500_nobs_60m
rename corr_x corr_bcom_sp500_60m

rangestat (corr) lret_bcom lret_sp500, interval(Date -119 0)
rename corr_nobs bcom_sp500_nobs_120m
rename corr_x corr_bcom_sp500_120m


* Portfolio 
twoway ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(1995m1), tm(2001m1)), color(gs13)) ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(2007m6), tm(2009m2)), color(gs13)) ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(2020m1), tm(2020m9)), color(gs13)) ///
(line beta_portfolio_120m Date if reg_nobs_portfolio_120m==120, yaxis(1) lcolor(gold)) ///
(line corr_portfolio_sp500_120m Date if portfolio_sp500_nobs_120m==120, yaxis(2) lcolor(blue) lpattern(dash)), ///
title("Portfolio: Rolling Beta (LHS) and Correlation (RHS) — 120M", size(medsmall)) ///
ytitle("Beta", axis(1)) ytitle("Correlation", axis(2)) xtitle("Date", margin(small)) ///
legend(order(4 "Beta (LHS)" 5 "Correlation (RHS)") rows(1)) ///
xlabel(`=tm(1995m2)' `=tm(2005m2)' `=tm(2015m2)' `=tm(2025m2)', format(%tmCY) labsize(small) angle(0) grid glcolor(gs14)) ///
name(plot_portfolio_120m, replace)  ///
scheme(s1color)

* TRCC 
twoway ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(1995m1), tm(2001m1)), color(gs13)) ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(2007m6), tm(2009m2)), color(gs13)) ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(2020m1), tm(2020m9)), color(gs13)) ///
(line beta_trcc_120m Date if reg_nobs_trcc_120m==120, yaxis(1) lcolor(red)) ///
(line corr_trcc_sp500_120m Date if trcc_sp500_nobs_120m==120, yaxis(2) lcolor(blue) lpattern(dash)), ///
title("TRCC: Rolling Beta (LHS) and Correlation (RHS) — 120M", size(medsmall)) ///
ytitle("Beta", axis(1)) ytitle("Correlation", axis(2)) xtitle("Date", margin(small)) ///
legend(order(4 "Beta" 5 "Correlation") rows(1)) ///
xlabel(`=tm(1995m2)' `=tm(2005m2)' `=tm(2015m2)' `=tm(2025m2)', format(%tmCY) labsize(small) angle(0) grid glcolor(gs14)) ///
name(plot_trcc_120m, replace)  ///
scheme(s1color)

* BCOM 
twoway ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(1995m1), tm(2001m1)), color(gs13)) ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(2007m6), tm(2009m2)), color(gs13)) ///
(rarea ylow120m yhigh120m Date if inrange(Date, tm(2020m1), tm(2020m9)), color(gs13)) ///
(line beta_bcom_120m Date if reg_nobs_bcom_120m==120, yaxis(1) lcolor(brown)) ///
(line corr_bcom_sp500_120m Date if bcom_sp500_nobs_120m==120, yaxis(2) lcolor(blue) lpattern(dash)), ///
title("BCOM: Rolling Beta (LHS) and Correlation (RHS) — 120M", size(medsmall)) ///
ytitle("Beta", axis(1)) ytitle("Correlation", axis(2)) xtitle("Date", margin(small)) ///
legend(order(4 "Beta" 5 "Correlation") rows(1)) ///
xlabel(`=tm(1995m2)' `=tm(2005m2)' `=tm(2015m2)' `=tm(2025m2)', format(%tmCY) labsize(small) angle(0) grid glcolor(gs14)) ///
name(plot_bcom_120m, replace)  ///
scheme(s1color)

graph combine plot_portfolio_120m plot_trcc_120m plot_bcom_120m,  ///
rows(1) ///
title("120M Rolling Betas & Correlations with Crisis Periods Highlighted", size(medium)) ///
iscale(0.7) xsize(8) ysize(4) ///
scheme(s1color)

* Combine: Rolling Beta Panel
graph combine beta1 beta2 beta3, ///
title("Rolling Betas Across Horizons") col(3) ///
iscale(0.8) xsize(9) ysize(3) scheme(s1color)

* Combine: Rolling Correlation Panel
graph combine correlation1 correlation2 correlation3, ///
title("Rolling Correlations Across Horizons") col(3) ///
iscale(0.8) xsize(9) ysize(3) scheme(s1color)


* Standard Correlation Matrix: Log Returns
pwcorr lret_portfolio lret_trcc lret_bcom lret_sp500
matrix Correlation = r(C)
heatplot Correlation, title("Correlation Matrix: Main Portfolios")

* Appendix-Wide Correlation Matrix
pwcorr lret_*
matrix Correlation = r(C)
heatplot Correlation, title("Correlation Matrix: All Log Returns")

* Crisis-Period Correlations

* Define crisis labels and conditions — each condition must be quoted
* Define crisis labels and corresponding filters
local crises GFC COVID TechBubble HighVIXorCrash

* Store each filter as a separate macro
local f1 inrange(Date, tm(2007m6), tm(2009m2))
local f2 inrange(Date, tm(2020m1), tm(2020m9))
local f3 inrange(Date, tm(1995m1), tm(2001m1))
local f4 VIX > 30 | lret_sp500 < -0.05

* Loop over crises
local i = 1
foreach label of local crises {

    * Correctly resolve the filter condition
    local cond = "`f`i''"

    display "=== Correlation During `label' ==="
    pwcorr lret_portfolio lret_trcc lret_bcom lret_sp500 if `cond'
    matrix Correlation = r(C)
    heatplot Correlation, title("Correlation: `label' Period")

    local ++i
}

* Define Crisis Dummies
gen gfc_crisis   = inrange(Date, tm(2007m6), tm(2009m2))
gen covid_crisis = inrange(Date, tm(2020m1), tm(2020m9))
gen tech_crisis  = inrange(Date, tm(1995m1), tm(2001m1))

* Run Regressions for GFC and COVID (excess returns)
local crises      gfc     covid
local crisis_var  gfc_crisis covid_crisis
local models      3       4

foreach a in portfolio trcc bcom {
    local i = 1
    foreach c of local crises {
        local cv : word `i' of `crisis_var'
        local m  : word `i' of `models'

        reg exret_`a' exret_sp500 `cv' c.exret_sp500#c.`cv'
        estimates store `a'`m'

        local ++i
    }
}

* Run Regressions for Tech Bubble (raw log returns due to US1M T-bill not introduced yet)
foreach a in portfolio trcc bcom {
    reg lret_`a' lret_sp500 tech_crisis c.lret_sp500#c.tech_crisis
    estimates store `a'5
}

* Export Model Results to CSV

* Model 3: GFC
esttab portfolio3 trcc3 bcom3 using Model3_results.csv, ///
    title("Model 3: GFC Model Returns") ///
    se star(* 0.10 ** 0.05 *** 0.01) replace b(%9.3f) se(%9.3f) compress

* Model 4: COVID
esttab portfolio4 trcc4 bcom4 using Model4_results.csv, ///
    title("Model 4: COVID-19 Model Returns") ///
    se star(* 0.10 ** 0.05 *** 0.01) replace b(%9.3f) se(%9.3f) compress

* Model 5: Tech Bubble
esttab portfolio5 trcc5 bcom5 using Model5_results.csv, ///
    title("Model 5: Tech-Bubble Model Returns") ///
    se star(* 0.10 ** 0.05 *** 0.01) replace b(%9.3f) se(%9.3f) compress


* Overlay plot of the VIX and the S&P500:	
twoway ///
(line VIX Date if Date >= tm(1990m1), yaxis(1) lcolor(blue) lwidth(medium)) ///
(line sp500_close Date if Date >= tm(1990m1), yaxis(2) lcolor(orange_red) lwidth(medium)), ///
title("The Link Between the S&P 500 and the VIX") ///
ylabel(10(10)70, axis(1) angle(horizontal) labcolor(blue)) ///
ylabel(0(1200)6000, axis(2) angle(horizontal) labcolor(orange_red)) ///
ytitle("VIX Index (LHS)", axis(1)) ///
ytitle("S&P 500 Index (RHS)", axis(2)) ///
xtitle("Date") ///
legend(order(1 "VIX Index (LHS)" 2 "S&P 500 Index (RHS)")) ///
scheme(s1color)

* Overlay plot of the VIX and the 3-Metal Portfolio:	
twoway ///
(line VIX Date if Date >= tm(1990m1), yaxis(1) lcolor(blue) lwidth(medium)) ///
(line portfolio_close Date if Date >= tm(1990m1), yaxis(2) lcolor(orange_red) lwidth(medium)), ///
title("The Link Between the S&P 500 and the VIX") ///
ylabel(10(10)70, axis(1) angle(horizontal) labcolor(blue)) ///
ylabel(0(300)1800, axis(2) angle(horizontal) labcolor(orange_red)) ///
ytitle("VIX Index (LHS)", axis(1)) ///
ytitle("Portfolio (RHS)", axis(2)) ///
xtitle("Date") ///
legend(order(1 "VIX Index (LHS)" 2 "Portfolio (RHS)")) ///
scheme(s1color)

* Overlay plot of the VIX and the TRCC:	
twoway ///
(line VIX Date if Date >= tm(1990m1), yaxis(1) lcolor(blue) lwidth(medium)) ///
(line trcc_close Date if Date >= tm(1990m1), yaxis(2) lcolor(orange_red) lwidth(medium)), ///
title("The Link Between the TRCC Index and the VIX") ///
ylabel(10(10)70, axis(1) angle(horizontal) labcolor(blue)) ///
ylabel(100(100)500, axis(2) angle(horizontal) labcolor(orange_red)) ///
ytitle("VIX Index (LHS)", axis(1)) ///
ytitle("TRCC Index (RHS)", axis(2)) ///
xtitle("Date") ///
legend(order(1 "VIX Index (LHS)" 2 "TRCC Index (RHS)")) ///
scheme(s1color)

* Overlay plot of the VIX and the BCOM:	
twoway ///
(line VIX Date if Date >= tm(1990m1), yaxis(1) lcolor(blue) lwidth(medium)) ///
(line bcom_close Date if Date >= tm(1990m1), yaxis(2) lcolor(orange_red) lwidth(medium)), ///
title("The Link Between the BCOM Index and the VIX") ///
ylabel(10(10)70, axis(1) angle(horizontal) labcolor(blue)) ///
ylabel(50(50)250, axis(2) angle(horizontal) labcolor(orange_red)) ///
ytitle("VIX Index (LHS)", axis(1)) ///
ytitle("BCOM Index (RHS)", axis(2)) ///
xtitle("Date") ///
legend(order(1 "VIX Index (LHS)" 2 "BCOM Index (RHS)")) ///
scheme(s1color)


* Set seed for reproducibility
set seed 12345

* Quantile regressions at the 1%, 2.5%, and 5% quantiles for Portfolio, TRCC, and BCOM
foreach asset in portfolio trcc bcom {
    foreach q in 0.01 0.025 0.05 {
        local qname = subinstr("`q'", ".", "_", .)
        di "Running Quantile Regression for `asset' at τ = `q'"
        bsqreg exret_`asset' exret_sp500, quantile(`q') reps(100)
        estimates store q`asset'_`qname'
    }
}

* Metals Portfolio
esttab qportfolio_0_01 qportfolio_0_025 qportfolio_0_05 using "Quantile_Portfolio.csv", ///
    se star(* 0.10 ** 0.05 *** 0.01) label b(4) se(4) ///
    title("Quantile Regression: 3-Metal Portfolio vs S&P500") ///
    replace

* TRCC Index
esttab qtrcc_0_01 qtrcc_0_025 qtrcc_0_05 using "Quantile_TRCC.csv", ///
    se star(* 0.10 ** 0.05 *** 0.01) label b(4) se(4) ///
    title("Quantile Regression: TRCC Index vs S&P500") ///
    replace

* BCOM Index
esttab qbcom_0_01 qbcom_0_025 qbcom_0_05 using "Quantile_BCOM.csv", ///
    se star(* 0.10 ** 0.05 *** 0.01) label b(4) se(4) ///
    title("Quantile Regression: BCOM Index vs S&P500") ///
    replace


* Calculating the VaR (Value at Risk) for Portfolio, TRCC, BCOM, and S&P500. Define portfolio value for each asset (portfolio and 3 indexes) - $100,000 invested in each:
local investment = 100000

* Convert log returns to simple returns
foreach var in lret_portfolio lret_trcc lret_bcom lret_sp500 {
    gen simple_`var' = exp(`var') - 1 
}

* Parametric VaR (Normal Distribution)
foreach var in lret_portfolio lret_trcc lret_bcom lret_sp500 {
    quietly summarize simple_`var'
    scalar mean_return = r(mean)
    scalar sd_return = r(sd)
    scalar z = invnormal(0.95)
    scalar VaR_parametric = `investment' * -(mean_return - z * sd_return)
    
* Dollar formatting with commas (advanced)
    local formatted_val = string(VaR_parametric, "%12.2fc")
    display "Parametric 95% VaR for `var': $" "`formatted_val'"
}

* Historical VaR (5th Percentile)
foreach var in lret_portfolio lret_trcc lret_bcom lret_sp500 {
  quietly _pctile simple_`var', p(5) // 5th percentile = worst 5% of returns
  scalar VaR_historical = 100000 * -r(r1)
  display "Historical 95% VaR for `var': $" %9.2fc VaR_historical
}

* Test for normality in distribution, null hypothesis rejection here means return data is non-normal (fat tails, large shocks).
swilk simple_*
* Strong evidence for non-normality, using Conditional Value at Risk (CVaR) - also known as Expected Shortfall (ES) as alternative:

* Calculate CVaR (95% confidence)
foreach var in simple_lret_portfolio simple_lret_trcc simple_lret_bcom simple_lret_sp500 {
  quietly _pctile `var', p(5) // Get 5th percentile
  scalar threshold = r(r1)
  quietly summarize `var' if `var' <= threshold // Average losses beyond threshold
  scalar CVaR = -r(mean) * 100000 // Dollar-denominated (positive value)
  display "CVaR (95%) for `var': $" %9.2fc CVaR
}

* Set rolling window (e.g., 12 months)
local window = 12

*  Rolling CVaR (12-Month Window)
foreach var of varlist lret_portfolio lret_trcc lret_bcom lret_sp500 {
  gen rCVaR_`var' = . 
  quietly forvalues i = `window'/`=_N' {
    local start = `i' - `window' + 1
    _pctile `var' in `start'/`i', p(5)
    scalar threshold = r(r1)
    summarize `var' in `start'/`i' if `var' <= threshold
    replace rCVaR_`var' = -r(mean) * 100000 in `i'  // Dollar value
  }
  line rCVaR_`var' Date, title("Rolling CVaR: `var'") ytitle("CVaR ($)") name(graph_`var', replace)
}

* Combine CVaR into a table
tabstat rCVaR_lret_*, stat(mean sd min max) format(%9.2fc)

* Stress Testing by identifying dates where CVaR values drove to maximum:
list Date if rCVaR_lret_portfolio > 18000 
list Date if rCVaR_lret_trcc > 18000  
list Date if rCVaR_lret_bcom > 18000  
list Date if rCVaR_lret_sp500 > 18000  
	
* Constructing a mean-variance portfolio (consisting of 3-Metal Portfolio, TRCC, BCOM, and the S&P500):

* Calculate expected returns (means)
matrix mean_returns = J(1, 4, .)
local i 1
foreach var in exret_portfolio exret_trcc exret_bcom exret_sp500 {
    quietly summarize `var'
    matrix mean_returns[1, `i'] = r(mean)
    local ++i
}
matrix colnames mean_returns = "Portfolio" "TRCC" "BCOM" "SP500"

* Compute covariance matrix using Mata 
mata:
    returns = st_data(., "exret_portfolio exret_trcc exret_bcom exret_sp500")
    cov_matrix = variance(returns)
    st_matrix("cov_matrix", cov_matrix)
end
matrix colnames cov_matrix = "Portfolio" "TRCC" "BCOM" "SP500"
matrix rownames cov_matrix = "Portfolio" "TRCC" "BCOM" "SP500"
matrix list cov_matrix

* Compute long position only weights:
mata:
    // Load matrices
    mean = st_matrix("mean_returns")'
    cov = st_matrix("cov_matrix")
    
    // Closed-form solution with non-negativity constraint
    optimal_weights = lusolve(cov, mean) / sum(lusolve(cov, mean))
    optimal_weights = optimal_weights :* (optimal_weights :> 0)  // Force weights ≥ 0
    optimal_weights = optimal_weights / sum(optimal_weights)     // Re-normalize to 100%
    
    // Save and print
    st_matrix("optimal_weights_lo", optimal_weights')
end

* Step 2: Label and display results
matrix colnames optimal_weights_lo = Portfolio TRCC BCOM SP500
matrix rownames optimal_weights_lo = "LongOnly_Weights"
matrix list optimal_weights_lo, format(%9.4f)

* Re-Compute weights but allowing for short-selling:

* Verify existing matrices
matrix list mean_returns
matrix list cov_matrix

mata:
    // Load and validate matrices
    if (st_matrix("mean_returns") == J(1,4,.)) {
        _error("mean_returns not calculated properly")
    }
    if (st_matrix("cov_matrix") == J(4,4,.)) {
        _error("cov_matrix not calculated properly")
    }
    
    mean = st_matrix("mean_returns")'
    cov = st_matrix("cov_matrix")
    
    // Core calculation (max Sharpe ratio)
    optimal_weights = lusolve(cov, mean)/sum(lusolve(cov, mean))
    
    // Force row vector output
    optimal_weights = optimal_weights'
    st_matrix("optimal_weights", optimal_weights)
end

* Display results of optimal weights:
matrix optimal_weights = optimal_weights[1,1..4]
matrix colnames optimal_weights = Portfolio TRCC BCOM SP500
matrix rownames optimal_weights = "Optimal_Weights"
matrix list optimal_weights, format(%9.4f)

* Constructing mean-variance optimised portfolio variables (short allowed and long only):

gen portf_return_short = (0.0883 * lret_portfolio) + (-0.0169 * lret_trcc) + ///
                         (0.6920 * lret_bcom) + (0.2366 * lret_sp500)

gen portf_return_long = (0.0868 * lret_portfolio) + (0 * lret_trcc) + ///
                        (0.6805 * lret_bcom) + (0.2327 * lret_sp500)

* Cumulative return (log compounding)
gen cum_mvo_short = exp(sum(portf_return_short))
gen cum_mvo_long  = exp(sum(portf_return_long))

twoway (line cum_mvo_short Date, lcolor(blue)) ///
       (line cum_mvo_long Date, lcolor(green) lpattern(dash)), ///
       title("Cumulative Value: MVO Portfolios (Short vs Long-Only)") ///
       ytitle("Index Level") xtitle("Date") legend(order(1 "Shorting Allowed" 2 "Long-Only"))

* Create cumulative log price series:
gen ln_price_mvo_lo = sum(portf_return_long)

* Convert to an index normalised to 100 at 2005m1
gen norm2005_mvo_lo = exp(ln_price_mvo_lo)
summarize norm2005_mvo_lo if Date == tm(2005m1)
scalar base_mvo_lo = r(mean)
replace norm2005_mvo_lo = (norm2005_mvo_lo / base_mvo_lo) * 100

* Plot MVO Vs Portfolio Vs S&P500:
twoway ///
(line norm2005_mvo_lo Date if Date >= tm(2005m1), lcolor(green) lwidth(medium)) ///
(line norm2005_portfolio Date if Date >= tm(2005m1), lcolor(blue) lwidth(medium)) ///
(line norm2005_sp500 Date if Date >= tm(2005m1), lcolor(black) lpattern(dash)), ///
title("Normalized Index: MVO Long-Only vs Portfolio vs S&P500 (Start = 100)") ///
ytitle("Index Level") xtitle("Date") ///
legend(label(1 "MVO Long-Only") label(2 "Portfolio") label(3 "S&P500")) ///
scheme(s1color)





*--------------------------------------------------------------------------------------------------------------------------------------------------------





* Setup for metals
local metals gold silver platinum

* Excess Return Variables
foreach m of local metals {
    gen exret_`m' = lret_`m' - (US1M / 100)
}

* Descriptive Statistics: Log Returns 
summarize lret_gold lret_silver lret_platinum

* Sharpe Ratios 
foreach m of local metals {
    summarize exret_`m'
    scalar sharpe_`m' = r(mean) / r(sd)
    display "Sharpe Ratio for `m': " sharpe_`m'
}

* Sortino Ratios 
foreach m of local metals {
    gen neg_exret_`m' = cond(exret_`m' < 0, exret_`m', .)

    summarize exret_`m'
    scalar mean_ex_`m' = r(mean)

    summarize neg_exret_`m'
    scalar sortino_`m'_ex = mean_ex_`m' / r(sd)

    display "Sortino Ratio for `m': " sortino_`m'_ex
}

* Jensen's Alpha (Intercept from CAPM) 
foreach m of local metals {
    reg exret_`m' exret_sp500
    scalar jensen_`m' = _b[_cons]
    display "Jensen's Alpha for `m': " jensen_`m'
}

* Market Beta (Slope from CAPM) 
foreach m of local metals {
    reg exret_`m' exret_sp500
    scalar beta_`m' = _b[exret_sp500]
    display "Beta for `m': " beta_`m'
}

* Maximum Drawdown Calculations 
foreach m of local metals {
    gen cum_log_`m' = sum(lret_`m')
    gen cum_`m' = exp(cum_log_`m')
    gen peak_`m' = cum_`m'
    replace peak_`m' = cum_`m'[1]
    forvalues i = 2/`=_N' {
        replace peak_`m' = max(peak_`m'[_n-1], cum_`m') in `i'
    }
    gen dd_`m' = (cum_`m' - peak_`m') / peak_`m'
    summarize dd_`m'
    scalar max_dd_`m' = r(min)
    display "Max Drawdown for `m': " max_dd_`m' * 100
}

* Drawdown Test: Gold vs S&P500 
ttest dd_gold == dd_sp500

* Drawdown Plots 
twoway (line dd_gold Date, lcolor(gold)) ///
       (line dd_sp500 Date, lcolor(black)), ///
       title("Drawdown Comparison: Gold vs S&P500") ///
       ytitle("Drawdown") xtitle("Date") ///
       legend(label(1 "Gold") label(2 "S&P500")) ///
       scheme(s1color)
	   
twoway (line dd_silver Date, lcolor(gold)) ///
       (line dd_sp500 Date, lcolor(black)), ///
       title("Drawdown Comparison: Silver vs S&P500") ///
       ytitle("Drawdown") xtitle("Date") ///
       legend(label(1 "Silver") label(2 "S&P500")) ///
       scheme(s1color)
	   
twoway (line dd_platinum Date, lcolor(gold)) ///
       (line dd_sp500 Date, lcolor(black)), ///
       title("Drawdown Comparison: Platinum vs S&P500") ///
       ytitle("Drawdown") xtitle("Date") ///
       legend(label(1 "Platinum") label(2 "S&P500")) ///
       scheme(s1color)
	   
twoway (line dd_gold Date, lcolor(gold)) ///
       (line dd_silver Date, lcolor(grey)) ///
       (line dd_platinum Date, lcolor(brown)), ///
       title("Drawdown Comparison: Gold vs Silver vs Platinum") ///
       ytitle("Drawdown") xtitle("Date") ///
       legend(label(1 "Gold") label(2 "Silver") label(3 "Platinum")) ///
       scheme(s1color)

* Skewness and Kurtosis 
foreach m of local metals {
    summarize lret_`m', detail
}

* Calculating the VaR (Value at Risk) for Gold, Silver, and Platinum:

* Define portfolio value for each asset - $100,000 invested in each
local investment = 100000

* Convert log returns to simple returns
foreach var in lret_gold lret_silver lret_platinum {
    gen simple_`var' = exp(`var') - 1 
}

* Parametric VaR (Normal Distribution)
foreach var in lret_gold lret_silver lret_platinum {
    quietly summarize simple_`var'
    scalar mean_return = r(mean)
    scalar sd_return = r(sd)
    scalar z = invnormal(0.95)
    scalar VaR_parametric = `investment' * -(mean_return - z * sd_return)
    
* Dollar formatting with commas
    local formatted_val = string(VaR_parametric, "%12.2fc")
    display "Parametric 95% VaR for `var': $" "`formatted_val'"
}

* Historical VaR (5th Percentile)
foreach var in lret_gold lret_silver lret_platinum {
  quietly _pctile simple_`var', p(5) // 5th percentile = worst 5% of returns
  scalar VaR_historical = 100000 * -r(r1)
  display "Historical 95% VaR for `var': $" %9.2fc VaR_historical
}

* Test for normality in distribution, null hypothesis rejection here means return data is non-normal (fat tails, large shocks).
swilk simple_*
* Strong evidence for non-normality, using Conditional Value at Risk (CVaR) - also known as Expected Shortfall (ES) as alternative:

* Calculate CVaR (95% confidence)
foreach var in simple_lret_gold simple_lret_silver simple_lret_platinum {
  quietly _pctile `var', p(5) // Get 5th percentile
  scalar threshold = r(r1)
  quietly summarize `var' if `var' <= threshold // Average losses beyond threshold
  scalar CVaR = -r(mean) * 100000 // Dollar-denominated (positive value)
  display "CVaR (95%) for `var': $" %9.2fc CVaR
}

* Set rolling window (e.g., 12 months)
local window = 12

*  Rolling CVaR (12-Month Window)
foreach var of varlist lret_gold lret_silver lret_platinum {
  gen rCVaR_`var' = . 
  quietly forvalues i = `window'/`=_N' {
    local start = `i' - `window' + 1
    _pctile `var' in `start'/`i', p(5)
    scalar threshold = r(r1)
    summarize `var' in `start'/`i' if `var' <= threshold
    replace rCVaR_`var' = -r(mean) * 100000 in `i'  // Dollar value
  }
  line rCVaR_`var' Date, title("Rolling CVaR: `var'") ytitle("CVaR ($)") name(graph_`var', replace)
}

* Combine CVaR into a table
tabstat rCVaR_lret_*, stat(mean sd min max) format(%9.2fc)

* Final calculations: Conditional Sharpe Ratio across every asset:

* List of assets
foreach asset in portfolio trcc bcom sp500 gold silver platinum {
    
    * Convert CVaR variable into scalar (mean of series)
    quietly summarize rCVaR_lret_`asset'
    scalar CVaR_`asset' = r(mean)
    
    * Mean excess return
    summarize exret_`asset'
    scalar mean_exret_`asset' = r(mean)
    
    * Calculate Conditional Sharpe Ratio
    scalar CSR_`asset' = mean_exret_`asset' / (CVaR_`asset' / 100000)
    
    display "Conditional Sharpe Ratio (`asset'): " CSR_`asset'
}

* and Excess Returns on VaR, also across every asset:

* Parametric VaR (Normal) — generate and store individual scalars
foreach asset in gold silver platinum portfolio trcc bcom sp500 {
    quietly summarize simple_lret_`asset'
    scalar mean_`asset' = r(mean)
    scalar sd_`asset' = r(sd)
    scalar VaR_`asset' = 100000 * -(mean_`asset' - invnormal(0.95) * sd_`asset')
}

* Now calculate Excess Return on VaR
foreach asset in portfolio trcc bcom sp500 gold silver platinum {
    scalar ERVaR_`asset' = mean_exret_`asset' / (VaR_`asset' / 100000)
    display "Excess Return on VaR (`asset'): " ERVaR_`asset'
}

	
* Creating line plots with grey zones indicating crises:

* Shading bounds
gen ylow = 5
gen yhigh = 1495

* Crisis shading line plots of entire time period:
twoway ///
    (rarea ylow yhigh Date if inrange(Date, tm(2007m6), tm(2009m2)), color(gs13) lcolor(gs14)) ///
    (rarea ylow yhigh Date if inrange(Date, tm(2020m1), tm(2020m9)), color(gs13) lcolor(gs14)) ///
    (rarea ylow yhigh Date if inrange(Date, tm(1995m1), tm(2001m1)), color(gs13) lcolor(gs14)) ///
    (line norm_portfolio Date, lcolor(gold) lwidth(medium)) ///
    (line norm_trcc Date, lcolor(red) lwidth(medium)) ///
    (line norm_bcom Date, lcolor(blue) lwidth(medium)) ///
    (line norm_sp500 Date, lcolor(black) lwidth(medium)), ///
    title("Normalized Index Performance (Start = 100)", size(medsmall)) ///
    subtitle("Shaded Areas Represent GFC, COVID-19 and Tech-Bubble Crises", size(small)) ///
    ytitle("Index Level (Base = 100)", size(small)) ///
    xtitle("Date", size(small)) ///
    ylabel(, angle(horizontal) labsize(small)) ///
    xlabel(, labsize(small)) ///
    legend(order(4 "3-Metal Portfolio" 5 "TRCC" 6 "BCOM" 7 "S&P500") ///
           col(2) size(small) region(lstyle(none))) ///
    graphregion(color(white)) ///
    plotregion(margin(zero)) ///
    scheme(s1color)

* Create updated shading bounds
gen ylow1 = 2
gen yhigh1 = 598

* Crisis shading line plots of sample time period (2005m1 - present):	
twoway ///
    (rarea ylow1 yhigh1 Date if inrange(Date, tm(2007m6), tm(2009m2)), color(gs13) lcolor(gs14)) ///
    (rarea ylow1 yhigh1 Date if inrange(Date, tm(2020m1), tm(2020m9)), color(gs13) lcolor(gs14)) ///
    (line norm2005_portfolio Date if Date >= tm(2005m1), lcolor(gold) lwidth(medium)) ///
    (line norm2005_trcc Date if Date >= tm(2005m1), lcolor(red) lwidth(medium)) ///
    (line norm2005_bcom Date if Date >= tm(2005m1), lcolor(blue) lwidth(medium)) ///
    (line norm2005_sp500 Date if Date >= tm(2005m1), lcolor(black) lwidth(medium)), ///
    title("Normalized Index Performance (Start = 100)", size(medsmall)) ///
    subtitle("Shaded Areas Represent GFC & COVID-19 Crises", size(small)) ///
    ytitle("Index Level (Base = 100)", size(small)) ///
    xtitle("Date", size(small)) ///
    ylabel(, angle(horizontal) labsize(small)) ///
    xlabel(, labsize(small)) ///
    legend(order(3 "3-Metal Portfolio" 4 "TRCC" 5 "BCOM" 6 "S&P500") ///
           col(2) size(small) region(lstyle(none))) ///
    graphregion(color(white)) ///
    plotregion(margin(zero)) ///
    scheme(s1color)

* Time-Series Distribution Plot:
twoway ///
	(line lret_portfolio Date) ///
	(line lret_sp500 Date) ///
	(line lret_trcc Date) ///
	(line lret_bcom Date), ///
	  title("Time-Series Plot of Log-Returns", size(medsmall)) ///
    ytitle("Log-Returns", size(small)) ///
    xtitle("Date", size(small)) ///
    ylabel(, angle(horizontal) labsize(small)) ///
    xlabel(, labsize(small)) ///
    legend(order(1 "3-Metal Portfolio" 2 "S&P500" 3 "TRCC" 4 "BCOM") ///
           col(2) size(small) region(lstyle(none))) ///
    graphregion(color(white)) ///
    plotregion(margin(zero)) ///
    scheme(s1color)
	
*...
