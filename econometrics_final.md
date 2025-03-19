Household Survey microdata

```stata

/*******************************************************************************************************

     * Project:   RARP Child Nutrition Analysis
     * Task:      Data Cleaning, Variable Construction, and Regression Modeling
     * Author:    Felipe Domínguez Cornejo
     * Date:      March 19, 2025
     * Data:      BDD_ENDI_R2_merged.dta — Ecuadorian household survey panel
     * Purpose:   Create key independent variables (mother’s marital status, household 
     *            demographics, socioeconomic controls), construct outcome (child 
     *            chronic malnutrition), and estimate baseline and interaction 
     *            regression models examining how mother’s marital status and 
     *            household resources relate to child nutrition outcomes.
     *
     * Workflow:
     *   1) Load and verify merged dataset integrity
     *   2) Generate primary exposure: mother_married & mother_responsible
     *   3) Create control variables (household composition, maternal characteristics, 
     *      child demographics, socioeconomic indices)
     *   4) Construct outcome: dcronica (chronic malnutrition)
     *   5) Produce summary statistics
     *   6) Run sequential OLS regressions with incremental covariate inclusion
     *      — Baseline, income, food insecurity, full controls
     *      — Interaction terms for marital status × income & food insecurity
     *   7) Export log file documenting all outputs
     *
     * Notes:
     *   • Missing and placeholder values (>=999999) recoded to missing.
     *   • All models use robust standard errors (`r` option).
     *   • Household IDs used for propagation of mother-level attributes.

 *******************************************************************************************************/

    set more 1  
    capture log close
    cd "/Users/felipedominguez/Desktop/Fall '24/Econometrics/RARP/BDD_ENDI_R2_dta"
    log using RARP, text replace

    use BDD_ENDI_R2_merged.dta, clear
    
    duplicates report id_hogar /*make sure merge worked */


**# Definition of independent variable

    gen married = . 
    replace married = 1 if f1_s1_18 == 1 | f1_s1_18 == 5 /*married or partnered*/
    replace married = 0 if f1_s1_18 == 2 | f1_s1_18 == 3 | f1_s1_18 == 4 | f1_s1_18 == 6 
    
    label variable married "Married/Partnered Status (1=Yes, 0=No)"
    label define married_label 0 "Not Married" 1 "Married/Partnered"
    label values married married_label
    
    
    destring(persona), replace
    
    
    generate mother_in_home = .
    replace mother_in_home = 1 if f1_s1_12 == 1 & grupo_edad_nin != . // Mother of child under 5 lives in the home
    replace mother_in_home = 0 if f1_s1_12 == 2 & grupo_edad_nin != . // Mother does not live in the home
    label variable mother_in_home "Mother lives in the home (1 = Yes, 0 = No)"


*Identify mother's position within the household

    gen mother_position = f1_s1_12_1 if f1_s1_12 == 1 & grupo_edad_nin != .

*Propagate mother's position to all rows in the same household

    bysort id_hogar (mother_position): replace mother_position = mother_position[_n] if !missing(mother_position)
    bysort id_hogar: replace mother_position = mother_position[_n-1] if missing(mother_position)
    
    gen mother_married = .
    replace mother_married = 1 if married ==1 & mother_position == persona
    replace mother_married = 0 if married == 0 & mother_position == persona
    
    label variable mother_married "Mother Married/Partnered Status (1=Yes, 0=No)"
    label define mother_married_label 0 "Mother Not Married" 1 "Mother Married/Partnered"
    label values mother_married mother_married_label


    tab married 
    tab mother_married

* Propagate mother's marital status

    bysort id_hogar (mother_married): replace mother_married = mother_married[_n] if !missing(mother_married)
    bysort id_hogar: replace mother_married = mother_married[_n-1] if missing(mother_married)




**# Controls

*Father lives in home dummy

    generate father_in_home = .
    replace father_in_home = 1 if f1_s1_11 == 1  & grupo_edad_nin != . // Father lives in the home
    replace father_in_home = 0 if f1_s1_11 == 2 & grupo_edad_nin != . // Father does not live in the home
    label variable father_in_home "Father lives in the home (1 = Yes, 0 = No)"

**#Mother specific controls

*Ethnicity

    gen mother_ethnicity = etnia if persona == mother_position
    
    tab mother_ethnicity, gen(mother_ethnicity)
    label variable mother_ethnicity1 "Ethnicity: Indígena"
    label variable mother_ethnicity2 "Ethnicity: Afroecuatoriana/o"
    label variable mother_ethnicity3 "Ethnicity: Montubia/o"
    label variable mother_ethnicity4 "Ethnicity: Mestiza/o"
    label variable mother_ethnicity5 "Ethnicity: Blanca/o u Otra"
    
* Propagate mother's ethnicity

    bysort id_hogar (mother_ethnicity): replace mother_ethnicity = mother_ethnicity[_n] if !missing(mother_ethnicity)
    bysort id_hogar: replace mother_ethnicity = mother_ethnicity[_n-1] if missing(mother_ethnicity)

* Propagate mother's ethnicity dummies

    foreach var of varlist mother_ethnicity1 mother_ethnicity2 mother_ethnicity3 ///
        mother_ethnicity4 mother_ethnicity5 {
        bysort id_hogar (`var'): replace `var' = `var'[_n] if !missing(`var')
        bysort id_hogar: replace `var' = `var'[_n-1] if missing(`var')
    }


