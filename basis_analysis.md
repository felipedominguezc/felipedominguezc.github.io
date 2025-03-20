Administrative Data

```stata

*********************************************************************************************************************************************

* Author: Felipe Dominguez Cornejo
* This is part of the preliminary quantitative analysis I conducted for a project focused on improving support for disabled students at the LSE

***********************************************************************************************************************************************


	cd "/Users/felipedominguez/Desktop/Research_Assistant/Inclusion"
	clear
	import excel "Disability Report.xlsx", firstrow case(lower)
	
	gen id = _n  // Temporary ID to maintain original order
	order id

*Declared_date variable

	rename startdate declared_date
	label var declared_date "Declaration Date"
	drop enddate


*Drop duplicates based on all relevant columns

	duplicates drop studentid declared_date disabilitycode, force

*Variable for whether or not they declared a disability

	gen declared_dis = .
	replace declared_dis = 0 if disabilitycode == "A"
	replace declared_dis = 1 if inlist(disabilitycode, "B", "C", "D", "E", "F", "G", "H", "I", "K")
	replace declared_dis = 2 if disabilitycode == "98"
	drop if disabilitycode == "99"

	label define declared_dis 0 "No" 1 "Yes" 2 "Prefer not to say"
	label values declared_dis declared_dis
	label var declared_dis "Declared Disability"


*Sequence variable

	bysort studentid (declared_date id): gen studentid_number = _n
	// This variable numbers the entries made by each studentid based on the declared_date and number of disabilities they declared

	label variable studentid_number "Studentid Identifier"

*Last Declaration Variable

	egen max_studentid_number = max(studentid_number), by(studentid)
	gen last_declaration = (studentid_number == max_studentid_number)
	drop max_studentid_number
		
	label variable last_declaration "Last Declaration" //dummy//	
	
*Declaration type variable

	gen declaration_type = . 
	replace declaration_type = 0 if studentid_number == last_declaration
	replace declaration_type = 1 if studentid_number ==1 & last_declaration != 1
	replace declaration_type = 2 if studentid_number !=1 & last_declaration ==1
	replace declaration_type = 3 if studentid_number > 1 & last_declaration != 1

	label define declaration_type 0 "Only" 1 "First" 2 "Last" 3 "Intermediate"
	label values declaration_type declaration_type
	label var declaration_type "Declaration Type"

*Sort the data by studentid, declared_date, and id

	bysort studentid (declared_date id): gen obs_count = _N
	label variable obs_count "# Obs Per Student"

	
	
	
**#Change Status variable
	
	
	* Identify the first and last non-missing declared_dis for each student
		bysort studentid (declared_date id): gen first_declared_dis = declared_dis if declaration_type == 1
		bysort studentid (declared_date id): gen last_declared_dis = declared_dis if declaration_type == 2

	* Replace missing values of first_declared_dis and last_declared_dis
		bysort studentid: replace first_declared_dis = first_declared_dis[1] if missing(first_declared_dis)
		bysort studentid: replace last_declared_dis = last_declared_dis[_N] if missing(last_declared_dis)

	* Calculate change_status based on first and last declarations
		gen change_status = cond(first_declared_dis == last_declared_dis, 0, ///
			cond(first_declared_dis == 0 & last_declared_dis == 1, 1, ///
			cond(first_declared_dis == 1 & last_declared_dis == 0, 2, ///
			cond(first_declared_dis == 0 & last_declared_dis == 2, 3, ///
			cond(first_declared_dis == 2 & last_declared_dis == 0, 4, ///
			cond(first_declared_dis == 1 & last_declared_dis == 2, 5, ///
			cond(first_declared_dis == 2 & last_declared_dis == 1, 6, .)))))))

	* Propagate the change_status to all observations for each student
		bysort studentid (declared_date id): replace change_status = change_status[1] if missing(change_status)

	* Handle cases with only one observation
		bysort studentid: replace change_status = 0 if obs_count == 1 & !missing(declared_dis)

	* Label values and variable name
		label define change_status 0 "No change" ///
		1 "No to yes" 2 "Yes to no" ///
		3 "No to Prefer not to say" ///
		4 "Prefer not to say to no" ///
		5 "Yes to prefer not to say" ///
		6 "Prefer not to say to yes"

		label values change_status change_status
		label var change_status "Change Status"

	* Drop intermediate variables

		drop first_declared_dis last_declared_dis

	



*Subsequent Change 


	* Generate lagged variable for declared_dis (one lag is sufficient for tracking immediate changes)

		bysort studentid (declared_date id): gen lag_declareddis = declared_dis[_n-1]

	* Initialize subsequent_change variable with missing values

		gen subsequent_change = .

	* Identify changes between consecutive observations

		replace subsequent_change = ///
			cond(lag_declareddis == declared_dis, 0, ///
			cond(lag_declareddis == 0 & declared_dis == 1, 1, ///
			cond(lag_declareddis == 1 & declared_dis == 0, 2, ///
			cond(lag_declareddis == 0 & declared_dis == 2, 3, ///
			cond(lag_declareddis == 2 & declared_dis == 0, 4, ///
			cond(lag_declareddis == 1 & declared_dis == 2, 5, ///
			cond(lag_declareddis == 2 & declared_dis == 1, 6, .)))))))

	* For the first observation of each student, set subsequent_change to missing (no comparison available)

		bysort studentid: replace subsequent_change = . if _n == 1
	
	* Set no change for those with only one obs_count

		replace subsequent_change = 0 if obs_count ==1

	* Propagate the change status to all observations until the next change

		bysort studentid (declared_date id): replace subsequent_change = subsequent_change[_n-1] if missing(subsequent_change)

	* Label the subsequent_change variable

		label define subsequent_change 0 "No change" ///
		1 "No to yes" 2 "Yes to no" ///
		3 "No to Prefer not to say" ///
		4 "Prefer not to say to no" ///
		5 "Yes to prefer not to say" ///
		6 "Prefer not to say to yes"

		label values subsequent_change subsequent_change
		label var subsequent_change "Subsequent Changes"

	* Drop the lagged variable as it's no longer needed

		drop lag_declareddis

/* Change identifier variable: numbers chronologically the subsequent changes (includes "no change") */

	bysort studentid (declared_date): gen change_number = sum(!missing(subsequent_change))

/* Change identifier variable: numbers chronologically the subsequent changes (excludes "no change") */

	bysort studentid (declared_date): gen change_number2 = sum(subsequent_change != 0 & !missing(subsequent_change))
	replace change_number2 = 0 if subsequent_change == 0



	
/* Average time between changes Variable (first and last) */

	bysort studentid (declared_date): gen initial_date = declared_date if studentid_number == 1
	by studentid (declared_date): gen last_date = declared_date if last_declaration == 1
	by studentid (declared_date): replace initial_date = initial_date[_n-1] if missing(initial_date)
	by studentid (declared_date): replace last_date = last_date[_n-1] if missing(last_date)

	gen time_difference = last_date - initial_date
	replace time_difference = 0 if change_status ==0
	by studentid: replace time_difference = time_difference[_N]
	drop initial_date last_date

	label variable time_difference "Total time difference" //in days//

/* Average time between changes Variable (first declaration and first change) */

	bysort studentid (declared_date): gen initial_date = declared_date if declaration_type == 1
	by studentid (declared_date): gen firstchange_date = declared_date if change_number == 1
	by studentid (declared_date): replace initial_date = initial_date[_n-1] if missing(initial_date)
	by studentid (declared_date): replace firstchange_date = firstchange_date[_n-1] if missing(firstchange_date)

	gen time_difference2 = firstchange_date - initial_date
	replace time_difference2 = 0 if change_status ==0
	by studentid: replace time_difference2 = time_difference2[_N]
	drop initial_date firstchange_date

	label variable time_difference2 "Time between Changes" //in days//


	
	
/* Restore original order of data  */

	sort id
	drop id


*Analysis and distributions

	*Initial Distribution of declaration    
		tab declared_dis if programmetype == "Postgraduate" & studentid_number == 1
		tab declared_dis if programmetype == "Postgraduate" & declaration_type < 2 //equivalent to each other // 

	*Distribution of changes (first to last)

		tab change_status if programmetype == "Postgraduate" & studentid_number == 1
		tab change_status if programmetype == "Postgraduate" & studentid_number == 1 & change_status != 0 

	*Distribution of intermediate changes

		tab subsequent_change if programmetype == "Postgraduate" & change_number == 1
		tab subsequent_change if programmetype == "Postgraduate"  & subsequent_change != 0 

	* Final Distribution of declaration

		tab declared_dis if last_declaration == 1 & programmetype == "Postgraduate"

	* Distribution of Declaration type. Overlaps because of multiple entries for each student

		tab declaration_type if programmetype == "Postgraduate"

	*Time it takes for students to change their status on from first to last

		bysort change_status: sum time_difference  if programmetype == "Postgraduate" & studentid_number == 1
		sum time_difference  if programmetype == "Postgraduate" & studentid_number == 1 & change_status !=0

	*Time it takes for students to change their status

		bysort subsequent_change: sum time_difference2  if programmetype == "Postgraduate" & change_number2 == 1
		sum time_difference2  if programmetype == "Postgraduate" & change_number2 == 1
	
	*Number of changes for students

		bysort subsequent_change: sum change_number if programmetype == "Postgraduate" & subsequent_change != 0 & !missing(subsequent_change)
		sum change_number if programmetype == "Postgraduate" & subsequent_change != 0 & !missing(subsequent_change)
	
/* Graphs */

	* Initial Distribution of Declaration Bar Graph

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


	*Distribution of changes in declaration graph

		graph bar (count) if programmetype == "Postgraduate" & studentid_number ==1 & change_status != 0, ///
			over(change_status) ///
		    yti("Frequency") ///
		    blabel(bar, format(%9.0f)) ///
		    scale(0.5) ///
		    ti("Frequency (Count) of Changes in Disability Declaration for Postgraduate Students") ///
		    name(countchangegraph, replace)
		
		graph bar (percent) if programmetype == "Postgraduate" & studentid_number ==1  & change_status != 0, ///
			over(change_status) ///
		    yti("Frequency") ///
		    blabel(bar, format(%9.0f)) ///
		    scale(0.5) ///
		    ti("Frequency (Percent) of Changes in Disability Declaration for Postgraduate Students") ///
		    name(percentchangegraph, replace)

	*Distribution of final declaration graph
	
		graph bar (count) if programmetype == "Postgraduate" & last_declaration ==1, ///
			over(declared_dis) ///
		    yti("Frequency") ///
		    blabel(bar, format(%9.0f)) ///
		    scale(0.8) ///
		    ti("Frequency (Count) of Final Disability Declaration for Postgraduate Students") ///
		    name(countfinalgraph, replace)
		
		graph bar (percent) if programmetype == "Postgraduate" & last_declaration ==1, ///
			over(declared_dis) ///
		    yti("Frequency") ///
		    blabel(bar, format(%9.0f)) ///
		    scale(0.8) ///
		    ti("Frequency (Percent) of Final Disability Declaration for Postgraduate Students") ///
		    name(percentfinalgraph, replace) 

	*Intermediate changes graphs

		graph bar (count) if programmetype == "Postgraduate" & subsequent_change != 0, ///
			yti("Frequency") ///
			over(subsequent_change) ///
		    blabel(bar, format(%9.0f)) ///
		    scale(0.5) ///
		    ti("Frequency (Count) of Total Changes in Disability Declaration for Postgraduate Students") ///
		    name(intermediatecountgraph, replace) 
		
		graph bar (percent) if programmetype == "Postgraduate" & subsequent_change != 0, ///
			yti("Frequency") ///
			over(subsequent_change) ///
		    blabel(bar, format(%9.0f)) ///
		    scale(0.5) ///
		    ti("Frequency (Count) of Total Changes in Disability Declaration for Postgraduate Students") ///
		    name(intermediatepercentgraph, replace) 
		
	
	

