CPS Data with p-Weight Aggregation for LFPR Analysis

This .do file expertly processes CPS data by first setting up the workspace and defining necessary labels and variables, then generates weighted indicators for adult status and labor force participation. It leverages conditional statements and looping constructs to create and analyze age groups, calculating both overall and gender-specific LFPR across the years 1999 and 2024. The script further demonstrates proficiency in data transformation, aggregation (including reweighting based on 1999 age shares), and visualization by generating comprehensive bar graphs to illustrate trends, making it an excellent exemplar of advanced data manipulation and analytical reporting in Stata.

## Advanced: CPS Data with p-Weight Aggregation for LFPR Analysis

**Proficiency Level:** Advanced

**Overview:**  
This Stata script demonstrates advanced data manipulation techniques such as weighted aggregations, looping constructs, dynamic recoding, and data visualization. It calculates the Labor Force Participation Rate (LFPR) across age groups and by sex for the years 1999 and 2024, and produces comparative bar graphs to illustrate trends over time.

**Key Techniques:**
- **Weighted Aggregation:** Uses sample weights to accurately estimate adult population and labor force participation.
- **Looping Constructs:** Iterates through age groups to dynamically compute LFPR for each subgroup.
- **Dynamic Recoding:** Creates age groups using conditional statements and labels them with meaningful categories.
- **Data Visualization:** Generates bar graphs to compare LFPR by gender and overall, facilitating a clear interpretation of trends.


**Stata Code:**

```stata
*******************************************************
* Author: Felipe Dominguez Cornejo
* Data Task for a Pre-doc position at a top university
*******************************************************

* Set directory and load data
cd "..."
use CPS_data.dta, clear /*Enter data name here */

* Define and apply labels for sex
label define sex_lab 1 "Male" 2 "Female" 9 "NIU"
label values sex sex_lab

****************************************************************************************
* Calculate LFPR by group with weights
****************************************************************************************
gen adult_w = (popstat == 1) * wtfinl // Adult civilian indicator weighted
gen in_lab_w = (labforce == 2) * wtfinl // Adult civilian in labor force indicator weighted

***************************************************************************************************
* **1 LFPR by Age Profile in 1999 & 2024**
***************************************************************************************************
* Create Age groups:
gen age_group = cond(age >= 15 & age <= 19, 1, ///
    cond(age >= 20 & age <= 24, 2, ///
    cond(age >= 25 & age <= 34, 3, ///
    cond(age >= 35 & age <= 44, 4, ///
    cond(age >= 45 & age <= 54, 5, ///
    cond(age >= 55 & age <= 64, 6, 7))))))
label define age_group_labels 0 "Under 15 years" 1 "15-19 years" 2 "20-24 years" 3 "25-34 years" 4 "35-44 years" 5 "45-54 years" 6 "55-64 years" 7 "65+ years"
label values age_group age_group_labels
label variable age_group "Age Group"

* Overall LFPR by year and age_group
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

* LFPR for each age_group by year, age_group, and sex
foreach age in `age_groups' {
    by year age_group sex, sort: egen adult_civilian`age' = total(((popstat == 1) & (age_group == `age')) * wtfinl)
    by year age_group sex: egen labor_force`age' = total(((labforce == 2) & (age_group == `age')) * wtfinl)
    gen lfpr`age' = (labor_force`age' / adult_civilian`age') * 100
}
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
    title("LFPR by Age Group, Men (1999 vs. 2024)") ///
    ytitle("Labor Force Participation Rate (%)") ///
    legend(label(1 "Ages 15-19") label(2 "Ages 20-24") label(3 "Ages 25-34") ///
           label(4 "Ages 35-44") label(5 "Ages 45-54") label(6 "Ages 55-64") ///
           label(7 "Ages 65+")) ///
    name(male, replace) ///
    nodraw
	   
graph bar lfpr* if sex == 2, over(year) ///
    title("LFPR by Age Group, Women (1999 vs. 2024)") ///
    ytitle("Labor Force Participation Rate (%)") ///
    legend(label(1 "Ages 15-19") label(2 "Ages 20-24") label(3 "Ages 25-34") ///
           label(4 "Ages 35-44") label(5 "Ages 45-54") label(6 "Ages 55-64") ///
           label(7 "Ages 65+")) ///
    name(female, replace) ///
    nodraw
	
graph bar overall_lfpr*, over(year) ///
    title("LFPR by Age Group, Overall (1999 vs. 2024)") ///
    ytitle("Labor Force Participation Rate (%)") ///
    legend(label(1 "Ages 15-19") label(2 "Ages 20-24") label(3 "Ages 25-34") ///
           label(4 "Ages 35-44") label(5 "Ages 45-54") label(6 "Ages 55-64") ///
           label(7 "Ages 65+")) ///
    name(overall, replace) ///
    nodraw
	
drop adult_civilian* labor_force*

******************************************************************************
* **3 Overall LFPR by Sex and Year**
******************************************************************************
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

******************************************************************************
* **4 2024 Aggregate LFPR with 1999 Age Distribution**
******************************************************************************
preserve
    collapse ///
        (sum) pop = adult_w /// Total adult population (weighted)
        (sum) lf  = in_lab_w, /// Total in labor force (weighted)
        by(year age_group)
    reshape wide pop lf, i(age_group) j(year)
    
    summarize pop1999
    local total_99 = r(sum)
    
gen share_1999 = pop1999 / `total_99'
gen lfpr_2024 = 100 * (lf2024 / pop2024)
gen w_lfpr_2024 = share_1999 * lfpr_2024
egen reweighted_2024 = total(w_lfpr_2024)
display "Reweighted 2024 LFPR (if age shares stayed at 1999 levels): " ///
         reweighted_2024
```

---

**Insights:**  
This script exemplifies advanced Stata programming by dynamically constructing age groups, applying weighted calculations, and generating comprehensive visualizations. The use of loops, conditional logic, and aggregate functions reflects a sophisticated approach to labor force analysis that is both efficient and scalable.

