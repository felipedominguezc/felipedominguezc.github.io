Experimental Data Management

```stata

	set more 1
	capture log close

/******************************************************************************************************************************************

Author: Felipe Dominguez Cornejo
U Chicago Booth School of Business
Data Task for RA position with Professors Pope & Dean

*********************************************************************************************************************************************/

	log using "/Users/felipedominguez/Desktop/Jobs & Apps/U Chicago/Profs. Dean & Pope/Data task/Stata Files/FDC_data_task.txt", replace

/******************************************************************************************************
	Why did I write this code?
		This is my coed for the data task for an RA position with Professor Dean and Professor Pope at 
		Chicago Booth. They focus on behavioral economics.
	
	Table of Contents:
		Each question has its own section in the code. I use preserve/restore commdands instead of 
		saving new .dta files
	
	Inputs/Outputs:
		This code reads in the two .csv files, converts them into .dta files (which are used throughout). 
		Outputs include graphs and tables needed to answer the questions
	
	Technical information/general assumptions:
		Test scores are correct and free of errors-in-variables
		Q1: I chose to use only the primary respondent because it provides a clear one-to-one comparison of guardian characteristics across treatment groups, under the assumption that the primary respondent's information accurately represents the overall guardian profile and is most relevant for the child's outcomes, thereby reducing complexity and minimizing measurement error.
		Hours worked outliers: Because less than 2% of observations exceed a plausible upper bound (112>hpw), recoding these outlier values as missing most cleanly preserves the validity of the baseline balance analysis while only marginally reducing sample size.
	
******************************************************************************************************/
	
*** Creating Directories ***

	global desktop "/Users/felipedominguez/Desktop" //desktop directory on my mac. change to your home directory
	global mainpath "$desktop/Jobs & Apps/U Chicago/Profs. Dean & Pope/Data task"
		//change to where files are stored. subfolders are data, graphs, Stata Files
	global output "$mainpath/Graphs" //change to where you want graphs to be saved

*** Installing Packages ***

	ssc install tabstatmat, replace
	ssc install estout, replace

*** Graphs Settings ***

	// set graphics on
	 set graphics off //toggle
	graph set window fontface "Times New Roman"
	global titlesize "size(medium)" // titles will have size medium
	global subsize "size(medsmall)" //axis titles size medsmall
	global labsize "size(small)" //axis labels size small

/****************************************************************

Dataset 1: Childdata

*****************************************************************/

	import delimited "$mainpath/data/childdata.csv", clear //import childata  from .csv file

*Set the correct variable types, according to codebook*

	recast byte c_totalscore_qmat c_totalscore_bug c_totalscore_m c_totalscore_v ///
	       c_totalscore_animal c_totalscore_qpns c_totalscore_bp ///
	       c_totalscore_col c_totalscore_cbnr c_totalscore_act c_totalscore_alpha ///
	       c_totalscore_prob c_totalscore_copy c_totalscore_pan treatment endline, force
	recast float age c_totalscore_all, force
	recast double c_totalscore_asm c_totalscore_ask c_totalscore_mot, force
	recast long id_child, force

*Label variables according to codebook*

	label variable c_totalscore_qmat "Score on Matrix Reasoning"
	label variable c_totalscore_bug "Score on Bug Search"
	label variable c_totalscore_m "Score on Picture Memory"
	label variable c_totalscore_v "Score on Vocabulary"
	label variable c_totalscore_animal "Score on Animal Coding"
	label variable c_totalscore_qpns "Score on Picture Naming"
	label variable c_totalscore_asm "Score on ASER Math"
	label variable c_totalscore_ask "Score on ASER Kannada"
	label variable c_totalscore_mot "Score on Motor Skills"
	label variable c_totalscore_bp "Score on Body Parts"
	label variable c_totalscore_col "Score on Colors"
	label variable c_totalscore_rvs "Score on Receptive Vocab"
	label variable c_totalscore_cbnr "Score on DIAL Number"
	label variable c_totalscore_act "Score on Actions"
	label variable c_totalscore_alpha "Score on Alphabet"
	label variable c_totalscore_prob "Score on Problem Solving"
	label variable c_totalscore_copy "Score on Copying"
	label variable c_totalscore_pan "Score on Panamath"
	label variable treatment "Treatment"
	label variable age "Age in Months"
	label variable c_totalscore_all "Total Score"
	
	label define treat_label 0 "Control" 1 "Treated"
	label values treatment treat_label
	
	label define endline_label 0 "Baseline" 1 "Endline"
	label values endline endline_label
	
	order id_child endline treatment age 
	replace age = . if age <0 //Marking as missing values
	save "$mainpath/data/childdata.dta", replace //saving it as a .dta file
	
	codebook //check its the same 
	des 


/****************************************************************

Dataset 2: guardian data

*****************************************************************/


	import delimited "$mainpath/data/guardiandata.csv", clear // Import the guardian data CSV file


/* --- Recast non-string variables ---*/
	recast long id_child, force
	recast byte endline, force
	recast byte respondent, force
	recast double hoursworked, force
	recast float edu, force

/* --- Convert string variables to numeric ---*/

	replace prim_type = lower(prim_type)


/* Create a new numeric variable from prim_type */

	gen prim_type_num = . //missing values already marked as missing values 
	replace prim_type_num = 1 if prim_type == "private"
	replace prim_type_num = 2 if prim_type == "public"
	drop prim_type
	rename prim_type_num prim_type
	recast byte prim_type, force


/* For gender */

	replace gender = lower(gender)
	gen gender_num = .
	replace gender_num = 0 if gender == "male"
	replace gender_num = 1 if gender == "female"
	replace gender_num = . if missing(gender) | gender == ""
	drop gender
	rename gender_num gender

***Labelling

	label define prim_type_lab  1 "Private" 2 "Public"
	label values prim_type prim_type_lab
	
	label define gender_lab 0 "Male" 1 "Female"
	label values gender gender_lab
	
	label variable endline "Endline Indicator"
	label variable respondent "Respondent Indicator"
	label variable hoursworked "Hours Worked"
	label variable edu "Years of Education"
	label variable prim_type "Type of primary school planned"
	label variable gender "Gender of Guardian/Respondent"
	
	order id_child endline gender
	replace hoursworked = . if hoursworked > 112 //Allows for a max of 16 hours a day, making it more realistic
	count if missing(hoursworked)

*** Save the formatted data as a .dta file

	save "$mainpath/data/guardiandata.dta", replace

	codebook //check its the same
	des

/****************************************************************

Dataset 3: Merging child and guardian data

*****************************************************************/

*** Open the child data (which has all the child variables)

	use "$mainpath/data/childdata.dta", clear
	sort id_child endline

*** Merge the guardian data (which has been formatted according to the codebook)

	merge 1:m id_child endline using "$mainpath/data/guardiandata.dta"

*** Check the merge result

	tab _merge //merge worked 
	drop _merge

	save "$mainpath/data/combined.dta", replace

/******************************************************************************************************

Question 1

******************************************************************************************************/

	use "$mainpath/data/combined.dta", clear

/*-----------------------------------------
* Baseline Balance Table 
*-----------------------------------------*/

		preserve
	    * Restrict to baseline observations
	    keep if endline == 0
		keep if respondent == 1
	
	    * For the control group (treatment==0)
	    estpost tabstat age c_totalscore_qmat c_totalscore_bug c_totalscore_m ///
	                c_totalscore_v c_totalscore_animal c_totalscore_rvs ///
	                c_totalscore_qpns c_totalscore_all edu hoursworked prim_type if treatment==0, c(stat) stat(mean sd min max n)
	    eststo control
	
	    * For the treated group (treatment==1)
	    estpost tabstat age c_totalscore_qmat c_totalscore_bug c_totalscore_m ///
	                c_totalscore_v c_totalscore_animal c_totalscore_rvs ///
	                c_totalscore_qpns c_totalscore_all edu hoursworked prim_type if treatment==1, c(stat) stat(mean sd min max n)
	    eststo treated
	
	    esttab control treated using "$output/baseline_balance.csv", ///
			replace ///
			cells("mean(fmt(2)) sd min(fmt(0)) max(fmt(0)) count(fmt(0))") /// 
			label mtitles("Control Group" "Treatment Group") nonumber ///
	           title("Baseline Balance Table") varwidth(25) collabels("Mean" "SD" "Min" "Max" "N")
			   
		*Raw table estimates are exported to a .csv file where I just have to format the table
	restore

/******************************************************************************************************

Question 2

******************************************************************************************************/

	preserve 
	keep if endline ==0 //more obs with complete info
	    * Keep only the variables needed for the disagreement calculation
	    keep id_child respondent prim_type
	
	    * Reshape so each child's guardian prim_type responses are in one row
	    reshape wide prim_type, i(id_child) j(respondent)
	
	    * Create an indicator for disagreement (if both responses are non-missing)
	    gen disagree = (prim_type0 != prim_type1) if !missing(prim_type0) & !missing(prim_type1)
	
	    * Calculate and display the percentage of disagreement
	    sum disagree
	    dis "Percentage of guardians who disagree on school type: " 100 * r(mean) "%"
	restore


/******************************************************************************************************

Question 3

******************************************************************************************************/

***
	* 1) Baseline & endline summary by treatment
		table treatment endline, statistic(mean c_totalscore_all) ///
		                        statistic(sd c_totalscore_all) ///
		                        statistic(count c_totalscore_all)
	
	* 2) Difference‑in‑differences calculation
		preserve
		    keep if !missing(c_totalscore_all)
		
		    quietly summarize c_totalscore_all if treatment==1 & endline==0
		    local m_treat_base = r(mean)
		    quietly summarize c_totalscore_all if treatment==1 & endline==1
		    local m_treat_end = r(mean)
		
		    quietly summarize c_totalscore_all if treatment==0 & endline==0
		    local m_ctrl_base = r(mean)
		    quietly summarize c_totalscore_all if treatment==0 & endline==1
		    local m_ctrl_end = r(mean)
		
		    display "Endline difference (Treated–Control): " %6.3f (`m_treat_end' - `m_ctrl_end')
		    display "Difference‑in‑differences: " %6.3f ((`m_treat_end'-`m_treat_base') - (`m_ctrl_end'-`m_ctrl_base'))
		
	* 3) Graph	    
		twoway ///
		    (kdensity c_totalscore_all if treatment==0 & endline==0, lcolor(blue) lpattern(solid)) ///
		    (kdensity c_totalscore_all if treatment==1 & endline==0, lcolor(red) lpattern(solid)) ///
		    (kdensity c_totalscore_all if treatment==0 & endline==1, lcolor(blue) lpattern(dash)) ///
		    (kdensity c_totalscore_all if treatment==1 & endline==1, lcolor(red) lpattern(dash)), ///
		    legend(order(1 "Control Baseline" 2 "Treatment Baseline" 3 "Control Endline" 4 "Treatment Endline")) ///
		    title("Distribution of Total Score by Treatment and Time", $titlesize) ///
		    xtitle("Total Score", $subsize) ytitle("Density", $subsize)
			graph export "$output/totalscore_distributions.png", replace
		
		restore
	

/******************************************************************************************************

Question 4

******************************************************************************************************/

***
	use "$mainpath/data/childdata.dta", clear
	preserve

	* 1)	
		keep if endline == 0
	
		local domains Reasoning Language Memory Numeracy Motor
	
	* List of variables for each domain
		local Reasoning qmat bug animal prob 
		local Language v rvs qpns ask
		local Memory m
		local Numeracy asm pan cbnr
		local Motor mot copy bp col act alpha

	* 2) Compute control-group means & SDs for each test

		foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
		    quietly summarize c_totalscore_`var' if treatment==0
		    local m`var' = r(mean)
		    local sd`var' = r(sd)
		}
		
	* 3) Generate z-scores

		foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
		    gen z_`var' = (c_totalscore_`var' - `m`var'')/`sd`var''
		}

