
## Advanced: Experimental Data Management

**Proficiency Level:** Advanced

**Overview:**  
This .do file demonstrates a robust experimental data management strategy designed for an RA application. It begins by importing and cleaning two CSV datasets—child and guardian data—by correctly recasting variable types, applying informative labels, and merging them into a combined dataset. The script then builds baseline balance tables using `estpost` and `esttab`, calculates disagreement indicators between guardian responses, and performs a difference‑in‑differences analysis to capture treatment effects from baseline to endline. Furthermore, it standardizes multiple test scores into z-scores and aggregates them into domain-specific indices (e.g., Reasoning, Language, Memory, Numeracy, Motor) before estimating treatment effects via regression, showcasing a detailed and methodical approach to experimental analysis in Stata.

**Key Techniques & Features:**
- **Data Import & Cleaning:** Reads in CSV files, recasts variable types, and applies informative labels.
- **Dataset Merging:** Combines child and guardian data to create a comprehensive dataset.
- **Baseline Balance & Disagreement Analysis:** Uses `estpost`/`esttab` to generate balance tables and calculates disagreement indicators.
- **Difference‑in‑Differences (DiD):** Captures treatment effects by comparing baseline and endline outcomes.
- **Test Score Standardization:** Computes z-scores for multiple tests and aggregates them into domain-specific indices.
- **Regression Modeling:** Runs treatment effect regressions and exports results for further analysis.

---

**Stata Code:**

