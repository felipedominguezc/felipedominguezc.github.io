
## Advanced: Survey Panel Data Analysis with Weights

**Proficiency Level:** Advanced

**Overview:**  
This .do file orchestrates a multifaceted survey panel data analysis workflow. In Part 1, it meticulously processes multiple datasets—assets, depression, and demographics—by verifying unique identifiers, cleaning and imputing missing asset values using median calculations across asset types, and aggregating data at the household-wave level. It further computes the Kessler depression score using several mental health indicators, categorizes depression severity, and carefully merges the datasets while checking for systematic missingness. In Part 2, the script explores the relationship between household wealth and depression through descriptive statistics, graphical visualizations (histograms and scatter plots with linear fits), and regression models. Additionally, it assesses the impact of treatment interventions across different waves and by gender, showcasing advanced data handling, merging techniques, and inferential analysis in Stata.

**Key Techniques & Features:**
- **Data Cleaning & Imputation:** Verifies unique identifiers and imputes missing asset values via median calculations.
- **Aggregation & Merging:** Collapses data to the household-wave level and merges datasets while assessing systematic missingness.
- **Depression Scoring:** Constructs the Kessler depression score, categorizing depression severity.
- **Descriptive & Inferential Analysis:** Generates histograms, scatter plots with linear fits, and runs regression models to examine the association between household wealth and depression.
- **Treatment Analysis:** Evaluates the effectiveness of treatment interventions, including subgroup analysis by gender.

---

**Stata Code:**

```stata
***********************************************************************************************************************************************
* AEI Economic Policy Studies
* Research Assistant Candidate Data Task
* Author: Felipe Dominguez Cornejo
************************************************************************************************************************************************

*Enter path to FOLDER below. Make sure to include quotation marks and avoid any unnecessary "\"
cd "/Users/felipedominguez/Desktop/Fall '24/Jobs & Apps/Northwestern/2. GPRL_StataAssessment_2025/data"
use nhis_00004.dta, clear /*Enter data name here*/

* Set up survey design
svyset psu [pweight=sampweight], strata(strata)

*******************************************
* PART 1
********************************************

**Q1 Unique identifiers
    use assets.dta, clear
    isid hhid wave Asset_Type InstanceNumber // no errors

    use depression.dta, clear
    isid hhid wave hhmid // no errors

    use demographics.dta, clear
    isid hhid wave hhmid // no errors

**Q2 Proxy for household size
    // Clean dataset
    keep if relationship <= 2  // restrict data to household head and spouse
    replace age = . if age < 18 | age > 100  // realistic ages only
    replace agemarried = . if agemarried == .d | agemarried < 18
    drop villageid spouseinhouse religionother fathereducother mothereducother  // drop irrelevant variables

    bysort hhid wave: gen n_members = _N  // count members per wave
    bysort hhid: egen hhsize = max(cond(wave ==1, n_members, .))  // propagate wave 1 household size
    sum hhsize 
    sum hhsize if wave == 1
    drop n_members

    // Alternative approach check:
    bysort hhid wave: egen wave_size_by_id = max(hhmid)
    bysort hhid: egen wave1_size = max(cond(wave==1, wave_size_by_id, .))
    sum wave1_size 
    sum wave1_size if wave ==1
    drop wave_size_by_id wave1_size

    tempfile temp_demographics
    save `temp_demographics', replace

**Q3: Impute missing values in currentvalue
    use assets.dta, clear
    bysort Asset_Type animaltype: egen mdn_currentvalue_animals = median(currentvalue) if Asset_Type==1
    bysort Asset_Type toolcode: egen mdn_currentvalue_tools = median(currentvalue) if Asset_Type ==2
    bysort Asset_Type durablegood_code: egen mdn_currentvalue_durable = median(currentvalue) if Asset_Type ==3
    gen mdn_currentvalue = max(mdn_currentvalue_animals, mdn_currentvalue_durable, mdn_currentvalue_tools)
    replace currentvalue = mdn_currentvalue if missing(currentvalue)
    sort hhid wave Asset_Type InstanceNumber
    drop mdn*

**Q4: Total monetary value
    gen total_currentvalue = currentvalue * quantity

**Q5: Dataset at the household-wave level
    // Category-Specific Value Variables
    gen value_animals = cond(Asset_Type == 1, total_currentvalue, 0)
    gen value_tools = cond(Asset_Type == 2, total_currentvalue, 0)
    gen value_durablegoods = cond(Asset_Type == 3, total_currentvalue, 0)
    
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

