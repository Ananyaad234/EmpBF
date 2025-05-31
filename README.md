# EmpBF
# Projects from Empirical Banking and Finance Course on Stata
----------------------------------------------------------------------------------

# 1: OLS Recap

* 1. Regression 1: We first investigate the effect of having a bank-based financial system on economic growth. To empirically investigate this we run the following regression:

reg gdpgrowth_1995_2017 bdgdp

correlate gdpgrowth bdgdp, covariance // Verifying OLS formula

twoway scatter gdpgrowth_1995_2017 bdgdp || lfit gdpgrowth_1995_2017 bdgdp // Plots regression line

twoway scatter gdpgrowth_1995_2017 bdgdp if country_id != 35|| lfit gdpgrowth_1995_2017 bdgdp if country_id != 35 // removes outlier and repeats

* 2: Weighted OLS regressions and standard errors.

reg gdpgrowth_1995_2017 bdgdp_quintiles, robust
estimates store robust // Same regression with 1995 QUINTILES OF BANK DEPOSITS to GDP DISTRIBUTION (%)

gen count_countries = 1 
collapse (sum) count_countries (mean) gdpgrowth_1995_2017, by(bdgdp_quintiles)
reg gdpgrowth_1995_2017 bdgdp_quintiles [aweight = count_countries], robust
estimates store w_robust // Weighting the regression by no. of countries in each quintile

reg gdpgrowth_1995_2017 bdgdp_quintiles [aweight = count_countries]
estimates store wnon_robust // previous, without adjustments to std. error

estimates table robust non_robust w_robust wnon_robust, b se stats(r2 N) // comparing all 4 models

twoway bar gdpgrowth_1995_2017 bdgdp_quintiles // bar plot

reg gdpgrowth_1995_2017 bdgdp_quintiles if country_id != 35, robust // previous, after removing outlier

* Regression 2: effect of "1995 BANK ASSETS to GDP (%)" on "1995-2017 growth of real GDP per capita" with "1995 STOCK MARKET CAPITALIZATION to GDP (%)" as a control

reg bdgdp stmktcap, robust
predict bdgdp_resid, residuals
regress gdpgrowth_1995_2017 bdgdp_resid //   Runs an auxillary regression

reg gdpgrowth_1995_2017 dbagdp stmktcap // 4.b

reg dbagdp stmktcap
predict dbagdp_resid, residuals
regress gdpgrowth_1995_2017 dbagdp_resid // two-step procedure

reg gdpgrowth dbagdp stmktcap, beta // comparing β1 and β2 