```stata
set more 1
capture log close

******************************************************************************************************************************************
* Author: Felipe Dominguez Cornejo
* Data Task for RA position
*********************************************************************************************************************************************

log using "/Users/felipedominguez/Desktop/Jobs & Apps/U Chicago/Profs. Dean & Pope/Data task/Stata Files/FDC_data_task.txt", replace

*********************************************************************************************************************************************
* **Why did I write this code?**
* This is my code for the data task for an RA position with Professor Dean and Professor Pope at Chicago Booth.
* They focus on behavioral economics.
*
* **Table of Contents:**
*   - Each question has its own section in the code.
*   - I use preserve/restore commands instead of saving new .dta files.
*
* **Inputs/Outputs:**
*   - This code reads in the two .csv files, converts them into .dta files (which are used throughout).
*   - Outputs include graphs and tables needed to answer the questions.
*
* **Technical information/general assumptions:**
*   - Test scores are correct and free of errors-in-variables.
*   - Q1: I chose to use only the primary respondent because it provides a clear one-to-one comparison of guardian characteristics across treatment groups, under the assumption that the primary respondent's information accurately represents the overall guardian profile and is most relevant for the child's outcomes.
*   - Hours worked outliers: Because less than 2% of observations exceed a plausible upper bound (112 > hpw), recoding these outlier values as missing most cleanly preserves the validity of the baseline balance analysis while only marginally reducing sample size.
**********************************************************************************************************************************************

*Creating Directories*
global desktop "/Users/felipedominguez/Desktop" // Desktop directory on my mac; change to your home directory
global mainpath "$desktop/Jobs & Apps/U Chicago/Profs. Dean & Pope/Data task"  // Adjust to where files are stored
global output "$mainpath/Graphs"  // Change to where you want graphs to be saved

*Installing Packages*
ssc install tabstatmat, replace
ssc install estout, replace

*Graph Settings*
set graphics off  // Toggle graphics on/off
graph set window fontface "Times New Roman"
global titlesize "size(medium)" // Title font size
global subsize "size(medsmall)" // Axis title font size
global labsize "size(small)"      // Axis label font size

***********************************************************************************************************************************************
* **Dataset 1: Childdata**
***********************************************************************************************************************************************
import delimited "$mainpath/data/childdata.csv", clear  // Import child data from CSV file

* Set correct variable types per codebook
recast byte c_totalscore_qmat c_totalscore_bug c_totalscore_m c_totalscore_v ///
       c_totalscore_animal c_totalscore_qpns c_totalscore_bp ///
       c_totalscore_col c_totalscore_cbnr c_totalscore_act c_totalscore_alpha ///
       c_totalscore_prob c_totalscore_copy c_totalscore_pan treatment endline, force
recast float age c_totalscore_all, force
recast double c_totalscore_asm c_totalscore_ask c_totalscore_mot, force
recast long id_child, force

* Label variables
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
replace age = . if age < 0  // Mark negative age values as missing
save "$mainpath/data/childdata.dta", replace  // Save as .dta file

codebook  // Check structure
des

***********************************************************************************************************************************************
* **Dataset 2: Guardian Data**
***********************************************************************************************************************************************
import delimited "$mainpath/data/guardiandata.csv", clear  // Import guardian data CSV file

* Recast non-string variables
recast long id_child, force
recast byte endline, force
recast byte respondent, force
recast double hoursworked, force
recast float edu, force

* Convert string variables to numeric
replace prim_type = lower(prim_type)
gen prim_type_num = .  // Create numeric version
replace prim_type_num = 1 if prim_type == "private"
replace prim_type_num = 2 if prim_type == "public"
drop prim_type
rename prim_type_num prim_type
recast byte prim_type, force

* Process gender variable
replace gender = lower(gender)
gen gender_num = .
replace gender_num = 0 if gender == "male"
replace gender_num = 1 if gender == "female"
replace gender_num = . if missing(gender) | gender == ""
drop gender
rename gender_num gender

* Apply labels
label define prim_type_lab 1 "Private" 2 "Public"
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
replace hoursworked = . if hoursworked > 112  // Cap unrealistic work hours
count if missing(hoursworked)

save "$mainpath/data/guardiandata.dta", replace  // Save formatted data as .dta file

codebook  // Check structure
des

***********************************************************************************************************************************************
* **Dataset 3: Merging Child and Guardian Data**
***********************************************************************************************************************************************
use "$mainpath/data/childdata.dta", clear
sort id_child endline

merge 1:m id_child endline using "$mainpath/data/guardiandata.dta"
*** Check merge result
tab _merge  // Verify successful merge
drop _merge

save "$mainpath/data/combined.dta", replace

***********************************************************************************************************************************************
* **Question 1: Baseline Balance Table**
***********************************************************************************************************************************************
use "$mainpath/data/combined.dta", clear

preserve
    keep if endline == 0
    keep if respondent == 1

    * Control Group (treatment == 0)
    estpost tabstat age c_totalscore_qmat c_totalscore_bug c_totalscore_m ///
                c_totalscore_v c_totalscore_animal c_totalscore_rvs ///
                c_totalscore_qpns c_totalscore_all edu hoursworked prim_type if treatment==0, c(stat) stat(mean sd min max n)
    eststo control

    * Treated Group (treatment == 1)
    estpost tabstat age c_totalscore_qmat c_totalscore_bug c_totalscore_m ///
                c_totalscore_v c_totalscore_animal c_totalscore_rvs ///
                c_totalscore_qpns c_totalscore_all edu hoursworked prim_type if treatment==1, c(stat) stat(mean sd min max n)
    eststo treated

    esttab control treated using "$output/baseline_balance.csv", replace ///
        cells("mean(fmt(2)) sd min(fmt(0)) max(fmt(0)) count(fmt(0))") ///
        label mtitles("Control Group" "Treatment Group") nonumber ///
        title("Baseline Balance Table") varwidth(25) collabels("Mean" "SD" "Min" "Max" "N")
restore

***********************************************************************************************************************************************
* **Question 2: Guardian Disagreement Analysis**
***********************************************************************************************************************************************
preserve
    keep if endline == 0
    keep id_child respondent prim_type
    reshape wide prim_type, i(id_child) j(respondent)
    gen disagree = (prim_type0 != prim_type1) if !missing(prim_type0) & !missing(prim_type1)
    sum disagree
    dis "Percentage of guardians who disagree on school type: " 100 * r(mean) "%"
restore

***********************************************************************************************************************************************
* **Question 3: Difference‑in‑Differences & Summary Statistics**
***********************************************************************************************************************************************
* Baseline & Endline Summary by Treatment
table treatment endline, statistic(mean c_totalscore_all) ///
       statistic(sd c_totalscore_all) ///
       statistic(count c_totalscore_all)

* Difference‑in‑Differences Calculation
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
    
    * Graph: Density Distributions
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

***********************************************************************************************************************************************
* **Question 4: Construction of Cognitive and Motor Indices & Regression Analysis**
***********************************************************************************************************************************************
use "$mainpath/data/childdata.dta", clear
preserve
    keep if endline == 0
    local domains Reasoning Language Memory Numeracy Motor

    * List of variables for each domain
    local Reasoning qmat bug animal prob 
    local Language v rvs qpns ask
    local Memory m
    local Numeracy asm pan cbnr
    local Motor mot copy bp col act alpha

    * Compute control-group means & SDs for each test
    foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
        quietly summarize c_totalscore_`var' if treatment==0
        local m`var' = r(mean)
        local sd`var' = r(sd)
    }
    
    * Generate z-scores
    foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
        gen z_`var' = (c_totalscore_`var' - `m`var'')/`sd`var''
    }

    * Build one index per domain
    foreach dom of local domains {
        local list
        foreach v of local `dom' {
            local list `list' z_`v'
        }
        egen idx_`dom' = rowmean(`list')
        label variable idx_`dom' "`dom' summary index base"
    }
    drop z_*

    tempfile temp_baseline_scores
    save `temp_baseline_scores', replace
restore

* Repeat procedure for endline
keep if endline == 1
local domains Reasoning Language Memory Numeracy Motor

* List of variables for each domain
local Reasoning qmat bug prob
local Language v rvs qpns ask
local Memory m
local Numeracy asm pan cbnr
local Motor mot copy bp col act alpha

* Compute control-group means & SDs for each test
foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
    quietly summarize c_totalscore_`var' if treatment==0
    local m`var' = r(mean)
    local sd`var' = r(sd)
}
    
