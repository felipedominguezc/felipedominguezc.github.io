CPS Data with p-Weight Aggregation for LFPR Analysis

```stata

************************************************************************************************************************************************

* Author: Felipe Dominguez Cornejo
	This is a Data Task for a Pre-doc position at a top university where I moved on onto the next hiring process

************************************************************************************************************************************************

*Enter path to FOLDER below. Make sure to include quotation marks and avoid any unnecessary "\"*

	cd "..."
	use CPS_data.dta, clear /*Enter data name here */
	
	label define sex_lab 1 "Male" 2 "Female" 9 "NIU"
	label values sex sex_lab

****************************************************************************************

**#Calculate LFPR by group with weights (Variables to be used for every question)

	gen adult_w = (popstat == 1) * wtfinl //adult_civilian indicator weighted
	gen in_lab_w = (labforce == 2) * wtfinl //adult civilian in labor force indicator weighted




***************************************************************************************************

* 1 LFPR by age Profile in 1999 & 2024

***************************************************************************************************

*Create Age groups:

	gen age_group = cond(age >= 15 & age <= 19, 1, ///
	    cond(age >= 20 & age <= 24, 2, ///
	    cond(age >= 25 & age <= 34, 3, ///
	    cond(age >= 35 & age <= 44, 4, ///
	    cond(age >= 45 & age <= 54, 5, ///
	    cond(age >= 55 & age <= 64, 6, 7))))))
	
	label define age_group_labels 0 "Under 15 years" 1 "15-19 years" 2 "20-24 years" 3 "25-34 years" 4 "35-44 years" 5 "45-54 years" 6 "55-64 years" 7 "65+ years"
	label values age_group age_group_labels
	label variable age_group "Age Group"



*Overall LFPR by year and age_group

	local age_groups 1 2 3 4 5 6 7

	foreach age in `age_groups' {
	    * Calculate LFPR for each age group
	    by year age_group, sort: egen adult_civilian_overall`age' = total(((popstat == 1) & (age_group == `age')) * wtfinl)
	    by year age_group: egen labor_force_overall`age' = total(((labforce == 2) & (age_group == `age')) * wtfinl)
	    gen overall_lfpr`age' = (labor_force_overall`age' / adult_civilian_overall`age') * 100
	}
	
	label variable overall_lfpr1 "Overall LFPR for Ages 15-19"
	label variable overall_lfpr2 "Overall LFPR for Ages 20-24"
	label variable overall_lfpr3 "Overall LFPR for Ages 25-34"
	label variable overall_lfpr4 "Overall LFPR for Ages 35-44"
	label variable overall_lfpr5 "Overall LFPR for Ages 45-54"
	label variable overall_lfpr6 "Overall LFPR for Ages 55-64"
	label variable overall_lfpr7 "Overall LFPR for Ages 65+"

*LFPR for each age_group by year, age_group, and sex

	foreach age in `age_groups' {
	    by year age_group sex, sort: egen adult_civilian`age' = total(((popstat == 1) & (age_group == `age')) * wtfinl)
	    by year age_group sex: egen labor_force`age' = total(((labforce == 2) & (age_group == `age')) * wtfinl)
	    gen lfpr`age' = (labor_force`age' / adult_civilian`age') * 100
	}

*Label LFPR variables for each age group

	label variable lfpr1 "LFPR for Ages 15-19"
	label variable lfpr2 "LFPR for Ages 20-24"
	label variable lfpr3 "LFPR for Ages 25-34"
	label variable lfpr4 "LFPR for Ages 35-44"
	label variable lfpr5 "LFPR for Ages 45-54"
	label variable lfpr6 "LFPR for Ages 55-64"
	label variable lfpr7 "LFPR for Ages 65+"

******************************************************************************

* Graphs

