Survey Panel Data
```stata
/*****************************************************************************************
GPRL Data Task

Author: Felipe Dominguez Cornejo
March 6, 2025

*****************************************************************************************/

	set more 1
	capture log close
	cd "/Users/felipedominguez/Desktop/Fall '24/Jobs & Apps/Northwestern/2. GPRL_StataAssessment_2025/data"
	log using data_task_GPRL.txt, replace

/*******************************************

PART 1

********************************************/

**# Q1 Unique idenifiers
	use assets.dta, clear
	 isid hhid wave Asset_Type InstanceNumber //no errors
	
	use depression.dta, clear
	 isid hhid wave hhmid //no errors
	
	use demographics.dta, clear
	 isid hhid wave hhmid //no errors
	 
**#Q2 Proxy for household size

//Clean dataset
	keep if relationship <= 2 //treatment is meant for household head and spouse, so I restrict data to those individuals
	replace age = . if age < 18 | age > 100 //clean age variable to more realistic ages
	replace agemarried = . if agemarried == .d | agemarried <18
	drop villageid spouseinhouse religionother fathereducother mothereducother //drop iirelavant variables or with too little information


	bysort hhid wave: gen n_members = _N //number of members by household in each wave
	bysort hhid: egen hhsize = max(cond(wave ==1, n_members, .)) //propagate hhsize from wave 1 to wave 2
	
	 sum hhsize 
	 sum hhsize if wave ==1
	drop n_members //drop intermediate variable

//Check with different approach: 
	bysort hhid wave: egen wave_size_by_id = max(hhmid)
	bysort hhid: egen wave1_size = max(cond(wave==1, wave_size_by_id, .))
	 sum wave1_size //same results
	 sum wave1_size if wave ==1 //same results

	drop wave_size_by_id wave1_size //drop variables from second approach
	
	tempfile temp_demographics
	save `temp_demographics', replace

**# Q3: Impute missing values in currentvalue

	use assets.dta, clear
	
	bysort Asset_Type animaltype: egen mdn_currentvalue_animals = median(currentvalue) ///
		if Asset_Type==1 //median for animal assets 
	bysort Asset_Type toolcode: egen mdn_currentvalue_tools = median(currentvalue) ///
		if Asset_Type ==2 //median for tool assets
	bysort Asset_Type durablegood_code: egen mdn_currentvalue_durable = median(currentvalue) ///
		if Asset_Type ==3 //median for durable goods assets
	
	gen mdn_currentvalue = max(mdn_currentvalue_animals, mdn_currentvalue_durable, mdn_currentvalue_tools) //one median variable for all
	
	replace currentvalue = mdn_currentvalue if missing(currentvalue) //impute missing values total
	
	sort hhid wave Asset_Type InstanceNumber //return to original sorting
	drop mdn* //drop intermediate variables 

**# Q4: Total monetary value

	gen total_currentvalue = currentvalue*quantity

**# Q5: Dataset at the household-wave level

	//Category-Specific Value Variables

	gen value_animals    = cond(Asset_Type == 1, total_currentvalue, 0)
	gen value_tools      = cond(Asset_Type == 2, total_currentvalue, 0)
	gen value_durablegoods = cond(Asset_Type == 3, total_currentvalue, 0)
	
	
	//Creating datset at the household-wave level
	collapse (sum) value_animals value_tools value_durablegoods, by(hhid wave)
	
	gen total_asset_value = value_animals + value_tools + value_durablegoods
	
	label variable value_animals "Total value of animals"
	label variable value_tools "Total value of tools"
	labe variable value_durablegoods "Total value of durable goods"
	label variable total_asset_value "Total asset value"
	
	destring hhid, replace
	recast long hhid 
	format hhid %9.0f
	
	tempfile temp_asset
	save `temp_asset', replace


**# Q6: Kessler Score

	use depression.dta, clear
	
	//Check write up for full methodology explanation
	egen valid_count = rownonmiss(tired nervous sonervous hopeless restless sorestless depressed everythingeffort nothingcheerup worthless) //valid items per row 
	egen sum_valid = rowtotal(tired nervous sonervous hopeless restless sorestless depressed everythingeffort nothingcheerup worthless), missing //sum all valid items (ignoring missing values)
	
	gen kessler_score = (sum_valid / valid_count) * 10 //Scale the sum to 10 items only if the respondent is missing at most one item 
	replace kessler_score = 99 if valid_count <9 //If more than 1 item is missing, set the score to 99
	replace kessler_score = round(kessler_score)
	
	gen kessler_categories = .
	replace kessler_categories = 1 if inrange(kessler_score, 10, 19)
	replace kessler_categories = 2 if inrange(kessler_score, 20, 24)
	replace kessler_categories = 3 if inrange(kessler_score, 25, 29)
	replace kessler_categories = 4 if inrange(kessler_score, 30, 50)
	
	label define kessler_cat 1 "No significant depression" ///
	                        2 "Mild depression"          ///
	                        3 "Moderate depression"      ///
	                        4 "Severe depression"
	label values kessler_categories kessler_cat
	
	drop tired nervous sonervous hopeless restless sorestless depressed everythingeffort nothingcheerup worthless sum_valid valid_count
	tempfile temp_depression
	save `temp_depression', replace


