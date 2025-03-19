# Data Cleaning Example


```stata


*******************************************************
* FILE:           Hurricanes_Music_Study.do
* AUTHOR:         FDC
* CREATED:        2025-03-19
* PURPOSE:        Clean, recode, and analyze MH_raw data for hurricane/music emotion study.
* INPUT:          MH_raw.dta
* OUTPUT:         Descriptive statistics, chi-square tests, and variables saved in memory
* DEPENDENCIES:   None (uses built-in Stata commands)

*******************************************************

********** SECTION 1: SETUP **********

cd "/Users/felipedominguez/Desktop/Abroad docs/Research_Assistant/Hurricanes & music "
use MH_raw.dta, clear


********** SECTION 2: DESCRIBE & SPLIT STRING VARIABLES **********

/* Split and destring emotionlist, planhome, itemsshelter, itemskit into numeric indicators */
split emotionlist, generate(emotionlist) parse(,) destring
split planhome, generate(p_home) parse(,) destring 
split itemsshelter, generate(ishelter) parse(,) destring 
split itemskit, generate(ikit) parse(,) destring 

********** SECTION 3: CREATE DUMMY VARIABLES FOR CATEGORICAL RESPONSES **********

/* Generate binary indicators for emotion categories */
gen emolist1 =0 
replace emolist1 = 1 if emotionlist1 == 1   /*dummy variable*/
replace emolist1 = 0 if emotionlist == ""   /*for missing values*/

gen emolist2 =0 
replace emolist2 = 1 if emotionlist1 == 2 | emotionlist2 == 2 
replace emolist2 = . if emotionlist == ""  


gen emolist3 =0 
replace emolist3 = 1 if emotionlist1 == 3 |  emotionlist2 == 3 | emotionlist3 == 3 
replace emolist3 = . if emotionlist == ""  


gen emolist4 =0 
replace emolist4 = 1 if emotionlist1 == 4 | emotionlist2 == 4 | emotionlist3 == 4 | emotionlist4 == 4 
replace emolist4 = . if emotionlist == ""  


gen emolist5 =0 
replace emolist5 = 1 if emotionlist1 == 5 | emotionlist2 == 5 | emotionlist3 == 5 |emotionlist4 == 5| emotionlist5 == 5 
replace emolist5 = . if emotionlist == ""  

gen emolist6 =0 
replace emolist6 = 1 if emotionlist1 == 6 | emotionlist2 == 6 | emotionlist3 == 6 | emotionlist4 == 6 | emotionlist5 == 6 | emotionlist6 == 6 
replace emolist6 = . if emotionlist == ""  

gen emolist7 =0 
replace emolist7 = 1 if emotionlist1 == 7 | emotionlist2 == 7 | emotionlist3 == 7 |emotionlist4 == 7 | emotionlist5 == 7 | emotionlist6 == 7 | emotionlist7 == 7 
replace emolist7 = . if emotionlist == ""  

gen emolist8 =0 
replace emolist8 = 1 if emotionlist1 == 8 | emotionlist2 == 8 | emotionlist3 == 8 |emotionlist4 == 8 | emotionlist5 == 8 | emotionlist6 == 8 | emotionlist7 == 8 |emotionlist8==8 
replace emolist8 = . if emotionlist == ""  

gen emolist9 =0 
replace emolist9 = 1 if emotionlist1 == 9 | emotionlist2 == 9 | emotionlist3 == 9 |emotionlist4 == 9 | emotionlist5 == 9 | emotionlist6 == 9 | emotionlist7 == 9 |emotionlist8==9 
replace emolist9 = . if emotionlist == ""  


drop emotionlist1 emotionlist2 emotionlist3 emotionlist4 emotionlist5 emotionlist6 emotionlist7 emotionlist8 emotionlist9 

/*new categorical values for planhome*/
gen planhome1 = 0 
replace planhome1 = 1 if p_home1 ==1 
replace planhome1 = . if planhome =="" 
 
gen planhome2 = 0 
replace planhome2 = 1 if p_home1 ==2 | p_home2==2 
replace planhome2 = . if planhome =="" 
 
gen planhome3 = 0 
replace planhome3 = 1 if p_home1 ==3 | p_home2==3 | p_home3 ==3 
replace planhome3 = . if planhome =="" 

gen planhome4 = 0 
replace planhome4 = 1 if p_home1 ==4 | p_home2==4 | p_home3 ==4 |p_home4==4 
replace planhome4 = . if planhome =="" 

drop p_home* 

/*new categorical values for itemsshelter*/
gen itemshelter1 = 0 
replace itemshelter1 = 1 if ishelter1 ==1 
replace itemshelter1 = . if itemsshelter =="" 

gen itemshelter2 = 0 
replace itemshelter2 = 1 if ishelter1 ==2 | ishelter2 ==2 
replace itemshelter2 = . if itemsshelter =="" 

gen itemshelter3 = 0 
replace itemshelter3 = 1 if ishelter1 ==3 | ishelter2 ==3 | ishelter3==3  
replace itemshelter3 = . if itemsshelter =="" 

gen itemshelter4 = 0 
replace itemshelter4 = 1 if ishelter1 ==4 | ishelter2 ==4 | ishelter3==4 |ishelter4 ==4 
replace itemshelter4 = . if itemsshelter =="" 

gen itemshelter5 = 0 
replace itemshelter5 = 1 if ishelter1 ==5 | ishelter2 ==5 | ishelter3==5 |ishelter4 ==5 
replace itemshelter5 = . if itemsshelter =="" 

drop ishelter* 

/*new categorical values for itemskit*/

gen itemskit1 = 0 
replace itemskit1 = 1 if ikit1 ==1 | ikit2 ==1 | ikit3==1 |ikit4 ==1 |ikit5 ==1 |ikit6==1 
replace itemskit1 = . if itemskit =="" 

gen itemskit2 = 0 
replace itemskit2 = 1 if ikit1 ==2 | ikit2 ==2 | ikit3==2 |ikit4 ==2 |ikit5 ==2 |ikit6==2 
replace itemskit2 = . if itemskit =="" 

gen itemskit3 = 0 
replace itemskit3 = 1 if ikit1 ==3 | ikit2 ==3 | ikit3==3 |ikit4 ==3  |ikit5 ==3 |ikit6==3 
replace itemskit3 = . if itemskit =="" 

gen itemskit4 = 0 
replace itemskit4 = 1 if ikit1 ==4 | ikit2 ==4 | ikit3==4 |ikit4 ==4  |ikit5 ==4 |ikit6==4 
replace itemskit4 = . if itemskit =="" 

gen itemskit5 = 0 
replace itemskit5 = 1 if ikit1 ==5 | ikit2 ==5 | ikit3==5 |ikit4 ==5  |ikit5 ==5 |ikit6==5 
replace itemskit5 = . if itemskit =="" 

gen itemskit6 = 0 
replace itemskit6 = 1 if ikit1 ==6 | ikit2 ==6 | ikit3==6 |ikit4 ==6  |ikit5 ==6 |ikit6==6 
replace itemskit6 = . if itemskit =="" 

drop ikit* 

********** SECTION 4: OPEN-ENDED EMOTION CODING **********

/* Generate binary emotion variables from open-ended responses */
gen eo_fear = 0 
replace eo_fear =1 if upper(eo_oneword)== "AFRAID" |  upper(eo_oneword)=="SCARED" | upper(eo_oneword)=="FRIGHTENED" | upper(eo_oneword)=="NERVOUS" |upper(eo_oneword)=="JITTERY" | upper(eo_oneword)=="SHAKY"  
replace eo_fear =1 if upper(eo_oneword2)== "AFRAID" |  upper(eo_oneword2)=="SCARED" | upper(eo_oneword2)=="FRIGHTENED" | upper(eo_oneword2)=="NERVOUS" |upper(eo_oneword2)=="JITTERY" | upper(eo_oneword2)=="SHAKY"  
replace eo_fear = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_hostility = 0 
replace eo_hostility =1 if upper(eo_oneword)== "ANGRY" | upper(eo_oneword)=="HOSTILE" | upper(eo_oneword)=="IRRITABLE" | upper(eo_oneword)=="SCORNFUL" |upper(eo_oneword)=="DISGUSTED" | upper(eo_oneword)=="LOATHING"  
replace eo_hostility =1 if upper(eo_oneword2)== "ANGRY" |  upper(eo_oneword2)=="HOSTILE" | upper(eo_oneword2)=="IRRITABLE" | upper(eo_oneword2)=="SCORNFUL" |upper(eo_oneword2)=="DISGUSTED" | upper(eo_oneword2)=="LOATHING"  
replace eo_hostility = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_guilty = 0 
replace eo_guilty =1 if upper(eo_oneword)== "GUILTY" |  upper(eo_oneword)=="ASHAMED" | upper(eo_oneword)=="BLAMEWORTHY"  
replace eo_guilty =1 if upper(eo_oneword)== "GUILTY" | upper(eo_oneword2)=="ASHAMED" | upper(eo_oneword2)=="BLAMEWORTHY" 
replace eo_guilty = . if eo_oneword =="" & eo_oneword2 =="" 


gen eo_sad = 0 
replace eo_sad =1 if upper(eo_oneword)== "SAD" |  upper(eo_oneword)=="BLUE" | upper(eo_oneword)=="DOWNHEARTED" | upper(eo_oneword)=="ALONE" |upper(eo_oneword)=="LONELY"  
replace eo_sad =1 if upper(eo_oneword2)== "SAD" | upper(eo_oneword2)=="BLUE" | upper(eo_oneword2)=="DOWNHEARTED" | upper(eo_oneword2)=="ALONE" |upper(eo_oneword2)=="LONELY"  
replace eo_sad = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_otherneg = 0 
replace eo_otherneg =1 if upper(eo_oneword)== "ANXIOUS" |  upper(eo_oneword)=="DISTRESSED"   
replace eo_otherneg =1 if upper(eo_oneword2)== "ANXIOUS" | upper(eo_oneword2)=="DISTRESSED" 
replace eo_otherneg = . if eo_oneword =="" & eo_oneword2 =="" 


gen eo_joviality = 0 
replace eo_joviality =1 if upper(eo_oneword)== "HAPPY" |  upper(eo_oneword)=="JOYFUL" | upper(eo_oneword)=="DELIGHTED" | upper(eo_oneword)=="CHEERFUL" |upper(eo_oneword)=="EXCITED" | upper(eo_oneword)=="ENTHUSIASTIC" | upper(eo_oneword)=="LIVELY"| upper(eo_oneword)=="ENERGETIC" 
replace eo_joviality =1 if upper(eo_oneword2)== "HAPPY" | upper(eo_oneword2)=="JOYFUL" | upper(eo_oneword2)=="DELIGHTED" | upper(eo_oneword2)=="CHEERFUL" |upper(eo_oneword2)=="EXCITED" |upper(eo_oneword2)=="ENTHUSIASTIC" | upper(eo_oneword2)=="LIVELY"| upper(eo_oneword2)=="ENERGETIC"  
replace eo_joviality = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_selfassurance = 0 
replace eo_selfassurance =1 if upper(eo_oneword)== "PROUD" | upper(eo_oneword)=="STRONG" | upper(eo_oneword)=="CONFIDENT" | upper(eo_oneword)=="BOLD" |upper(eo_oneword)=="DARING" | upper(eo_oneword)=="FEARLESS"  
replace eo_selfassurance =1 if upper(eo_oneword2)== "PROUD" |  upper(eo_oneword2)=="STRONG" | upper(eo_oneword2)=="CONFIDENT" | upper(eo_oneword2)=="BOLD" |upper(eo_oneword2)=="DARING" | upper(eo_oneword2)=="FEARLESS"  
replace eo_selfassurance = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_attentiveness = 0 
replace eo_attentiveness =1 if upper(eo_oneword)== "ALERT" | upper(eo_oneword)=="ATTENTIVE" | upper(eo_oneword)=="CONCENTRATING" | upper(eo_oneword)=="DETERMINED"  
replace eo_attentiveness =1 if upper(eo_oneword2)== "ALERT" |  upper(eo_oneword2)=="ATTENTIVE" | upper(eo_oneword2)=="CONCENTRATING" | upper(eo_oneword2)=="DETERMINED"  
replace eo_attentiveness = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_otherpos = 0 
replace eo_otherpos =1 if upper(eo_oneword)== "CURIOUS" | upper(eo_oneword)=="HOPE"  
replace eo_otherpos =1 if upper(eo_oneword2)== "CURIOUS" | upper(eo_oneword2)=="HOPE"  
replace eo_otherpos = . if eo_oneword =="" & eo_oneword2 =="" 

gen eo_other = 0 
replace eo_other =1 if upper(eo_oneword)== "CALM" | upper(eo_oneword)=="AT EASE" | upper(eo_oneword)=="RELAXED" | upper(eo_oneword)=="CONFUSED" | upper(eo_oneword)=="AMAZED" | upper(eo_oneword)=="SURPRISED" | upper(eo_oneword)=="ASTONISHED" | upper(eo_oneword)=="OTHER"  
replace eo_other =1 if upper(eo_oneword2)== "CALM" |  upper(eo_oneword2)=="AT EASE" | upper(eo_oneword2)=="RELAXED" | upper(eo_oneword2)=="CONFUSED" | upper(eo_oneword2)=="AMAZED" | upper(eo_oneword2)=="SURPRISED" | upper(eo_oneword2)=="ASTONISHED" | upper(eo_oneword)=="OTHER"  
replace eo_other = . if eo_oneword =="" & eo_oneword2 =="" 


********** SECTION 5: AGGREGATE EMOTION SCALES **********

/* Create composite positive, negative, other emotion scales */
gen eopen_positive =0 
replace eopen_positive =1 if eo_joviality==1 | eo_selfassurance==1 |  eo_attentiveness ==1 | eo_otherpos ==1 
replace eopen_positive =. if eo_joviality==. | eo_selfassurance==. |  eo_attentiveness ==. | eo_otherpos ==. 

gen eopen_negative = 0 
replace eopen_negative =1 if eo_otherneg ==1 | eo_sad==1 | eo_guilty ==1 | eo_hostility ==1 | eo_fear==1 
replace eopen_negative =. if eo_otherneg ==. | eo_sad==. | eo_guilty ==. | eo_hostility ==. | eo_fear==. 

gen eopen_other =0 
replace eopen_other=1 if eo_other ==1 
replace eopen_other=. if eo_other==. 

/*Frequencies of each word */
tab eo_oneword 
tab eo_oneword2 

/*Create new information seeking behaviour variable */

gen info_seeking = . 
replace info_seeking = 0 if share ==0 & reminder ==0 
replace info_seeking = 1 if share ==1 | reminder ==1 
replace info_seeking =2 if share ==1 & reminder ==1 


********** SECTION 6: DESCRIPTIVE ANALYSES **********

/* Frequencies, chi-square tests, t-tests, Kruskal-Wallis */
tab treatment makeplan , chi2  /*treatment (for control) vs outcome1: makeplan*/
tab treatment intention , chi2  /*treatment (for control) vs outcome2: intention*/


/* Kruskal-Wallis test vs t test for treatment vs intention */

kwallis intention, by(treatment)  /* Kruskal-Wallis is Statistically significant */

ttest intention if treatment !=0, by(treatment) unequal  /* t =   0.6447*/
ttest intention if treatment !=1, by(treatment) unequal  /* t =  -1.9079*/
ttest intention if treatment !=2, by(treatment) unequal  /* t =  -2.5694*/

/*Demographics summary statistics  */

/* Compute proportions and chi-square tests for country and gender*/
tab country treatment, chi2 
tab gender treatment,chi2  

/* Compute means and standard deviations for past experience*/
sum experience if treatment == 0 
sum experience if treatment ==1 
sum experience if treatment ==2 
/*t test for pairwise comparisons between treatment groups */
ttest experience if treatment !=0, by(treatment) unequal  /* t = 1.5720 p = 0.1164 */
ttest experience if treatment !=1, by(treatment) unequal  /* t=-0.6304 p = 0.5287*/
ttest experience if treatment !=2, by(treatment) unequal  /* t =  -2.1457 p=  0.0323*/
	
tab country gender 
sum country experience 

/* Kruskal-Wallis test for country and gender */
kwallis country, by(treatment)
kwallis gender, by(treatment)

	
********** SECTION 7: VARIABLE LABELING & CLEANUP **********

label define treatment 0 "Control" 1 "Directive" 2 "Music"
label values treatment treatment
/*Rename emotionlist variables*/

rename emolist1 Happiness 
rename emolist2 Worry 
rename emolist3 Joy 
rename emolist4 Fear 
rename emolist5 Pride 
rename emolist6 Anxiety
rename emolist7 Hope
rename emolist8 Safety 
rename emolist9 Other

/* Label gender variable and check for non-binary */
tab gender /* no non-binary participants */
label define sex 1 "Male" 2 "Female" 
label values gender sex

/*rename risk perception and relabel*/
rename internalization prisk
label variable prisk "Perceived Risk"

/*label values of valence variable */
sum valence, detail
tab valence


/*Rename and relabel affected_emotionally variable */
rename affected_emotionally affected
label variable affected "Affected"

/*Rename and relabel messageliking variable*/
rename messageliking liking
label variable liking "Liking"

/*Label countries */
label define country 1 "Anguilla" 2 "Bahamas" 3 "Barbados" 4 "Grenada" 5 "St Lucia" 6 "Trinidad & Tobago" 8 "Jamaica" 9 "Dominica" 10 "Belize" 11 "Antigua & Barbuda" 12 "St Kitts and Nevis"
label values country country

/*frequencies for emotions open words after categorization */
tab eo_oneword 
tab eo_oneword2
//Correct grammar in subscribe variable//
rename suscribe subscribe
label variable subscribe "Subscribe"



/*Drop string variables */

drop emotionsopen 
drop emotionlist 
drop planhome 
drop itemsshelter 
drop itemskit 

********** SECTION 8: HANDLE MISSING DATA **********
/* Replace missing intentions, subscriptions, etc. */
replace subscribe = 1 if subscribe == 2

/*Replace missing values in emotionlist variables*/

replace Happiness = 0 if missing(Happiness)
replace Worry = 0 if missing(Worry)
replace Joy =0 if missing(Joy)
replace Fear = 0 if missing(Fear)
replace Pride = 0 if missing(Pride)
replace Anxiety = 0 if missing(Anxiety)
replace Hope = 0 if missing(Hope)
replace Safety = 0 if missing(Safety)
replace Other = 0 if missing(Other)



replace subscribe = 1 if subscribe ==2 /* in the survey 1=no and 2=yes */
replace subscribe =0 if missing(subscribe) /*two observations missing = did not subsribe */

replace makeplan =0 if missing(makeplan) /*one missing observation from someone who did not complete the entire survey*/
tab makeplan

gen intentionmiss = intention
replace intentionmiss = . if missing(intention)
sum intentionmiss

replace intention = 1 if missing(intention) /*three missing observations coded into 1 meaning no intention at all*/
sum intention