******************************************************************************

	graph bar lfpr* if sex == 1, over(year) ///
	    title("LFPR by Age Group, Men(1999 vs. 2024)") ///
	    ytitle("Labor Force Participation Rate (%)") ///
	    legend(label(1 "Ages 15-19") label(2 "Ages 20-24") label(3 "Ages 25-34") ///
	           label(4 "Ages 35-44") label(5 "Ages 45-54") label(6 "55-64") ///
	           label(7 "Ages 65+")) ///
		name(male, replace) ///
		nodraw
			   
	graph bar lfpr* if sex == 2, over(year) ///
	    title("LFPR by Age Group, Women(1999 vs. 2024)") ///
	    ytitle("Labor Force Participation Rate (%)") ///
	    legend(label(1 "Ages 15-19") label(2 "Ages 20-24") label(3 "Ages 25-34") ///
	           label(4 "Ages 35-44") label(5 "Ages 45-54") label(6 "55-64") ///
	           label(7 "Ages 65+")) ///
		name(female, replace) ///
		nodraw
	
	graph bar overall_lfpr*, over(year) ///
	    title("LFPR by Age Group, Overall (1999 vs. 2024)") ///
	    ytitle("Labor Force Participation Rate (%)") ///
	    legend(label(1 "Ages 15-19") label(2 "Ages 20-24") label(3 "Ages 25-34") ///
	           label(4 "Ages 35-44") label(5 "Ages 45-54") label(6 "55-64") ///
	           label(7 "Ages 65+")) ///
		name(overall, replace) ///
		nodraw
	
	drop adult_civilian* labor_force*

*****************************************************************************************************

* 3 Overall LFPR by sex and year**

*****************************************************************************************************

	keep if age >= 16 
	
	gen adult_total_men      = cond(sex == 1, adult_w, 0)
	gen labor_force_total_men = cond(sex == 1, in_lab_w, 0)
	gen adult_total_women    = cond(sex == 2, adult_w, 0)
	gen labor_force_total_women = cond(sex == 2, in_lab_w, 0)
	
	preserve
	collapse (sum) adult_total_full = adult_w ///
	         (sum) labor_force_total_full = in_lab_w ///
	         (sum) adult_total_men      = adult_total_men ///
	         (sum) labor_force_total_men = labor_force_total_men ///
	         (sum) adult_total_women    = adult_total_women ///
	         (sum) labor_force_total_women = labor_force_total_women, by(year)
	
	
	gen overall_lfpr_full  = (labor_force_total_full  / adult_total_full)  * 100
	gen overall_lfpr_men   = (labor_force_total_men   / adult_total_men)   * 100
	gen overall_lfpr_women = (labor_force_total_women / adult_total_women) * 100
	
	label variable overall_lfpr_full  "Overall LFPR Full Population 16+"
	label variable overall_lfpr_men   "Overall LFPR Men 16+"
	label variable overall_lfpr_women "Overall LFPR Women 16+"
	
	tempfile LFPRresults
	save "`LFPRresults'", replace
	restore
	merge m:1 year using "`LFPRresults'", nogen
	
	drop adult_total* labor_force_total*
	
	bysort year: sum overall_lfpr_full
	bysort year: sum overall_lfpr_men
	bysort year: sum overall_lfpr_women

*****************************************************************************************************

* 4 2024 Aggregate LFPR with 1999 age distribution

*****************************************************************************************************

	*Get total adult population and labor force by year & age_group

	preserve
	
		collapse ///
		    (sum) pop = adult_w /// total adult population (weighted)
		    (sum) lf  = in_lab_w, /// total in labor force (weighted)
		    by(year age_group)
		
		reshape wide pop lf, i(age_group) j(year)
	
	* 1999 share of total adult population

		summarize pop1999
		local total_99 = r(sum)
	
	gen share_1999 = pop1999 / `total_99'

	* 2024 LFPR = lf2024 / pop2024

		gen lfpr_2024 = 100 * (lf2024 / pop2024)


	* Multiply 1999 share by 2024 LFPR and sum across age groups

		gen w_lfpr_2024 = share_1999 * lfpr_2024
	
	* The sum of w_lfpr_2024 over all age groups is the CF LFPR in 2024

		egen reweighted_2024 = total(w_lfpr_2024)
	
		display "Reweighted 2024 LFPR (if age shares stayed at 1999 levels): " ///
	        	reweighted_2024


