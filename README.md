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

-----------------------------------------------------------------------------------

# 2: Instrumental Variables, This exercise follows [Levine et al., 2000] using an instrumental variable based on [Porta et al., 1998]
to study the relationship between finance and growth. Uses national legal origin ([Porta et al., 1998]) of a country as an instrument for
its financial development.

ssc install catplot
ssc install ivreg2
ssc install ranktest

* 1: legal origin Instrument (legor)

catplot, over(legor) ///
 ytitle("Frequency") title("Frequency of Legal Origins") // Freq plot
graph bar (mean) credit_t_gdp_1960, over(legor) ///
 ytitle("Mean Credit-to-GDP (1960)") ///
 title("Average Credit-to-GDP by Legal Origin") 
tabstat credit_t_gdp_1960, by(legor) // calculates mean value of credit to gdp for each level of legor
 
* 2: regression 1
reg gdpgrowth_1960_1980 credit_t_gdp_1960
reg gdpgrowth_1960_1995 credit_t_gdp_1960
reg gdpgrowth_1960_2020 credit_t_gdp_1960 // runs the regressions

ivreg2 gdpgrowth_1960_1980 (credit_t_gdp_1960 = legor_fr), robust
ivreg2 gdpgrowth_1960_1995 (credit_t_gdp_1960 = legor_fr), robust
ivreg2 gdpgrowth_1960_2020 (credit_t_gdp_1960 = legor_fr), robust // Two-Stage-Least-Squares

* 3: IV with controls and interactions
ivreg2 gdpgrowth_1960_1995 (credit_t_gdp_1960 = legor_fr) pop_1960, robust // includes an exogenous control variable

reg credit_t_gdp_1960 legor_uk legor_fr legor_so legor_ge pop_1960, robust 
test (legor_uk=0) (legor_fr=0) (legor_so=0) (legor_ge=0) // Regress credit_t_gdp_1960 on the four legal origin dummies

reg gdpgrowth_1960_1995 c.credit_t_gdp_1960##c.pop_1960, robust // includes population interaction term

ivreg2 gdpgrowth_1960_1995 (c.credit_t_gdp_1960 c.credit_t_gdp_1960#c.pop_1960 = i.legor_fr c.legor_fr#c.pop_1960) c.pop_1960, robust // Two-Stage-Least-Squares with French Legal Origin as instrument for credit to GDP (1960)


* 4: IV with several instruments

ivreg2 gdpgrowth_1960_1995 (credit_t_gdp_1960 = legor_uk legor_fr legor_so legor_ge), robust // Two-Stage-Least-Squares using all 4 Legors as instrument for credit to GDP (1960)
reg credit_t_gdp_1960 legor_uk legor_fr legor_so legor_ge, robust
test (legor_uk=0) (legor_fr=0) (legor_so=0) (legor_ge=0) 
ivreg2 gdpgrowth_1960_1995 (credit_t_gdp_1960 = legor_uk legor_fr legor_so legor_ge), robust // For testing whether the instruments are valid