* 4) Build one index per domain

	foreach dom of local domains {
	    local vars = "`dom'"
	    local list
	    foreach v of local `dom' {
	        local list `list' z_`v'
	    }
	    egen idx_`dom' = rowmean(`list')
	    label variable idx_`dom' "`dom' summary index base"
	}
	drop z_* //get rid of intermediate variables

	tempfile temp_baseline_scores
	save `temp_baseline_scores', replace

	restore

*** Repeat procedure for endline ***

	keep if endline == 1

	local domains Reasoning Language Memory Numeracy Motor

	* List of variables for each domain

		local Reasoning qmat bug prob
		local Language v rvs qpns ask
		local Memory m
		local Numeracy asm pan cbnr
		local Motor mot copy bp col act alpha

	* 2) Compute control-group means & SDs for each test

		foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
		    quietly summarize c_totalscore_`var' if treatment==0
		    local m`var' = r(mean)
		    local sd`var' = r(sd)
		}
		
	* 3) Generate z-scores
		foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
		    gen z_`var' = (c_totalscore_`var' - `m`var'')/`sd`var''
		}

	* 4) Build one index per domain
		foreach dom of local domains {
		    local vars = "`dom'"
		    local list
		    foreach v of local `dom' {
		        local list `list' z_`v'
		    }
		    egen idx_`dom' = rowmean(`list')
		    label variable idx_`dom' "`dom' summary index"
		}
		drop z_* //get rid of intermediate variables
		
		* Save endline indices before merging
		tempfile temp_endline_scores
		save `temp_endline_scores', replace

***Reload full dataset

	use "$mainpath/data/combined.dta", clear

	* Merge baseline scores

		merge m:1 id_child  using `temp_baseline_scores'
		rename (idx_Reasoning idx_Language idx_Memory idx_Numeracy idx_Motor) ///
		       (idx_Reasoning_base idx_Language_base idx_Memory_base ///
		        idx_Numeracy_base idx_Motor_base)
		
		
		drop _merge

	*Merge endline scores

		merge m:1 id_child using `temp_endline_scores'
		drop _merge

/*-----------------------------------------

* Prepare dataset for regression

*-----------------------------------------*/

***
	* Create mother_edu only for female guardians

		gen mother_edu = .
		replace mother_edu = edu if gender == 1
		
	* Generate baseline age variable (only for baseline observations)

		gen base_age = age if endline == 0
	
	* Propagate values to all observations for each child

		bysort id_child (endline): replace base_age = base_age[_n-1] if missing(base_age)
		bysort id_child (endline): replace mother_edu = mother_edu[_n-1] if missing(mother_edu)
	
	* Drop unnecessary variables

		drop c_totalscore* hoursworked prim_type edu age respondent
	
	* Keep only endline observations (for treatment effect estimation)

		keep if endline == 1  
		keep if gender == 1

/*-----------------------------------------

* Perform the regression

*-----------------------------------------*/

*** Run regressions for each index

	foreach idx in Reasoning Language Memory Numeracy Motor {
	    regress idx_`idx' treatment  base_age mother_edu idx_`idx'_base
	    eststo `idx'
	}

*** Export regression results

	esttab Reasoning Language Memory Numeracy Motor using "$output/treatment_effects.csv", ///
	    replace se star(* 0.10 ** 0.05 *** 0.01) ///
	    label title("Treatment Effects on Cognitive and Motor Indices") ///
	    coeflabels(treatment "Treatment Effect") ///
	    stats(r2 N, labels("R-squared" "Observations"))
	// Exported as .csv ready to format
	log close
	
	

