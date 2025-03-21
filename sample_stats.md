
## Basic: Summary Statistics Example

**Proficiency Level:** Basic

**Overview:**  
This do file, created in May 2024, lays the groundwork for initial exploratory analysis of the Hurricanes & Music dataset. It calculates key descriptive measures for variables such as experience, gender, country, and various behavioral outcomes (e.g., makeplan, reminder, share, intention, subscribe) by treatment group (Control, Directive, Music). Additionally, it summarizes psychological outcomes (e.g., perceived risk, efficacy, trust, happiness, joy, hope, safety, pride, worry, fear, anxiety) and includes a section for summarizing a mediator variable (transport). Although more advanced techniques have been developed since then, this baseline script effectively showcases the fundamental data summarization skills used in early stages of research analysis.

**Key Techniques & Features:**
- **Descriptive Summaries:** Computes means, standard deviations, and tabulations for key variables.
- **Group Comparisons:** Breaks down summary statistics by treatment groups to highlight differences.
- **Mediator Analysis:** Includes additional summaries for a mediator variable.
- **Foundational Skills:** Demonstrates basic yet essential techniques in exploratory data analysis using Stata.

---

**Stata Code:**

```stata
****************************************************************************************************************************

   cd "/Users/felipedominguez/Desktop/Research_Assistant/Hurricanes & music " 
   use "2024-05-23 - revised dataset2.dta", clear

*****************************************************************************************************************************

**TABLE STATS**

*Mean and SD for experience*
   
   sum experience if treatment == 0 & experience != . // Control
   sum experience if treatment == 1 & experience != . // Directive
   sum experience if treatment == 2 & experience != . // Music
   sum experience if experience !=. // All
   
*Proportions Gender*
   
   tab gender if treatment == 0 & experience != . // Control
   tab gender if treatment == 1 & experience != . // Directive
   tab gender if treatment == 2 & experience != . // Music
   tab gender if experience !=. // All
   
*Proportions Country*
   
   tab country if treatment == 0 & experience != . // Control
   tab country if treatment == 1 & experience != . // Directive
   tab country if treatment == 2 & experience != . // Music
   tab country if experience !=. // All


*Proportions Behavioral outcomes*

   tab makeplan if treatment == 0 & experience != . // Control
   tab makeplan if treatment == 1 & experience != . // Directive
   tab makeplan if treatment == 2 & experience != . // Music
   tab makeplan if experience != . // All

   tab reminder if treatment == 0 & experience != . // Control
   tab reminder if treatment == 1 & experience != . // Directive
   tab reminder if treatment == 2 & experience != . // Music
   tab reminder if experience != . // All
   
   tab share if treatment == 0 & experience != . // Control
   tab share if treatment == 1 & experience != . // Directive
   tab share if treatment == 2 & experience != . // Music
   tab share if experience !=. // All
   
   tab intention if treatment == 0 & experience != . // Control
   tab intention if treatment == 1 & experience != . // Directive
   tab intention if treatment == 2 & experience != . // Music
   tab intention if experience !=. // All
   
   sum intention if treatment == 0 & experience != . // Control
   sum intention if treatment == 1 & experience != . // Directive
   sum intention if treatment == 2 & experience != . // Music
   sum intention if experience !=. // All
   
   
   tab subscribe if treatment == 0 & experience !=. // Control
   tab subscribe if treatment == 1 & experience !=. // Directive
   tab subscribe if treatment == 2 & experience !=. // Music
   tab subscribe if experience !=. // All


*Psychological outcomes*

   sum prisk if treatment == 0 & experience != . // Control
   sum prisk if treatment == 1 & experience != . // Directive
   sum prisk if treatment == 2 & experience != . // Music
   sum prisk if experience != . // All
   
   sum efficacy if treatment == 0 & experience != . // Control
   sum efficacy if treatment == 1 & experience != . // Directive
   sum efficacy if treatment == 2 & experience != . // Music
   sum efficacy if experience !=. // All
    
   sum trust if treatment == 0 & experience != . // Control
   sum trust if treatment == 1 & experience != . // Directive
   sum trust if treatment == 2 & experience != . // Music
   sum trust if experience !=. // All
   
   sum Happiness if treatment == 0 & experience != . // Control
   sum Happiness if treatment == 1 & experience != . // Directive
   sum Happiness if treatment == 2 & experience != . // Music
   sum Happiness if experience !=. // All

   sum Joy if treatment == 0 & experience !=. // Control
   sum Joy if treatment == 1 & experience != . // Directive
   sum Joy if treatment == 2 & experience != . // Music
   sum Joy if experience !=. // All
   
   sum Hope if treatment == 0 & experience != . // Control
   sum Hope if treatment == 1 & experience != . // Directive
   sum Hope if treatment == 2 & experience != . // Music
   sum Hope if experience !=. // All
   
   sum Safety if treatment == 0 & experience != . // Control
   sum Safety if treatment == 1 & experience != . // Directive
   sum Safety if treatment == 2 & experience != . // Music
   sum Safety if experience !=. // All
   
   sum Pride if treatment == 0 & experience != . // Control
   sum Pride if treatment == 1 & experience != . // Directive
   sum Pride if treatment == 2 & experience != . // Music
   sum Pride if experience !=. // All
   
   sum Worry if treatment == 0 & experience != . // Control
   sum Worry if treatment == 1 & experience != . // Directive
   sum Worry if treatment == 2 & experience != . // Music
   sum Worry if experience !=. // All
   
   sum Fear if treatment == 0 & experience !=. // Control
   sum Fear if treatment == 1 & experience !=. // Directive
   sum Fear if treatment == 2 & experience !=. // Music
   sum Fear if experience !=. // All
   
   sum Anxiety if treatment == 0 & experience !=. // Control
   sum Anxiety if treatment == 1 & experience !=. // Directive
   sum Anxiety if treatment == 2 & experience !=. // Music
   sum Anxiety if experience !=. // All


*Mediator Transport (summarized in another do file)*
   sum transport if treatment == 0 // Control
   sum transport if treatment == 1 // Directive
   sum transport if treatment == 2 // Music
   sum transport // All
```

---

**Insights:**  
- This script effectively generates foundational descriptive statistics to inform subsequent analyses.  
- It breaks down measures by treatment group (Control, Directive, Music) and across all observations, facilitating early insights into the data structure.  
- Although now superseded by more advanced techniques, this baseline script showcases essential data summarization skills that laid the foundation for later research improvements.
