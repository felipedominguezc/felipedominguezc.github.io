
## Intermediate: Graphs Example

**Proficiency Level:** Intermediate

**Overview:**  
This .do file begins by setting the working directory and loading a revised dataset. It then renames key variables for clarity and produces a series of descriptive bar graphs. The script first generates mean bar graphs for psychological outcomes (Perceived Risk, Efficacy, Trust) by treatment group (Control, Directive, Music, Pooled), followed by bar graphs that display the summed frequency of various emotion indicators. Finally, it converts behavioral outcome proportions to percentages and plots these outcomes as mean bar graphs by treatment, ultimately combining related graphs for cohesive visual presentations. This approach effectively communicates comparative study results through well-structured and visually appealing charts.

**Key Techniques & Features:**
- **Data Preparation:** Sets the working directory, loads data, and renames variables for clarity.
- **Emotion Frequency Analysis:** Generates bar graphs to display the summed frequency of various emotion indicators.
- **Behavioral Outcomes Conversion:** Converts proportions to percentages for intuitive visual comparisons.
- **Graph Combination:** Combines individual graphs into cohesive multi-panel displays.

---

**Stata Code:**

```stata
set more 1
capture log close 

log using "/Users/felipedominguez/Desktop/Abroad docs/Research_Assistant/Hurricanes & music/bar_graphs.txt", replace

**********************************************************************************
	* Why did I write this code: Bar graphs for publication on the Hurricanes and Music Project with Dr. Shreedhar
	*Data using: Revised dataset with 974 observations when filtering by not missing experience var
	*Output: Graphs into my folder as .png files
	*Technical Considerations: using color-blind friendly scheme from schemepack package
	*Issues: Psychological outcomes have n<974
**********************************************************************************

*** Installing packages***
ssc install schemepack, replace 

*** Creating Directories ***
global desktop "/Users/felipedominguez/Desktop"
global mainpath "$desktop/Abroad docs/Research_Assistant/Hurricanes & music"
global output "$mainpath/Bar graphs/Final graphs"

*** Setting graph settings***
graph set window fontface "Times New Roman"
global scheme "cblind1" // color-blind friendly scheme
global titlesize "size(medium)" // titles will have size medium
global subsize "size(medsmall)" //axis titles size medsmall
global labsize "labsize(small)" //axis labels size small
global cond "if experience != ." //focus on correct sample size


use "$mainpath/2024-05-23 - revised dataset2.dta" , clear
**********************************************************************************
*Psychological Outcomes Bar Graphs
**********************************************************************************

* PerceivedRisk
	preserve
	collapse (mean) prisk $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) prisk $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'


	gen id = _n
	reshape wide prisk, i(id) j(group) string
	rename  (priskAll priskControl priskDirective priskMusic) (All Control Directive Music)
	drop treatment id

	set graphic off
		graph bar Control Directive Music All, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(1(1)7, angle(horizontal) $labsize) /// 
			yscale(range(1 7)) ///
			ytitle("Mean", $subsize) ///
			title("Perceived Risk", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme) ///
			name(outcome1, replace)
	restore

*Efficacy

	preserve
	collapse (mean) efficacy $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) efficacy $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore
	preserve
	use `treat', clear
	append using `pooled'


	gen id = _n
	reshape wide efficacy, i(id) j(group) string
	rename  (efficacyAll efficacyControl efficacyDirective efficacyMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(1(1)7, angle(horizontal) $labsize) /// 
			yscale(range(1 7)) ///
			ytitle("Mean", $subsize) ///
			title("Efficacy", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme) ///
			name(outcome2, replace)
	restore

*Trust

	preserve
	collapse (mean) trust $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) trust $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore
	preserve
	use `treat', clear
	append using `pooled'


	gen id = _n
	reshape wide trust, i(id) j(group) string
	rename  (trustAll trustControl trustDirective trustMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(1(1)7, angle(horizontal) $labsize) /// 
			yscale(range(1 7)) ///
			ytitle("Mean", $subsize) ///
			title("Trust", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme) ///
			name(outcome3, replace)
			
*All psychological outomes
	graph combine outcome1 outcome2 outcome3, ///
		scheme($scheme ) hol()
	graph export "$output/psychological_outcomes_one.png", replace
restore


**********************************************************************************
*Emotion list Bar graphs by Treatment and Pooled Sample
**********************************************************************************

*Happiness
	preserve
	collapse (sum) Happiness $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Happiness $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Happiness, i(id) j(group) string
	rename  (HappinessAll HappinessControl HappinessDirective HappinessMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Happiness", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion1, replace)
	restore

*Worry
	preserve
	collapse (sum) Worry $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Worry $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Worry, i(id) j(group) string
	rename  (WorryAll WorryControl WorryDirective WorryMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Worry", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion2, replace)
	restore

*Joy
	preserve
	collapse (sum) Joy $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Joy $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Joy, i(id) j(group) string
	rename  (JoyAll JoyControl JoyDirective JoyMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Joy", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion3, replace)
	restore

*Fear
	preserve
	collapse (sum) Fear $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Fear $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Fear, i(id) j(group) string
	rename  (FearAll FearControl FearDirective FearMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Fear", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion4, replace)
	restore

*Pride
	preserve
	collapse (sum) Pride $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Pride $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Pride, i(id) j(group) string
	rename  (PrideAll PrideControl PrideDirective PrideMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Pride", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion5, replace)
	restore

*Anxiety
	preserve
	collapse (sum) Anxiety $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Anxiety $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Anxiety, i(id) j(group) string
	rename  (AnxietyAll AnxietyControl AnxietyDirective AnxietyMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Anxiety", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion6, replace)
	restore

*Hope
	preserve
	collapse (sum) Hope $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Hope $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Hope, i(id) j(group) string
	rename  (HopeAll HopeControl HopeDirective HopeMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Hope", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion7, replace)
	restore

*Safety
	preserve
	collapse (sum) Safety $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Safety $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Safety, i(id) j(group) string
	rename  (SafetyAll SafetyControl SafetyDirective SafetyMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Safety", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion8, replace)
	restore

*Other
	preserve
	collapse (sum) Other $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (sum) Other $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Other, i(id) j(group) string
	rename  (OtherAll OtherControl OtherDirective OtherMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.0g)) ///   
			ylabel(#10, angle(horizontal) $labsize ) ///
			ytitle("Count", $subsize ) ///
			title("Other", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(emotion9, replace)

*All emotions
set graphics off
	graph combine emotion1 emotion2 emotion3 emotion4 emotion5 emotion6 emotion7 emotion8 emotion9, ///
		scheme($scheme ) 
	graph export "$output/emotions_one.png", replace
	
restore


**********************************************************************************
*Behavioral Outcomes Bar Graphs
**********************************************************************************


gen Makeplan = makeplan *100
gen Reminder = reminder *100
gen Share = share*100
gen Subscribe = subscribe *100

*Makeplan
	preserve
	collapse (mean) Makeplan $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) Makeplan $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Makeplan, i(id) j(group) string
	rename  (MakeplanAll MakeplanControl MakeplanDirective MakeplanMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(#10, angle(horizontal) $labsize format(%9.2f)) ///
					yscale(range(1 100)) ///
			ytitle("Proportion", $subsize ) ///
			title("Make a Plan", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(outcome1, replace)
	restore

*Reminder
	preserve
	collapse (mean) Reminder $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) Reminder $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Reminder, i(id) j(group) string
	rename  (ReminderAll ReminderControl ReminderDirective ReminderMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(#10, angle(horizontal) $labsize format(%9.2f)) ///
					yscale(range(1 100)) ///
			ytitle("Proportion", $subsize ) ///
			title("Reminder", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(outcome2, replace)
	restore

*Share
	preserve
	collapse (mean) Share $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) Share $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Share, i(id) j(group) string
	rename  (ShareAll ShareControl ShareDirective ShareMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(#10, angle(horizontal) $labsize format(%9.2f)) ///
			yscale(range(1 100)) ///
			ytitle("Proportion", $subsize ) ///
			title("Share", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(outcome3, replace)
			
	restore

*Subscribe
	preserve
	collapse (mean) Subscribe $cond, by(treatment)
	gen group = cond(treatment==0, "Control", ///
			 cond(treatment==1, "Directive", ///
			 cond(treatment==2, "Music", "")))
	tempfile treat
	save `treat', replace
	restore

	preserve
	collapse (mean) Subscribe $cond
	gen group = "All"
	tempfile pooled
	save `pooled', replace
	restore

	preserve
	use `treat', clear
	append using `pooled'

	gen id = _n
	reshape wide Subscribe, i(id) j(group) string
	rename  (SubscribeAll SubscribeControl SubscribeDirective SubscribeMusic) (All Control Directive Music)
	drop treatment id

	set graphics off
		graph bar All Control Directive Music, ///
			blabel(bar, format(%9.2f)) ///   
			ylabel(#10, angle(horizontal) $labsize format(%9.2f)) ///
			yscale(range(1 100)) ///
			ytitle("Proportion", $subsize ) ///
			title("Subscribe", $titlesize ) ///
			bargap(15) ///
			showyvars ///
			legend(off) ///
			scale(0.8) ///
			nolabel ///
			scheme($scheme ) ///
			name(outcome4, replace)
			

*All outcomes
set graphics off
	graph combine outcome1 outcome2 outcome3 outcome4, ///
		scheme($scheme )
	graph export "$output/behavioral_one.png", replace
	
restore 

log close 
```

---

**Insights:**  
- The script efficiently loads and prepares data by renaming key variables.
- Emotion frequency and behavioral outcomes are effectively visualized, converting proportions to percentages for clarity.
- Combining graphs ensures that all visualizations are presented cohesively.