**# Q7: Combining datasets 

	use "`temp_demographics'", clear
	sort hhid hhmid wave
	//Merge demographics with depression data
	merge 1:1 hhid hhmid wave using "`temp_depression'", keep(master match)
	tab _merge //for about 4% of individuals, the depression variables are  missing 

//Check for systematic vs random missing values for depression data

	gen missing_depression = missing(kessler_score)
	misstable summarize kessler_score
	tab treat_hh missing_depression, chi2 //not significant
	tab wave missing_depression, chi2 //SIGNIFICANT
	ttest age, by(missing_depression) //SIGNIFICANT
	
	drop _merge

//Merge resulting dataset with assets data

	sort hhid wave
	merge m:1 hhid wave using "`temp_asset'", keep(master match)
	tab _merge // <1% of variables missing
	
	//Check for systematic vs random missing values after the merge
	list hhid treat_hh gender age relationship kessler_categories wave if _merge==1
	gen asset_missing = (_merge == 1)
	tab treat_hh asset_missing, chi2 //not significant
	tab wave asset_missing, chi2 //not significant
	tab relationship asset_missing, chi2 //not significant
	
	bysort hhid hhmid: gen nobs = _N
	bysort hhid hhmid wave: gen nobs1 = _N
	//checking if there are <=2 obs per individual
	sum nobs
	sum nobs1
	drop nobs* _merge asset_missing
	
	save combined.dta, replace



/***************************************

PART 2

****************************************/

	use combined.dta, clear


**#Q1:Relationship between depression and household wealth

	gen log_total_asset_value = ln(total_asset_value) 
	gen log_tools_value = ln(value_tools)
	gen log_animals_value = ln(value_animals)
	gen log_durables = ln(value_durablegoods)
	
	preserve 
	
	keep if wave ==1
	misstable summarize kessler_score total_asset_value //Check missing values for depression and asset data
	drop if missing(kessler_score) | missing(total_asset_value)
		// Since my focus is exploratory analysis, I drop cases missing either depression or asset data
	
	sum kessler_score total_asset_value age if kessler_score != 99


//Histogram for Depression Scores with axis titles and reduced graph size

	histogram kessler_score if kessler_score != 99, percent title("Depression Scores") ///
	    xtitle("Kessler Score") ytitle("Percentage") ///
	    name(depression_hist, replace) xsize(4) ysize(3)

//Histogram for Log Household Wealth with axis titles and reduced graph size

	histogram log_total_asset_value, percent title("Log Household Wealth") ///
	    xtitle("Log Total Asset Value") ytitle("Percentage") ///
	    name(wealth_hist, replace) xsize(4) ysize(3)

//Combine the two graphs in two columns

	graph combine depression_hist wealth_hist, title("Distributions in Wave 1") cols(1) ///
		name(distributions, replace) ///
		nodraw


** Scatter + linear fit: Depression vs. Household Wealth

	twoway ///
	    (scatter kessler_score log_total_asset_value  if kessler_score != 99, ///
	        msize(vsmall) mcolor(%40) ///
	        title("Depression vs. Household Wealth") ///
	        xtitle("Log Total Asset Value") ///
	        ytitle("Kessler Score") ///
	        xlabel(, grid) ylabel(, grid)) ///
	    (lfit kessler_score log_total_asset_value, lcolor(red)), ///
	    legend(off) name(scatter_wealth, replace)

**Scatter + linear fit: Depression vs. Age
	twoway ///
	    (scatter kessler_score age if kessler_score != 99, ///
	        msize(vsmall) mcolor(%40) ///
	        title("Depression vs. Age (Wave 1)") ///
	        xtitle("Age") ///
	        ytitle("Kessler Score") ///
	        xlabel(, grid) ylabel(, grid)) ///
	    (lfit kessler_score age, lcolor(red)), ///
	    name(scatter_age, replace)

**Combine the two plots side by side
	graph combine scatter_wealth scatter_age, ///
	    title("Depression vs. Household Wealth and Age") ///
	    cols(1) ///
		name(lines, replace) ///
		nodraw



// Compute correlation coefficient with significance tests

	pwcorr kessler_score log_total_asset_value ///
		log_tools_value log_animals_value log_durables age, sig 

//simple regression

	reg kessler_score log_total_asset_value if kessler_score != 99, r
	reg kessler_score log_total_asset_value age if kessler_score != 99, r
	reg kessler_score log_durables if kessler_score != 99, r
	reg kessler_score log_durables age if kessler_score != 99, r

//Histogram of age 

	histogram age, percent title("Distribution of Age (Wave 1)") ///
		name(age, replace) ///
		nodraw

//Simple regression

	reg kessler_score age if kessler_score != 99, r	

**#Q2: Were the GT sessions effective?

	restore
	keep if wave ==2 
	
	drop if missing(kessler_score)

// Compare mean depression scores between treatment and control groups

	ttest kessler_score, by(treat_hh)

// Run a simple regression to estimate the treatment effect

	reg kessler_score i.treat_hh, r
	di "The adjusted R-squared is:" e(r2_a)
	reg kessler_score i.treat_hh age hhsize, r
	di "The adjusted R-squared is:" e(r2_a)
	reg kessler_score i.treat_hh age hhsize log_total_asset_value, r
	di "The adjusted R-squared is:" e(r2_a)

**#Q3: GT sessions by gender

	gen woman = (gender ==5) //makes binary variable for woman
	reg kessler_score i.woman##i.treat_hh, r
	testparm i.woman#i.treat_hh


	log close 