*Disability

    gen mother_disability = .
    replace mother_disability = 1 if f1_s1_8 == 1 & persona == mother_position
    replace mother_disability = 0 if f1_s1_8 ==2 & persona == mother_position

* Propagate mother's disability status

    bysort id_hogar (mother_disability): replace mother_disability = mother_disability[_n] if !missing(mother_disability)
    bysort id_hogar: replace mother_disability = mother_disability[_n-1] if missing(mother_disability)

*Mother education

    gen mother_ed = f1_s1_15_1 if persona == mother_position
    tab mother_ed, gen(mother_ed)
    label variable mother_ed1 "Education: None"
    label variable mother_ed2 "Education:  Alfabetización (EBJA)"
    label variable mother_ed3 "Education: Primaria"
    label variable mother_ed4 "Education: Educacion General Basica"
    label variable mother_ed5 "Education: Secundaria"
    label variable mother_ed6 "Education: Bachillerato"
    label variable mother_ed7 "Education: Ciclo Postbachillerato (no superior)"
    label variable mother_ed8 "Education: Educación Técnica o Tecnológica Superior"
    label variable mother_ed9 "Education: Educacion Superior"
    label variable mother_ed10 "Education: Maestria/Especializacion"
    label variable mother_ed11 "Education: PHD/doctorado"

* Propagate mother's education level

    bysort id_hogar (mother_ed): replace mother_ed = mother_ed[_n] if !missing(mother_ed)
    bysort id_hogar: replace mother_ed = mother_ed[_n-1] if missing(mother_ed)

