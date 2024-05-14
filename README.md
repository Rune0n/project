# Stata II (Intermediate) Week 6-8 Project 

### Research Aim
 To determine the association between self-reported health and mortality.
  
### Data Sources
  [National Health and Nutrition Examination Survey](https://www.cdc.gov/nchs/nhanes/index.htm)
  - [NHANES Survey Dataset](https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/DEMO.XPT)
  - [NHANES Mortality Dataset](https://ftp.cdc.gov/pub/HEALTH_STATISTICS/NCHS/datalinkage/linked_mortality/)
    - `NHANES_1999_2000_MORT_2019_PUBLIC.dat`
    
## Data

- **Data Acquisition and Preparation**
  * Survey Data
     * Import the survey data from the 1999-2000 National Health and Nutrition Examination Survey (NHANES):
    ```stata
       import sasxport5 "https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/DEMO.XPT", clear
    ```
   * Mortality Follow-up Data
      * Obtain follow-up mortality data to analyze over a 20-year period from the National Center for Health Statistics (NCHS). Detailed linkage instructions are available on the [Linked Mortality](https://ftp.cdc.gov/pub/) page.
   ```stata
   //data
   global mort_1999_2000 https://ftp.cdc.gov/pub/HEALTH_STATISTICS/NCHS/datalinkage/linked_mortality/NHANES_1999_2000_MORT_2019_PUBLIC.dat
   //code
   cat https://ftp.cdc.gov/pub/HEALTH_STATISTICS/NCHS/datalinkage/linked_mortality/Stata_ReadInProgramAllSurveys.do
   ```

## Code Development
- **Edit and Rename Provided Script**
  * Download `Stata_ReadInProgramAllSurveys.do`, edit it and rename it to `followup.do`. Commit the changes with the description “Updated DEMO.XPT linkage .do file".

- **Data merging**
  * Execute the following Stata code to merge the survey data with the mortality data, ensuring alignment on the unique sequence numbers:
    ```stata
    //use your own username/project repo instead of the class repo below
    global repo "https://github.com/Rune0n/project/main/"
    do ${repo}followup.do
    save followup, replace 
    import sasxport5 "https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/DEMO.XPT", clear
    merge 1:1 seqn using followup
    lookfor follow
    ```

## Key Parameters for Analysis

- **Import the self-report health assessment data**
  * Import the specific health questionnaire data and prepare it for analysis in Week 7:
    ```stata
    import sasxport5 "https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/HUQ.XPT", clear
    ```
  * Here’s a first iteration of a script that answers they project main goal. Save it as project.do and upload it to you repo. Keep updating it over the next two weeks, with a meaningful commit statement each time for version control.
    ```stata
    global repo "https://github.com/Rune0n/project/main/"
    do ${repo}followup.do
    save followup, replace
    import sasxport5 "https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/DEMO.XPT", clear
    merge 1:1 seqn using followup
    lookfor follow
    lookfor mortstat permth_int eligstat
    keep if eligstat==1
    capture g years=permth_int/12
    codebook mortstat
    stset years, fail(mortstat)
    sts graph, fail
    save demo_mortality, replace
    import sasxport5 "https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/HUQ.XPT", clear
    merge 1:1 seqn using demo_mortality, nogen
    sts graph, by(huq010) fail
    stcox i.huq010
    ```

## Inferences

Please review [documentation](https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/HUQ.htm#HUQ010) for the file `HUQ.XPT`, which includes the variable `huq010`.
  * Variable of interests is huq10, which is general health condition with the following categories:
       * Excellent,
       * Very good,
       * Good,
       * Fair, or
       * Poor?
       * Refused
       * Don't know
       * Missing

  * Run the following:
    ```stata
    merge 1:1 seqn using demo_mortality, nogen
    sts graph, by(huq010) fail
    stcox i.huq010
    ```
    ```stata
    import sasxport5 "https://wwwn.cdc.gov/Nchs/Nhanes/1999-2000/HUQ.XPT", clear 
    huq010 
    desc huq010
    codebook huq010
    ```

## Week 7 Analyses

Click [here](dyndoc.html) to view nonparametric and semiparametric risk estimates from Stata.

### Nonparametric Model

Click [here](nonpara.png).
```stata
cls 
//1. data
global repo "https://github.com/jhustata/project/raw/main/"
global nhanes "https://wwwn.cdc.gov/Nchs/Nhanes/"

//2. code
do ${repo}followup.do
save followup, replace 
import sasxport5 "${nhanes}1999-2000/DEMO.XPT", clear
merge 1:1 seqn using followup, nogen
save survey_followup, replace 

//3. parameters
import sasxport5 "${nhanes}1999-2000/HUQ.XPT", clear
tab huq010 
merge 1:1 seqn using survey_followup, nogen keep(matched)
rm followup.dta
rm survey_followup.dta
g years=permth_int/12
stset years, fail(mortstat)
replace huq010=. if huq010==9
label define huq 1 "Excellent" 2 "Very Good" 3 "Good" 4 "Fair" 5 "Poor"
label values huq010 huq 
levelsof huq010, local(numlevels)
local i=1
foreach l of numlist `numlevels' {
    local vallab: value label huq010 
	local catlab: lab `vallab' `l'
	global legend`i' = "`catlab'"
	local i= `i' + 1
}
save week7, replace 
sts graph, ///
    by(huq010) ///
	fail ///
	per(100) ///
	ylab(0(20)80 , ///
	    format(%2.0f) ///
	) ///
	xlab(0(5)20) ///
	tmax(20) ///
	ti("Self-Reported Health and Mortality") ///
	legend( ///
	    order(5 4 3 2 1) ///
		lab(1 "$legend1") ///
		lab(2 "$legend2") ///
		lab(3 "$legend3") ///
		lab(4 "$legend4") ///
		lab(5 "$legend5") ///
		ring(0) pos(11) ///
	)
graph export nonpara.png, replace
```

### Semi-parametric Model

Click [here](semipara_unadj.png).
```stata
/* -- earlier code --*/
stcox i.huq010, basesurv(s0)
matrix define mat = r(table)
matrix list mat 
matrix mat = mat'
svmat mat
preserve 
keep mat*
drop if missing(mat1)
rename (mat1 mat2 mat3 mat4 mat5 mat6 mat7 mat8 mat9)(b se z p ll ul df crit eform)
g x=_n
replace b=log(b)
replace ll=log(ll)
replace ul=log(ul)
twoway (scatter b x) || ///
       (rcap ll ul x, ///
	       yline(0, lcol(lime)) ///
		   ylab( ///
		       -2.08 "0.125" ///
			   -1.39 "0.25" ///
			   -.69 "0.5" ///
			     0 "1"  ///
			   .69 "2" ///
			   1.39 "4" ///
			   2.08 "8" ///
			   2.78 "16") ///
		   legend(off)  ///
		xlab( ///
           1 "$legend1" ///
		   2 "$legend2" ///
		   3 "$legend3" ///
		   4 "$legend4" ///
		   5 "$legend5") ///
	   xti("Self-Reported Health") ///
	   	   ) 
graph export semipara_unadj.png, replace 
graph save semipara_unadj.gph, replace 
restore
```
