Graphs Example

```stata

*******************************************************
* FILE:           Hurricanes_Music_Graphs.do
* AUTHOR:         FDC
* CREATED:        2025-03-19
* LAST UPDATED:   2025-03-19
* PURPOSE:        Generate descriptive bar graphs for psychological and behavioral outcomes
*                 by experimental treatment in the Hurricane & Music study.
* INPUT:          2024-05-23 - revised dataset2.dta
* OUTPUT:         Combined graphs displayed in Stata; saved graph names for export
* DEPENDENCIES:   None (uses built-in Stata commands)

*******************************************************

********** SECTION 1: SETUP **********

    cd "/Users/felipedominguez/Desktop/Research_Assistant/Hurricanes & music "
    use "2024-05-23 - revised dataset2.dta", clear

********** SECTION 2: VARIABLE RENAMING **********

    rename prisk PerceivedRisk
    rename efficacy Efficacy
    rename trust Trust

********** SECTION 3: PSYCHOLOGICAL OUTCOMES GRAPHS **********

* Mean bar graphs of Perceived Risk, Efficacy, Trust by treatment condition

    foreach grp in 0 1 2 {
        graph bar (mean) PerceivedRisk Efficacy Trust if treatment==`grp' & experience!=., ///
            blabel(bar, format(%9.2f)) ylabel(#10, angle(horizontal)) ytitle("Mean") ///
            title("Psychological Outcomes - " + cond(`grp'==0,"Control", cond(`grp'==1,"Directive","Music")) + " Group") ///
            nolabel showyvars bargap(15) legend(off) scale(0.8) scheme(s1color*0.3) ///
            name(psgraph`grp'+1, replace)
    }
    graph combine psgraph1 psgraph2 psgraph3, name(ps_all, replace)

********** SECTION 4: EMOTION FREQUENCY GRAPHS **********

* Bar graphs of summed emotion indicators by treatment condition

    foreach grp in 0 1 2 {
        graph bar (sum) Happiness Worry Joy Fear Pride Anxiety Hope Safety Other if treatment==`grp' & experience!=., ///
            blabel(bar, format(%9.0g)) ylabel(#10, angle(horizontal)) ytitle("Count") ///
            title("Frequency of Emotions - " + cond(`grp'==0,"Control", cond(`grp'==1,"Directive","Music")) + " Group") ///
            nolabel showyvars bargap(16) legend(off) scale(0.6) scheme(s1color*0.3) ///
            name(emotion`grp'+1, replace)
    }
    graph combine emotion1 emotion2 emotion3, name(emotions_all, replace)

********** SECTION 5: BEHAVIORAL OUTCOMES GRAPHS **********

* Convert proportions to percentages

    foreach var in makeplan reminder share subscribe {
        gen `=upper(substr("`var'",1,1))'+substr("`var'",2,. ) = `var'*100
    }

* Mean bar graphs of behavioral outcomes

    foreach grp in 0 1 2 {
        graph bar (mean) Makeplan Reminder Share Subscribe if treatment==`grp' & experience!=., ///
            blabel(bar, format(%9.2f)) ylabel(#10, angle(horizontal)) ytitle("Proportion") ///
            title("Behavioural Outcomes - " + cond(`grp'==0,"Control", cond(`grp'==1,"Directive","Music")) + " Group") ///
            nolabel showyvars bargap(15) legend(off) scale(0.8) scheme(s1color*0.3) ///
            name(graph`grp'+1, replace)
    }

    graph bar (mean) Makeplan Reminder Share Subscribe if experience!=., over(treatment) ///
        blabel(bar, format(%9.2f)) ylabel(#10, angle(horizontal)) ytitle("Proportion") ///
        title("Behavioural Outcomes by Experimental Condition") bargap(15) scale(0.8) scheme(s1color*0.3) name(graph_all, replace)
    graph combine graph1 graph2 graph3 graph_all, name(behaviors_all, replace)
