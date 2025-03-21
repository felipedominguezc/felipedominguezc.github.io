
## Basic Preliminary Data Analysis: Administrative Data for Disabled Students

**Proficiency Level:** Basic

**Overview:**  
This Stata script lays out the preliminary quantitative analysis for a project aimed at improving support for disabled students at the London School of Economics. It cleans and merges administrative records, tracks changes in students’ declared disability status, and computes time intervals between declarations. The script concludes with descriptive statistics and visualizations, offering insight into how students’ disability declarations evolve over time.

**Key Techniques:**
- **Data Import & Cleaning:** Reading data from Excel, removing duplicates, and standardizing categorical codes.  
- **Sequential Tracking:** Using Stata commands to identify first, last, and intermediate declarations, along with changes in disability status.  
- **Derived Indicators:** Generating new variables (e.g., `change_status`, `subsequent_change`) to measure how often and when declarations switch from “No” to “Yes” (or vice versa).  
- **Time Intervals & Summaries:** Calculating the number of days between declarations to gauge how quickly students modify their disability information.  
- **Visualization:** Creating bar charts to display initial, final, and intermediate declarations and changes among postgraduate students.

---

**Stata Code:**

```stata
****************************************************************************************************************************************
* Author: Felipe Dominguez Cornejo
* Preliminary quantitative analysis for improving support for disabled students at the LSE
****************************************************************************************************************************************

cd "/Users/felipedominguez/Desktop/Research_Assistant/Inclusion"
clear
import excel "Disability Report.xlsx", firstrow case(lower)

gen id = _n  // Temporary ID to maintain original order
order id

* Declared_date variable
rename startdate declared_date
label var declared_date "Declaration Date"
drop enddate

* Drop duplicates
duplicates drop studentid declared_date disabilitycode, force

* Generate a variable for disability declaration
gen declared_dis = .
replace declared_dis = 0 if disabilitycode == "A"
replace declared_dis = 1 if inlist(disabilitycode, "B", "C", "D", "E", "F", "G", "H", "I", "K")
replace declared_dis = 2 if disabilitycode == "98"
drop if disabilitycode == "99"

label define declared_dis 0 "No" 1 "Yes" 2 "Prefer not to say"
label values declared_dis declared_dis
label var declared_dis "Declared Disability"

* Sequence variable for each student
bysort studentid (declared_date id): gen studentid_number = _n
label variable studentid_number "Studentid Identifier"

* Last declaration indicator
egen max_studentid_number = max(studentid_number), by(studentid)
gen last_declaration = (studentid_number == max_studentid_number)
drop max_studentid_number
label variable last_declaration "Last Declaration"

* Declaration type variable
gen declaration_type = . 
replace declaration_type = 0 if studentid_number == last_declaration
replace declaration_type = 1 if studentid_number == 1 & last_declaration != 1
replace declaration_type = 2 if studentid_number != 1 & last_declaration == 1
replace declaration_type = 3 if studentid_number > 1 & last_declaration != 1

label define declaration_type 0 "Only" 1 "First" 2 "Last" 3 "Intermediate"
label values declaration_type declaration_type
label var declaration_type "Declaration Type"

bysort studentid (declared_date id): gen obs_count = _N
label variable obs_count "# Obs Per Student"

*********************************************************************************
* Change Status variable
*********************************************************************************
bysort studentid (declared_date id): gen first_declared_dis = declared_dis if declaration_type == 1
bysort studentid (declared_date id): gen last_declared_dis = declared_dis if declaration_type == 2

bysort studentid: replace first_declared_dis = first_declared_dis[1] if missing(first_declared_dis)
bysort studentid: replace last_declared_dis  = last_declared_dis[_N] if missing(last_declared_dis)

gen change_status = cond(first_declared_dis == last_declared_dis, 0, ///
    cond(first_declared_dis == 0 & last_declared_dis == 1, 1, ///
    cond(first_declared_dis == 1 & last_declared_dis == 0, 2, ///
    cond(first_declared_dis == 0 & last_declared_dis == 2, 3, ///
    cond(first_declared_dis == 2 & last_declared_dis == 0, 4, ///
    cond(first_declared_dis == 1 & last_declared_dis == 2, 5, ///
    cond(first_declared_dis == 2 & last_declared_dis == 1, 6, .)))))))

bysort studentid (declared_date id): replace change_status = change_status[1] if missing(change_status)
bysort studentid: replace change_status = 0 if obs_count == 1 & !missing(declared_dis)

label define change_status 0 "No change" ///
                         1 "No to yes" ///
                         2 "Yes to no" ///
                         3 "No to Prefer not to say" ///
                         4 "Prefer not to say to no" ///
                         5 "Yes to prefer not to say" ///
                         6 "Prefer not to say to yes"

label values change_status change_status
label var change_status "Change Status"

drop first_declared_dis last_declared_dis

*********************************************************************************
* Subsequent Change
*********************************************************************************
bysort studentid (declared_date id): gen lag_declareddis = declared_dis[_n-1]
gen subsequent_change = .

replace subsequent_change = ///
    cond(lag_declareddis == declared_dis, 0, ///
    cond(lag_declareddis == 0 & declared_dis == 1, 1, ///
    cond(lag_declareddis == 1 & declared_dis == 0, 2, ///
    cond(lag_declareddis == 0 & declared_dis == 2, 3, ///
    cond(lag_declareddis == 2 & declared_dis == 0, 4, ///
    cond(lag_declareddis == 1 & declared_dis == 2, 5, ///
    cond(lag_declareddis == 2 & declared_dis == 1, 6, .)))))))

bysort studentid: replace subsequent_change = . if _n == 1
replace subsequent_change = 0 if obs_count == 1

bysort studentid (declared_date id): replace subsequent_change = subsequent_change[_n-1] if missing(subsequent_change)

label define subsequent_change 0 "No change" ///
                              1 "No to yes" ///
                              2 "Yes to no" ///
                              3 "No to Prefer not to say" ///
                              4 "Prefer not to say to no" ///
                              5 "Yes to prefer not to say" ///
                              6 "Prefer not to say to yes"

label values subsequent_change subsequent_change
label var subsequent_change "Subsequent Changes"

drop lag_declareddis

bysort studentid (declared_date): gen change_number = sum(!missing(subsequent_change))
bysort studentid (declared_date): gen change_number2 = sum(subsequent_change != 0 & !missing(subsequent_change))
replace change_number2 = 0 if subsequent_change == 0

*********************************************************************************
* Time Differences
*********************************************************************************
bysort studentid (declared_date): gen initial_date = declared_date if studentid_number == 1
by studentid (declared_date): gen last_date = declared_date if last_declaration == 1
by studentid (declared_date): replace initial_date = initial_date[_n-1] if missing(initial_date)
by studentid (declared_date): replace last_date = last_date[_n-1] if missing(last_date)

gen time_difference = last_date - initial_date
replace time_difference = 0 if change_status == 0
by studentid: replace time_difference = time_difference[_N]
drop initial_date last_date
label variable time_difference "Total time difference (days)"

bysort studentid (declared_date): gen initial_date = declared_date if declaration_type == 1
by studentid (declared_date): gen firstchange_date = declared_date if change_number == 1
by studentid (declared_date): replace initial_date = initial_date[_n-1] if missing(initial_date)
by studentid (declared_date): replace firstchange_date = firstchange_date[_n-1] if missing(firstchange_date)

gen time_difference2 = firstchange_date - initial_date
replace time_difference2 = 0 if change_status == 0
by studentid: replace time_difference2 = time_difference2[_N]
drop initial_date firstchange_date
label variable time_difference2 "Time between Changes (days)"

* Restore original order
sort id
drop id

*********************************************************************************
* Descriptive Analysis & Graphs
*********************************************************************************
* Examples of tabulations and summarizations
tab declared_dis if programmetype == "Postgraduate" & studentid_number == 1
tab change_status if programmetype == "Postgraduate" & studentid_number == 1 & change_status != 0
tab declared_dis if last_declaration == 1 & programmetype == "Postgraduate"

* Graph examples
graph bar (count) if programmetype == "Postgraduate" & declaration_type <= 1, ///
    over(declared_dis) ///
    yti("Frequency") ///
    blabel(bar, format(%9.0f)) ///
    scale(0.8) ///
    ti("Frequency (Count) of Disability Declaration for Postgraduate Students") ///
    name(countgraph, replace)

graph bar (percent) if programmetype == "Postgraduate" & declaration_type <= 1, ///
    over(declared_dis) yti("Percentage") ///
    blabel(bar, format(%9.0f)) ///
    scale(0.8) ///
    ti("Frequency (Percent) of Disability Declaration for Postgraduate Students") ///
    name(percentgraph, replace)

* Additional graphs for changes, final declarations, etc.
graph bar (count) if programmetype == "Postgraduate" & studentid_number ==1 & change_status != 0, ///
    over(change_status) ///
    yti("Frequency") ///
    blabel(bar, format(%9.0f)) ///
    scale(0.5) ///
    ti("Frequency (Count) of Changes in Disability Declaration for Postgraduate Students") ///
    name(countchangegraph, replace)
```

---

**Insights:**  
- This file shows how to **track changes over time** in administrative data by creating variables for first, last, and intermediate declarations.  
- **Time-based variables** (e.g., `time_difference`, `time_difference2`) provide a detailed view of how quickly students change their status.  
- **Visualization** through bar graphs offers an immediate understanding of the frequency and nature of disability declarations.  
