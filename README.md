# Stata II (Intermediate) Week 6-8 Project 

### 1. Research Aim
 To determine the association between self-reported health and mortality.
  
### 2. Data Sources
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

* Further updates will be coming in Week 7 and beyond.
