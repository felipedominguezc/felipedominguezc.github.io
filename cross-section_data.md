Cross-Sectional Survey Data Analysis with Weights

```stata

set more 1

/******************************************************************************************************************************************* 

AEI Economic Policy Studies

Research Assistant Candidate Data Task
Author: Felipe Dominguez Cornejo

**********************************************************************************************************************************************/

/*Enter path to FOLDER below. Make sure to include quotation marks and avoid any unnecessary "\" */

	cd "/Users/felipedominguez/Desktop/Fall '24/Jobs & Apps/AEI" 
	use nhis_00004.dta, clear /*Enter data name here */

	svyset psu [pweight=sampweight], strata(strata) //setting up survey design


** # 2a: Diabetes Type Distribution for 2022

	tab year diabtype if diabtype <=2, row //Group proportion
	svy: tab year diabtype if diabtype <=2, row //Population Proportion
	svy, subpop(if  diabtype <=2): tab year diabtype, row //population prop with variance estimations

**# 2b: Diabetes Type Distribution for 2022 65+

	tab year diabtype if (age >= 65 & age <= 85) & diabtype <=2, row //Group proportion
	svy: tab year diabtype if (age >= 65 & age <= 85) & diabtype <=2, row //pop Proportion
	svy, subpop(if (age >= 65 & age <= 85) & diabtype <=2): tab year diabtype, row //pop prop with variance etimations

**# 2c: Diabetes Type Distribution for 2022 Obese

	tab year diabtype if bmicat == 4 & diabtype <= 2, row //group proportion
	svy: tab year diabtype if bmicat == 4 & diabtype <= 2, row //population proportion


**# 2d: Diabetes Type Distribution for 2022 65+ Obese

	tab year diabtype if (age >= 65 & age <= 85) & bmicat==4 & diabtype <=2, row //group proportion
	svy: tab year diabtype if (age >= 65 & age <= 85) & bmicat ==4 & diabtype <=2, row //pop prop


**#3 Variable for diabetes prevalence

	tab year if !missing(diabeticage)
	tab year if !missing(diabeticev) 


	gen diabetes = 0
	replace diabetes = 1 if diabeticage <=85
	/*Generates a variable relevant to diabetes prevalence: People with diabetes are coded as 1, 0 otherwise */
	
	gen diabetes1 = .
	replace diabetes1 = 1 if diabeticev == 2  // Yes = 1 (Diagnosed with Diabetes)
	replace diabetes1 = 0 if diabeticev == 1  // No = 0 (Not Diagnosed)
	
	sum diabetes*
	
	gen diabetes_final = .
	replace diabetes_final = 1 if diabeticev ==2 | diabeticage <= 85
	replace diabetes_final = 0 if diabeticev == 1  // No = 0 
	list if diabeticev == 1 & diabeticage <=85 //No conflict between them
	

**#4 Calculate proportions by subpopulations using diabetes_final

***************************************************************************************************************************

	levelsof year, local(years)
	
	tempfile prev_data
	tempname results
	postfile `results' year overall pop65 obese pop65_obese using "`prev_data'", replace
	
	foreach yr of local years {
	    * Overall prevalence for each year
	    svy: mean diabetes_final if year == `yr'
	    matrix M = r(table)
	    scalar overall = M[1,1] *100
	
	    * Prevalence among 65+ population
	    svy, subpop(if age >= 65 & age <= 85): mean diabetes_final if year == `yr'
	    matrix M = r(table)
	    scalar pop65 = M[1,1] *100
	
	    * Prevalence among obese population (bmicat==4)
	    svy, subpop(if bmicat == 4): mean diabetes_final if year == `yr'
	    matrix M = r(table)
	    scalar obese = M[1,1] *100
	
	    * Prevalence among 65+ obese population
	    svy, subpop(if age >= 65 & age <= 85 & bmicat == 4): mean diabetes_final if year == `yr'
	    matrix M = r(table)
	    scalar pop65_obese = M[1,1] *100

	    * Post the estimates for this year
	    post `results' (`yr') (overall) (pop65) (obese) (pop65_obese)
	}
	postclose `results'
	preserve
	use "`prev_data'", clear


*Graph

*********************************************************************************************************************************************

		twoway (line overall year, lcolor(red) lpattern(solid) lwidth(medium)) ///
		       (line pop65 year, lcolor(red) lpattern(dash) lwidth(medium)) ///
		       (line obese year, lcolor(blue) lpattern(solid) lwidth(medium)) ///
		       (line pop65_obese year, lcolor(blue) lpattern(dash) lwidth(medium)), ///
		       xline(2019, lcolor(black) lpattern(solid) lwidth(thin)) ///
		       title("Diabetes Prevalence Over Time") ///
		       xtitle("Year") ytitle("Prevalence (Percentage)") ///
		       legend(order(1 "Overall" 2 "65+ Population" 3 "Obese Population" 4 "65+ Obese Population")) ///
			   scale(0.9) ///
			   note("Notes: Obesity defined as BMI greater than 30." "Black line in 2019 marks the change in sampling methodology, trends should not be compared." "Source: Data are from IPUMS NHIS, University of Minnesota, www.ipums.org.")
		graph export representative_trends.jpg //automatically saves in folder

	*Check loop worked: 
		restore 
		svy: tab year diabetes_final, row //pop prop with variance 
		svy, subpop(if (age >= 65 & age <= 85)): tab year diabetes_final, row //pop prop  65+
		svy, subpop(if bmicat==4): tab year diabetes_final, row //pop prop obese
		svy, subpop(if (age >= 65 & age <= 85) & bmicat==4): tab year diabetes_final, row //pop prop 65+ obese

**5 Further Analysis

	gen age_group = .
		replace age_group = 1 if age >= 1 & age <= 15
		replace age_group = 2 if age >= 16 & age <= 30
		replace age_group = 3 if age >= 31 & age <= 45
		replace age_group = 4 if age >= 46 & age <= 60
		replace age_group = 5 if age >= 61 & age <= 75
		replace age_group = 6 if age >= 76 & age <= 85
		
	svy: tab sex, percent
	svy: tab age_group, percent
	svy: tab marstcur, percent
	svy: tab racenew, percent
	svy: tab hispeth, percent
	