* Generate z-scores
foreach var in qmat bug m v animal rvs qpns asm ask mot bp col cbnr act alpha prob copy pan {
    gen z_`var' = (c_totalscore_`var' - `m`var'')/`sd`var''
}

* Build one index per domain
foreach dom of local domains {
    local list
    foreach v of local `dom' {
        local list `list' z_`v'
    }
    egen idx_`dom' = rowmean(`list')
    label variable idx_`dom' "`dom' summary index"
}
drop z_*

tempfile temp_endline_scores
save `temp_endline_scores', replace

* Reload full dataset and merge scores
use "$mainpath/data/combined.dta", clear

merge m:1 id_child using `temp_baseline_scores'
rename (idx_Reasoning idx_Language idx_Memory idx_Numeracy idx_Motor) ///
       (idx_Reasoning_base idx_Language_base idx_Memory_base ///
        idx_Numeracy_base idx_Motor_base)
drop _merge

merge m:1 id_child using `temp_endline_scores'
drop _merge

* Prepare dataset for regression
gen mother_edu = .
replace mother_edu = edu if gender == 1
gen base_age = age if endline == 0
bysort id_child (endline): replace base_age = base_age[_n-1] if missing(base_age)
bysort id_child (endline): replace mother_edu = mother_edu[_n-1] if missing(mother_edu)
drop c_totalscore* hoursworked prim_type edu age respondent
keep if endline == 1  
keep if gender == 1

* Perform regression analyses for each index
foreach idx in Reasoning Language Memory Numeracy Motor {
    regress idx_`idx' treatment base_age mother_edu idx_`idx'_base
    eststo `idx'
}

* Export regression results
esttab Reasoning Language Memory Numeracy Motor using "$output/treatment_effects.csv", ///
    replace se star(* 0.10 ** 0.05 *** 0.01) ///
    label title("Treatment Effects on Cognitive and Motor Indices") ///
    coeflabels(treatment "Treatment Effect") ///
    stats(r2 N, labels("R-squared" "Observations"))
log close
```

---

**Insights:**  
- **Data Integration:** The script efficiently merges child and guardian datasets after thorough cleaning and recasting.
- **Baseline Balance & Disagreement Checks:** It uses `estpost` and `esttab` to build balance tables and evaluate guardian response consistency.
- **Difference‑in‑Differences Approach:** Captures treatment effects by comparing baseline and endline outcomes.
- **Test Score Standardization:** Z-scores are generated and aggregated into domain-specific indices for nuanced regression analysis.
- **Comprehensive Regression Modeling:** Sequential models incorporate various controls and interaction terms to assess treatment effects on cognitive and motor outcomes.
