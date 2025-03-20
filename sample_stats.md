Summary Statistics Example. This was done may 2024, since then I have gained skills much more advanced, but this is a baseline.

```stata

****************************************************************************************************************************

   cd "/Users/felipedominguez/Desktop/Research_Assistant/Hurricanes & music " 
   use "2024-05-23 - revised dataset2.dta" , clear

*****************************************************************************************************************************

**TABLE STATS**

   //mean and SD for experience//
   
      sum experience if treatment == 0 & experience != . //control//
      sum experience if treatment == 1 & experience != . //directive//
      sum experience if treatment == 2 & experience != . //Music//
      sum experience if experience !=. //all/
   
   //Proportions Gender //
   
      tab gender if treatment == 0 & experience != . //control//
      tab gender if treatment == 1 & experience != . //directive//
      tab gender if treatment == 2 & experience != . //Music//
      tab gender if experience !=. //all/
   
   //Proportions Country //
   
      tab country if treatment == 0 & experience != . //control//
      tab country if treatment == 1 & experience != . //directive//
      tab country if treatment == 2 & experience != . //Music//
      tab country if experience !=. //all/


**Proportions Behavioral outcomes**

   tab makeplan if treatment == 0 & experience != . //control//
   tab makeplan if treatment == 1 & experience != . //directive//
   tab makeplan if treatment == 2 & experience != . //Music//
   tab makeplan if experience != . //all/

   tab reminder if treatment == 0 & experience != . //control//
   tab reminder if treatment == 1 & experience != . //directive//
   tab reminder if treatment == 2 & experience != . //Music//
   tab reminder if experience != . //all/
   
   tab share if treatment == 0 & experience != . //control//
   tab share if treatment == 1 & experience != . //directive//
   tab share if treatment == 2 & experience != . //Music//
   tab share if experience !=. //all/
   
   
   
   tab intention if treatment == 0 & experience != . //control//
   tab intention if treatment == 1 & experience != . //directive//
   tab intention if treatment == 2 & experience != . //Music//
   tab intention if experience !=. //all/
   
   sum intention if treatment == 0 & experience != . //control//
   sum intention if treatment == 1 & experience != . //directive//
   sum intention if treatment == 2 & experience != . //Music//
   sum intention if experience !=. //all/
   
   
   tab subscribe if treatment == 0 & experience !=. //control//
   tab subscribe if treatment == 1 & experience !=. //directive//
   tab subscribe if treatment == 2 & experience !=. //Music//
   tab subscribe if experience !=. //all/


**Psychological outcomes**

   sum prisk if treatment == 0 & experience != . //control//
   sum prisk if treatment == 1 & experience != . //directive//
   sum prisk if treatment == 2 & experience != . //Music//
   sum prisk if experience != . //all/
   
   sum efficacy if treatment == 0 & experience != . //control//
   sum efficacy if treatment == 1 & experience != . //directive//
   sum efficacy if treatment == 2 & experience != . //Music//
   sum efficacy if experience !=. //all/
    
   sum trust if treatment == 0 & experience != . //control//
   sum trust if treatment == 1 & experience != . //directive//
   sum trust if treatment == 2 & experience != . //Music//
   sum trust if experience !=. //all/
   
   sum Happiness if treatment == 0 & experience != . //control//
   sum Happiness if treatment == 1 & experience != . //directive//
   sum Happiness if treatment == 2 & experience != . //Music//
   sum Happiness if experience !=. //all/

   sum Joy if treatment == 0 & experience !=. //control//
   sum Joy if treatment == 1 & experience != . //directive//
   sum Joy if treatment == 2 & experience != . //Music//
   sum Joy if experience !=. //all/
   
   
   sum Hope if treatment == 0 & experience != . //control//
   sum Hope if treatment == 1 & experience != . //directive//
   sum Hope if treatment == 2 & experience != . //Music//
   sum Hope if experience !=. //all/
   
   sum Safety if treatment == 0 & experience != . //control//
   sum Safety if treatment == 1 & experience != . //directive//
   sum Safety if treatment == 2 & experience != . //Music//
   sum Safety if experience !=. //all/
   
   sum Pride if treatment == 0 & experience != . //control//
   sum Pride if treatment == 1 & experience != . //directive//
   sum Pride if treatment == 2 & experience != . //Music//
   sum Pride if experience !=. //all/
   
   sum Worry if treatment == 0 & experience != . //control//
   sum Worry if treatment == 1 & experience != . //directive//
   sum Worry if treatment == 2 & experience != . //Music//
   sum Worry if experience !=. //all/
   
   sum Fear if treatment == 0 & experience !=. //control//
   sum Fear if treatment == 1 & experience !=. //directive//
   sum Fear if treatment == 2 & experience !=. //Music//
   sum Fear if experience !=. //all/
   
   sum Anxiety if treatment == 0 & experience !=. //control//
   sum Anxiety if treatment == 1 & experience !=. //directive//
   sum Anxiety if treatment == 2 & experience !=. //Music//
   sum Anxiety if experience !=. //all/



**Mediator Transport in other do file**

   sum transport if treatment == 0 //control//
   sum transport if treatment == 1 //directive//
   sum transport if treatment == 2 //Music//
   sum transport //all/