**Q6: Kessler Score
    use depression.dta, clear
    // Compute valid items and total score for depression indicators
    egen valid_count = rownonmiss(tired nervous sonervous hopeless restless sorestless depressed everythingeffort nothingcheerup worthless)
    egen sum_valid = rowtotal(tired nervous sonervous hopeless restless sorestless depressed everythingeffort nothingcheerup worthless), missing
    gen kessler_score = (sum_valid / valid_count) * 10
    replace kessler_score = 99 if valid_count < 9
    replace kessler_score = round(kessler_score)
    gen kessler_categories = .
    replace kessler_categories = 1 if inrange(kessler_score, 10, 19)
    replace kessler_categories = 2 if inrange(kessler_score, 20, 24)
    replace kessler_categories = 3 if inrange(kessler_score, 25, 29)
    replace kessler_categories = 4 if inrange(kessler_score, 30, 50)
    label define kessler_cat 1 "No significant depression" 2 "Mild depression" 3 "Moderate depression" 4 "Severe depression"
    label values kessler_categories kessler_cat
    drop tired nervous sonervous hopeless restless sorestless depressed everythingeffort nothingcheerup worthless sum_valid valid_count
    tempfile temp_depression
    save `temp_depression', replace

**Q7: Combining datasets
    use "`temp_demographics'", clear
    sort hhid hhmid wave
    merge 1:1 hhid hhmid wave using "`temp_depression'", keep(master match)
    tab _merge  // ~4% missing depression data
    gen missing_depression = missing(kessler_score)
    misstable summarize kessler_score
    tab treat_hh missing_depression, chi2
    tab wave missing_depression, chi2
    ttest age, by(missing_depression)
    drop _merge

    // Merge with assets data
    sort hhid wave
    merge m:1 hhid wave using "`temp_asset'", keep(master match)
    tab _merge
    list hhid treat_hh gender age relationship kessler_categories wave if _merge==1
    gen asset_missing = (_merge == 1)
    tab treat_hh asset_missing, chi2
    tab wave asset_missing, chi2
    tab relationship asset_missing, chi2
    bysort hhid hhmid: gen nobs = _N
    bysort hhid hhmid wave: gen nobs1 = _N
    sum nobs
    sum nobs1
    drop nobs* _merge asset_missing
    save combined.dta, replace

************************************************************************************************************************************************
* PART 2
************************************************************************************************************************************************
use combined.dta, clear

**Q1: Relationship between depression and household wealth
    gen log_total_asset_value = ln(total_asset_value)
    gen log_tools_value = ln(value_tools)
    gen log_animals_value = ln(value_animals)
    gen log_durables = ln(value_durablegoods)
    
    preserve 
    keep if wave == 1
    misstable summarize kessler_score total_asset_value
    drop if missing(kessler_score) | missing(total_asset_value)
    sum kessler_score total_asset_value age if kessler_score != 99

    // Histogram for Depression Scores
    histogram kessler_score if kessler_score != 99, percent title("Depression Scores") ///
        xtitle("Kessler Score") ytitle("Percentage") name(depression_hist, replace) xsize(4) ysize(3)

    // Histogram for Log Household Wealth
    histogram log_total_asset_value, percent title("Log Household Wealth") ///
        xtitle("Log Total Asset Value") ytitle("Percentage") name(wealth_hist, replace) xsize(4) ysize(3)

    // Combine the two graphs
    graph combine depression_hist wealth_hist, title("Distributions in Wave 1") cols(1) name(distributions, replace) nodraw

    ** Scatter + Linear Fit: Depression vs. Household Wealth
    twoway ///
        (scatter kessler_score log_total_asset_value if kessler_score != 99, msize(vsmall) mcolor(%40) ///
            title("Depression vs. Household Wealth") xtitle("Log Total Asset Value") ytitle("Kessler Score") ///
            xlabel(, grid) ylabel(, grid)) ///
        (lfit kessler_score log_total_asset_value, lcolor(red)), legend(off) name(scatter_wealth, replace)

    ** Scatter + Linear Fit: Depression vs. Age
    twoway ///
        (scatter kessler_score age if kessler_score != 99, msize(vsmall) mcolor(%40) ///
            title("Depression vs. Age (Wave 1)") xtitle("Age") ytitle("Kessler Score") ///
            xlabel(, grid) ylabel(, grid)) ///
        (lfit kessler_score age, lcolor(red)), name(scatter_age, replace)

    ** Combine the two plots
    graph combine scatter_wealth scatter_age, title("Depression vs. Household Wealth and Age") cols(1) name(lines, replace) nodraw

    // Compute correlation coefficients
    pwcorr kessler_score log_total_asset_value log_tools_value log_animals_value log_durables age, sig 

    // Simple regressions
    reg kessler_score log_total_asset_value if kessler_score != 99, r
    reg kessler_score log_total_asset_value age if kessler_score != 99, r
    reg kessler_score log_durables if kessler_score != 99, r
    reg kessler_score log_durables age if kessler_score != 99, r

    // Histogram of age
    histogram age, percent title("Distribution of Age (Wave 1)") name(age, replace) nodraw

    // Simple regression for age effect
    reg kessler_score age if kessler_score != 99, r

**Q2: Were the GT sessions effective?
    restore
    keep if wave == 2 
    drop if missing(kessler_score)
    // Compare mean depression scores between treatment and control groups
    ttest kessler_score, by(treat_hh)
    // Simple regression to estimate treatment effect
    reg kessler_score i.treat_hh, r
    di "The adjusted R-squared is:" e(r2_a)
    reg kessler_score i.treat_hh age hhsize, r
    di "The adjusted R-squared is:" e(r2_a)
    reg kessler_score i.treat_hh age hhsize log_total_asset_value, r
    di "The adjusted R-squared is:" e(r2_a)

**Q3: GT sessions by gender
    gen woman = (gender == 5) // Binary variable for woman
    reg kessler_score i.woman##i.treat_hh, r
    testparm i.woman#i.treat_hh

log close
```

---

**Insights:**  
- **Data Integration & Cleaning:** The script effectively integrates multiple datasets by validating unique identifiers, cleaning asset data, and imputing missing values using medians.
- **Depression Scoring:** It constructs a composite Kessler depression score and categorizes depression severity.
- **Graphical & Inferential Analysis:** The file generates histograms, scatter plots with linear fits, and runs regressions to analyze the relationship between household wealth and depression, while also assessing treatment effects.
- **Advanced Techniques:** Use of loops, temporary files, and survey commands demonstrates sophisticated data handling and merging techniques.
