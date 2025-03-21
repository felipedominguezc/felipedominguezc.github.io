
## Intermediate: Graphs Example

**Proficiency Level:** Intermediate

**Overview:**  
This .do file begins by setting the working directory and loading a revised dataset. It then renames key variables for clarity and produces a series of descriptive bar graphs. The script first generates mean bar graphs for psychological outcomes (Perceived Risk, Efficacy, Trust) by treatment group (Control, Directive, Music), followed by bar graphs that display the summed frequency of various emotion indicators. Finally, it converts behavioral outcome proportions to percentages and plots these outcomes as mean bar graphs by treatment, ultimately combining related graphs for cohesive visual presentations. This approach effectively communicates comparative study results through well-structured and visually appealing charts.

**Key Techniques & Features:**
- **Data Preparation:** Sets the working directory, loads data, and renames variables for clarity.
- **Descriptive Visualization:** Uses loops to create mean bar graphs for psychological outcomes by treatment group.
- **Emotion Frequency Analysis:** Generates bar graphs to display the summed frequency of various emotion indicators.
- **Behavioral Outcomes Conversion:** Converts proportions to percentages for intuitive visual comparisons.
- **Graph Combination:** Combines individual graphs into cohesive multi-panel displays.

---

**Stata Code:**

```stata
***********************************************************************************************************************************************
* FILE:           Hurricanes_Music_Graphs.do
* AUTHOR:         FDC
* CREATED:        2025-03-19
* LAST UPDATED:   2025-03-19
* PURPOSE:        Generate descriptive bar graphs for psychological and behavioral outcomes
*                 by experimental treatment in the Hurricane & Music study.
* INPUT:          2024-05-23 - revised dataset2.dta
* OUTPUT:         Combined graphs displayed in Stata; saved graph names for export
* DEPENDENCIES:   None (uses built-in Stata commands)
***********************************************************************************************************************************************

**************************************************************************************************
* SECTION 1: SETUP
**************************************************************************************************
    cd "/Users/felipedominguez/Desktop/Research_Assistant/Hurricanes & music "
    use "2024-05-23 - revised dataset2.dta", clear

**************************************************************************************************
* SECTION 2: VARIABLE RENAMING
**************************************************************************************************
    rename prisk PerceivedRisk
    rename efficacy Efficacy
    rename trust Trust

**************************************************************************************************
* SECTION 3: PSYCHOLOGICAL OUTCOMES GRAPHS
**************************************************************************************************
* Mean bar graphs of Perceived Risk, Efficacy, Trust by treatment condition
    foreach grp in 0 1 2 {
        graph bar (mean) PerceivedRisk Efficacy Trust if treatment==`grp' & experience!=., ///
            blabel(bar, format(%9.2f)) ylabel(#10, angle(horizontal)) ytitle("Mean") ///
            title("Psychological Outcomes - " + cond(`grp'==0,"Control", cond(`grp'==1,"Directive","Music")) + " Group") ///
            nolabel showyvars bargap(15) legend(off) scale(0.8) scheme(s1color*0.3) ///
            name(psgraph`grp'+1, replace)
    }
    graph combine psgraph1 psgraph2 psgraph3, name(ps_all, replace)

**************************************************************************************************
* SECTION 4: EMOTION FREQUENCY GRAPHS
**************************************************************************************************
* Bar graphs of summed emotion indicators by treatment condition
    foreach grp in 0 1 2 {
        graph bar (sum) Happiness Worry Joy Fear Pride Anxiety Hope Safety Other if treatment==`grp' & experience!=., ///
            blabel(bar, format(%9.0g)) ylabel(#10, angle(horizontal)) ytitle("Count") ///
            title("Frequency of Emotions - " + cond(`grp'==0,"Control", cond(`grp'==1,"Directive","Music")) + " Group") ///
            nolabel showyvars bargap(16) legend(off) scale(0.6) scheme(s1color*0.3) ///
            name(emotion`grp'+1, replace)
    }
    graph combine emotion1 emotion2 emotion3, name(emotions_all, replace)

**************************************************************************************************
* SECTION 5: BEHAVIORAL OUTCOMES GRAPHS
**************************************************************************************************
* Convert proportions to percentages
    foreach var in makeplan reminder share subscribe {
        gen `=upper(substr("`var'",1,1))'+substr("`var'",2,.) = `var'*100
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
```

---

**Insights:**  
- The script efficiently loads and prepares data by renaming key variables.
- It uses loops to generate descriptive bar graphs, facilitating comparative visualization across treatment groups.
- Emotion frequency and behavioral outcomes are effectively visualized, converting proportions to percentages for clarity.
- Combining graphs ensures that all visualizations are presented cohesively.