* Propagate mother's education dummies

    foreach var of varlist mother_ed1 mother_ed2 mother_ed3 mother_ed4 mother_ed5 ///
        mother_ed6 mother_ed7 mother_ed8 mother_ed9 mother_ed10 mother_ed11 {
        bysort id_hogar (`var'): replace `var' = `var'[_n] if !missing(`var')
        bysort id_hogar: replace `var' = `var'[_n-1] if missing(`var')
    }


*Mother worked

    gen mother_worked = inlist(f1_s2_1, 1, 2, 3, 4, 5, 6)  if persona == mother_position
        replace mother_worked = 0 if f1_s2_1 == 7
    label variable mother_worked "Mother worked last week (1=Yes, 0=No)"

* Propagate mother's work status

    bysort id_hogar (mother_worked): replace mother_worked = mother_worked[_n] if !missing(mother_worked)
    bysort id_hogar: replace mother_worked = mother_worked[_n-1] if missing(mother_worked)



*multiple jobs (think of interaction term)

    gen multiple_jobs = (f1_s2_7 == 2) if persona == mother_position
    label variable multiple_jobs "Mother had multiple jobs (1=Yes, 0=No)"

* Propagate mother's multiple jobs indicator

    bysort id_hogar (multiple_jobs): replace multiple_jobs = multiple_jobs[_n] if !missing(multiple_jobs)
    bysort id_hogar: replace multiple_jobs = multiple_jobs[_n-1] if missing(multiple_jobs)



*Mother household head or "spouse" to him

    gen mother_head_or_spouse = inlist(f1_s1_1, 1, 2) if persona == mother_position
    label variable mother_head_or_spouse "Mother is head or spouse of household head (1=Yes, 0=No)"
    //only moderately correlated with mother_married

*** Propagate mother is head or spouse of household head

    bysort id_hogar (mother_head_or_spouse): replace mother_head_or_spouse = mother_head_or_spouse[_n] if !missing(mother_head_or_spouse)
    bysort id_hogar: replace mother_head_or_spouse = mother_head_or_spouse[_n-1] if missing(mother_head_or_spouse)


*** Mother monetary income

    replace f1_s2_12 = . if f1_s2_12 >= 999999 //replace place holder for missing values 
    replace f1_s2_12 = 0 if mother_worked == 0 | f1_s2_4 == 1 | f1_s2_1 == 7  //if didn't work, no slary

    gen  mother_monetary_income =  f1_s2_12 if mother_position ==persona
    replace mother_monetary_income = 0 if mother_worked ==0 & mother_position ==persona
    label variable mother_monetary_income "Mother's monetary income"

*** Propagate mother's monetary income

    bysort id_hogar (mother_monetary_income): replace mother_monetary_income = mother_monetary_income[_n] if !missing(mother_monetary_income)
    bysort id_hogar: replace mother_monetary_income = mother_monetary_income[_n-1] if missing(mother_monetary_income)


*** In-Kind Income Test on regression (low number of obs)

    replace f1_s2_14_2 = 0 if f1_s2_4 == 1 | f1_s2_1 == 7 //if didn't work, no slary
    replace f1_s2_14_2 = 0 if f1_s2_14_1 == 2 //if didn't work, no slary
    replace f1_s2_14_2 = . if f1_s2_14_2 >= 999999 //replace place holder for missing values 
    gen mother_inkind_income = f1_s2_14_2 if persona == mother_position
    replace mother_inkind_income = 0 if mother_worked ==0 & mother_position ==persona
    label variable mother_inkind_income "Mother's in-kind income"

*** Propagate mother's in-kind income
    bysort id_hogar (mother_inkind_income): replace mother_inkind_income = mother_inkind_income[_n] if !missing(mother_inkind_income)
    bysort id_hogar: replace mother_inkind_income = mother_inkind_income[_n-1] if missing(mother_inkind_income)



*** Social Transfers income

    replace f1_s2_22 = 0 if f1_s2_21 ==2
    replace f1_s2_24 = 0 if f1_s2_23 ==2
    replace f1_s2_26 = 0 if f1_s2_25 ==2

    gen mother_social_transfers = f1_s2_22 + f1_s2_24 + f1_s2_26 if persona == mother_position
    label variable mother_social_transfers "Mother's social transfers income"

*** Propagate mother's social transfers
    bysort id_hogar (mother_social_transfers): replace mother_social_transfers = mother_social_transfers[_n] if !missing(mother_social_transfers)
    bysort id_hogar: replace mother_social_transfers = mother_social_transfers[_n-1] if missing(mother_social_transfers)

    replace f1_s2_20_2 = 0 if f1_s2_20_1 ==2
    replace f1_s2_20_2 = . if f1_s2_20_2 >= 999999
    sum f1_s2_20_2

*mother responsible for the child

    gen mother_responsible = .
    replace mother_responsible = 0 if f1_s6_1 != mother_position & grupo_edad_nin != .
    replace mother_responsible = 1 if f1_s6_1 == mother_position & grupo_edad_nin != .
    label variable mother_responsible "Mother is primary caregiver"
    label define caregiver_label 0 "Not Mother" 1 "Mother Primary Caregiver"
    label values mother_responsible caregiver_label

*** Propagate mother's responsibility status
    bysort id_hogar (mother_responsible): replace mother_responsible = mother_responsible[_n] if !missing(mother_responsible)
    bysort id_hogar: replace mother_responsible = mother_responsible[_n-1] if missing(mother_responsible)


*mother age
    gen mother_age = f1_s1_3_1 if persona == mother_position

*** Propagate mother's age
    bysort id_hogar (mother_age): replace mother_age = mother_age[_n] if !missing(mother_age)
    bysort id_hogar: replace mother_age = mother_age[_n-1] if missing(mother_age)



**#Child specific controls

    *Sex of child at birth

        gen male = .
        replace male = 1 if f1_s1_2 == 1 & grupo_edad_nin != .
        replace male = 0 if f1_s1_2 == 2 & grupo_edad_nin != .
        label variable male "Child is male (1=Yes, 0=No)"
        
        gen age_group_child = grupo_edad_nin if grupo_edad_nin != .
        
        gen anemia = ane6_59 if grupo_edad_nin != .

*** Create total household income by summing income for all members in the household
    replace f1_s2_12 = . if f1_s2_12 >=999999
    bysort id_hogar: egen household_income = total(f1_s2_12)
    label variable household_income "Total household income"
    


*** Area

    gen rural = .
    replace rural =1 if area ==2
    replace rural =0 if area ==1
    

*** Roof condition: Good vs Not Good

    gen roof_good = (f1_s3_4 == 1)
    label variable roof_good "Roof in good condition"

*** Walls condition: Good vs Not Good

    gen walls_good = (f1_s3_6 == 1)
    label variable walls_good "Walls in good condition"

*** Floor condition: Good vs Not Good

    gen floor_good = (f1_s3_8 == 1)
    label variable floor_good "Floor in good condition"

*** Summary: Housing quality index

    gen housing_quality_index = roof_good + walls_good + floor_good
    label variable housing_quality_index "Housing quality index (0-3)"


    gen piped_water = .
    replace piped_water = 1 if f1_s3_9 == 1 | f1_s3_9 == 2 | f1_s3_9 == 3
    replace piped_water = 0 if f1_s3_9 ==0
    label variable piped_water "Household has piped water"
    
    gen safe_water = .
    replace safe_water = 1 if f1_s3_10 == 1 | f1_s3_10 == 2 //public system
    replace safe_water = 0 if f1_s3_10 == 3 | f1_s3_10 == 4 | f1_s3_10 == 5
    label variable safe_water "Safe water source"
    
    gen improved_sanitation = .
    replace improved_sanitation = 1 if f1_s3_11 == 1 | f1_s3_11 == 2 | f1_s3_11 == 3
    replace improved_sanitation = 0 if f1_s3_11 == 4 | f1_s3_11 == 5 | f1_s3_11 == 6 | f1_s3_11 ==7
    label variable improved_sanitation "Improved sanitation access"
    
    gen water_sanitation_index = piped_water + safe_water + improved_sanitation
    label variable water_sanitation_index "Water and Sanitation Index (0-3)"

***Room for cooking

    gen separate_kitchen = (f1_s3_17 == 1)
    label variable separate_kitchen "Household has separate kitchen space"

*** Count total people per household (id_hogar)

     bysort id_hogar: gen total_members = _N

*** Generate household density (people per room)

    gen household_density = total_members / f1_s3_15


*** Recode food insecurity variables into binary

    recode f1_s4_1_1 (1 = 1) (2/8 = 0), gen(worried_food)
    recode f1_s4_1_2 (1 = 1) (2/8 = 0), gen(lacked_nutrition)
    recode f1_s4_1_4 (1 = 1) (2/8 = 0), gen(skipped_meals)
    recode f1_s4_1_7 (1 = 1) (2/8 = 0), gen(felt_hungry)
    recode f1_s4_1_8 (1 = 1) (2/8 = 0), gen(no_food_all_day)

*** Create food insecurity index

    gen food_insecurity_index = worried_food + lacked_nutrition + skipped_meals + felt_hungry + no_food_all_day
    label variable food_insecurity_index "Food insecurity index (0-5)"

*** Recode chlorine variable

    recode cloro_v (3/4 = 0 "Safe") (1/2 = 1 "Unsafe"), gen(water_chlorine_safe)
    label variable water_chlorine_safe "Presence of chlorine in water"

*** Recode E. coli variable

    recode ecoli_v (3/4 = 0 "Safe") (1/2 = 1 "Unsafe"), gen(water_ecoli_unsafe)
    label variable water_ecoli_unsafe "E. coli presence in water"

*** Combine water safety indicators

    gen water_overall_safe = (water_chlorine_safe == 1 & water_ecoli_unsafe == 0)
    label variable water_overall_safe "Overall safe water (chlorine present & no E. coli)"
    


    keep if grupo_edad_nin != .

**# Summary stats

    sum dcronica i.age_group_child male mother_married i.mother_ed mother_ethnicity* mother_age  mother_monetary_income father_in_home rural nbi_1 i.quintil food_insecurity_index water_sanitation_index 


*** Graphs ***

    graph bar (mean) dcronica, over(mother_married) ///
        bar(1, color(green)) title("Chronic Malnutrition by Mother's Marital Status") ///
        ytitle("Proportion with Chronic Malnutrition") blabel(bar) 
    
    drop mother_ethnicity
    drop mother_ethnicity4 //relative to mestiza
    drop mother_ed
    drop mother_ed1 //relative to none

**# MODELS

    gen log_household_income = ln(household_income+1)
    gen log_mother_monetary_income = ln(mother_monetary_income+1)
    
    reg dcronica mother_married, r

*** Create vector controls

    global child_controls i.age_group_child male i.anemia
    
    global mother_controls mother_ethnicity* mother_ed* mother_age mother_disability
    
    global geo_socio_economic_controls rural i.region i.prov altitud i.quintil i.nbi_1
    
    global household_controls housing_quality_index water_sanitation_index separate_kitchen household_density water_overall_safe

*** 1. Baseline

    reg dcronica mother_married mother_responsible  $child_controls , r

*** 2. Include household_income

    reg dcronica mother_married mother_responsible log_household_income  $child_controls  , r

*** 3. Include food insecurity index

    reg dcronica mother_married mother_responsible log_household_income food_insecurity_index $child_controls   , r
    test log_household_income food_insecurity_index

*** 4. Include all controls

    reg dcronica mother_married mother_responsible log_household_income food_insecurity_index $child_controls $mother_controls $household_controls $geo_socio_economic_controls , r

*** 5. base interaction with household income

    gen mother_married_household_income = mother_married*log_household_income
        reg dcronica mother_married log_household_income mother_married_household_income mother_responsible  $child_controls , r
        test mother_married_household_income

*** 6. interaction with household income and controls

    reg dcronica mother_married log_household_income mother_married_household_income ///
            mother_responsible  $child_controls $mother_controls $household_controls ///
            $geo_socio_economic_controls , r
    test mother_married_household_income


*** 7. base interaction with food index

    gen mother_married_food_index = mother_married*food_insecurity_index
    reg dcronica mother_married food_insecurity_index mother_married_food_index mother_responsible  $child_controls , r
    test mother_married_food_index

*** 8. interaction with food index
    reg dcronica mother_married mother_responsible log_household_income ///
        food_insecurity_index mother_married_food_index mother_married_household_income  ////
        $child_controls $mother_controls $household_controls $geo_socio_economic_controls , r
    test mother_married_household_income mother_married_food_index

    log close
