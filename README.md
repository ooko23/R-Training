# STI-DATA ANALYSIS
### Project Overview
This project aims to understand some of the risk factors that leads to contraction of STIs among youths.
### Data Sources
STI-data unclean. This data contains risk factors for contracting STIs
### Tools
- Excel: data entry
- STATA: data cleaning and analysis
- PowerBI: Creating reports
### Data Cleaning
In the initial data preparation phase we performed the following tasks:
1. Data loading and inspection
2. Handling missing values
3. Data cleaning and formatting
### Exploratory data analysis
EDA involved exploring the sales data to answer key questions such as;
- What is the overall sales trend?
- Which products are top sellers?
- What are the peak sales periods?
### Data Analysis 
```
*** importing datasets
import excel "C:\Users\admin\Downloads\STIData.xls",sheet(STIData) firstrow


*** viewing dataset
describe
codebook A1Age
codebook A5MaritalStatus

***********************************************************************************************************************************************************************************

*** Compute a variable
egen avg_age=mean(A1Age)
browse avg_age

tabstat IdNumber,by(A1Age) stat(mean median sd)
bysort CaseStatus:summarize Height Weight

*** listing
list A1Age
*** sorting
*** sorting in ascending order
browse A1Age
sort A1Age

*** sorting in descending order
gsort- A1Age
*** checking for inconsistency
codebook CaseStatus
list IdNumber if CaseStatus==3
***replacing inconsistencies
replace CaseStatus=1 if IdNumber==1
replace CaseStatus=2 if IdNumber==31
codebook CaseStatus
*** checking missing values
tab Sex, missing 
codebook sex
list IdNumber if Sex==""
replace Sex="Female" if IdNumber==213
replace Sex="Male" if IdNumber==48
codebook Sex
*** handling duplicates
duplicates list IdNumber
browse if IdNumber==51
drop if IdNumber==51 & A1Age==23
browse if IdNumber==51
order A1Age Date
*** data manipulation-26/01/2023 
*** generating new variables
gen age_cat=.
replace age_cat=1 if A1Age>0 & A1Age<=24
replace age_cat=2 if A1Age>=25 & A1Age<=34
replace age_cat=3 if A1Age>=35 & A1Age<=44
replace age_cat=4 if A1Age>44
codebook A1Age
label define age_cat 1"children"2"youth"3"adult"4"old"
label values age_cat age_cat
codebook age_cat
***Populating new variables by calculation
gen bmi=Weight/((Height/100)^2)
browse bmi
gen bmi2=round(bmi,1)
browse bmi2
gen bmi_cat=""
replace bmi_cat="obese" if bmi2>30
replace bmi_cat="overweight" if bmi2>=25 & bmi2<=30
replace bmi_cat="underweight" if bmi2<18
replace bmi_cat="normal" if bmi2>=18 & bmi2<=24
codebook bmi_cat
browse bmi_cat
codebook A2Occupation
gen occupation=""
replace occupation="employed" if A2Occupation=="2 informal" | A2Occupation=="3 formal"
replace occupation="unemployed" if A2Occupation=="1 unemployed" | A2Occupation=="4 student"
codebook occupation
***subseting a variable
keep if Sex=="female"
***converting numeric to string
encode occupation,gen(occupation2)
*** working with dates
insheet using "C:\Users\admin\Downloads\link_unformatted_extraTrain.csv"
browse _savepoint_timestamp
gen date=substr(_savepoint_timestamp,1,10)
browse date
*** converting date from string to numeric
gen date2=date(date,"YMD")
format date2 %td
br date2 date
***for sd_a_birthdate
gen dob=substr( sd_a_birthdate,1,10)
gen dob2=date(dob,"YMD")
format dob2 %td
br dob2
gen age=round((date2-dob2)/365,1)
br age
***ROTATING DATASETS
*appending-combines the dataset vertically by adding observations; should have same variable names
* merging-combines the dataset horizontally by adding variables
use "C:\Users\admin\Downloads\seta_append.dta",clear
append using "C:\Users\admin\Downloads\setb_append.dta"
rename a1age age
browse age
*now a1age has disappeared
browse a1age
***merging
*1:1
clear
use "C:\Users\admin\Downloads\set1_1to1.dta"
merge 1:1 idnumber using "C:\Users\admin\Downloads\set2_1to1.dta"
*1:many
clear
use "C:\Users\admin\Downloads\set1_1tom.dta"
merge 1:m idnumber using "C:\Users\admin\Downloads\set2_1tom.dta"
browse
*if we want to generate a new variable ie IDnumber for our dataset
gen id=_n
**When ordering our variable;
order id idnumber
*many:1

***RESHAPING DATASETS
*It entails shaping from a wide format(row format) to a long format(column format)
clear
import excel "C:\Users\admin\Downloads\TEST-TREAT-TRACK - FEASIBILITY EVALUATION OF THE TTTT STRATEGY (created 2019-07-17) 2019-07-17.xlsx",sheet(Repeat- Repeat_group2C) firstrow
browse
***CHOOSING A STATISTICAL TEST
clear
import excel "C:\Users\admin\OneDrive\Desktop\data1.xlsx",sheet(Sheet1) firstrow
browse 
gen j=_n
reshape wide age, i(id) j(j)
br
clear
import excel "C:\Users\admin\OneDrive\Desktop\data1.xlsx",sheet(Sheet2) firstrow
br
gen j=_n
reshape long age, i(id) j(num)
drop j
drop num
br
***summarizes data for id 1
sum age if id==1
**we want to use STIData
clear
import excel "C:\Users\admin\Downloads\STIData.xls",sheet(STIData) firstrow
sum A1Age,d
replace CaseStatus=1 if IdNumber==1
replace CaseStatus=2 if IdNumber==31
replace CaseStatus=0 if CaseStatus==2
codebook CaseStatus
gen occupation=""
replace occupation="employed" if A2Occupation=="2 informal" | A2Occupation=="3 formal"
replace occupation="unemployed" if A2Occupation=="1 unemployed" | A2Occupation=="4 student"
codebook occupation
tab occupation CaseStatus,row
label define CaseStatus 1"Yes"0"No"
label values CaseStatus CaseStatus
codebook CaseStatus
**replacing YES and NO with positive and negative
clear
import excel "C:\Users\admin\Downloads\STIData.xls",sheet(STIData) firstrow
sum A1Age,d
duplicates list IdNumber
browse if IdNumber==51
drop if IdNumber==51 & A1Age==23
browse if IdNumber==51
replace CaseStatus=1 if IdNumber==1
replace CaseStatus=2 if IdNumber==31
replace CaseStatus=0 if CaseStatus==2
codebook CaseStatus
gen occupation=""
replace occupation="employed" if A2Occupation=="2 informal" | A2Occupation=="3 formal"
replace occupation="unemployed" if A2Occupation=="1 unemployed" | A2Occupation=="4 student"
codebook occupation
tab occupation CaseStatus,row
label define CaseStatus 1"Positive"0"Negative"
label values CaseStatus CaseStatus
codebook CaseStatus
tab occupation CaseStatus,row
*report
Majority of the people who were employed were negative (N=75(54.79%) whereas majority of the unemployed were positive(N=51(56.67%))
tab Sex CaseStatus,row
*report
Majority of the female were negative (N=55(51.40))whereas majority of the people who were male were positive (N=59(50.43)) 
tab A5MaritalStatus CaseStatus,row
*report
Majority of the people who 
tab A5MaritalStatus CaseStatus,col
***DATA VISUALIZATION
**1. Pie Chart
graph pie,over(A5MaritalStatus) plabel(_all percent) title("Proportion of Marital Status") note("case of 2023") legend(off)
graph pie,over(Sex) plabel(_all name) title("Gender") note("2020") plabel(_all percent)
**2. Boxplot
clear
sysuse auto.dta
graph box mpg,over(foreign)noout title("Boxplot")
**3. Histogram
histogram rep78, percent 
histogram rep78,discrete percent 
**Drawing Normal curve
histogram rep78,discrete percent normal
**4. testing for normality
*i)skewness
sktest rep78
*p-value=0.9286. since p-value is greator than 0.05, we fail to reject H0 and conclude that its normally distributed
*ii)Shapiro wilk
swilk rep78
*iii)Shapiro francia
sfrancia rep78
**4. Scatterplot
scatter weight displacement,by(foreign)
twoway lfit weight displacement, by(foreign)
**Correlation
*i)correlation
corr trunk mpg
*strong negative correlation btw trunk and mpg
*ii)spearman rho
spearman trunk mpg
*strong negative correlation btw trunk and mpg
*iii)Kendall's tau
ktau trunk mpg
*weak negative correlation
***COMPARISON OF MEANS-31/01/2023
**1.Independent ttest
import excel "C:\Users\admin\Downloads\STIData.xls",sheet(STIData) firstrow
ttest Weight,by(Sex)
*there is no difference in means
**2. Paired ttest
import excel "C:\Users\admin\OneDrive\Desktop\ttest.xlsx",sheet(Sheet1) firstrow
ttest salaryafter==salarybefore
*p-value is less than 0.05, hence we reject Ho and conclude that there is a significant diff. in means
**3. One Way ANOVA
import excel "C:\Users\admin\Downloads\STIData.xls",sheet(STIData) firstrow
oneway Weight A2Occupation
*p-value=0.0161
oneway Weight A5MaritalStatus2
*p-value=0.7227
**4. Post Hoc
encode A5MaritalStatus, gen(A5MaritalStatus2)
pwmean Weight,over(A5MaritalStatus2) mcompare(tukey)effects
**5. Two-Way ANOVA-two categorical variables
encode A2Occupation, gen(Occupation2)
anova Height Occupation2##A5MaritalStatus2
*fail to reject Ho, and conclude that there is no significant interaction between marital status and occupation on height 
pwmean Height,over(Occupation2 A5MaritalStatus2) mcompare(tukey)effects
***BIVARIATE ANALYSIS
*1. Linear regression
regress Height i.CaseStatus
*Holding other factors constant, a unit change in CaseStatus results to an increase in Height by 16.296% on people who were negative as compared to the ones who were positive.
regress Height ib2.A5MaritalStatus2
glm CaseStatus i.Occupation2,link (logit) family(binomial)robust eform
*People who were were informally employed had lower lower odds (0.401)of having the disease compared to the unemployed
glm CaseStatus i.Occupation2,link(log) family(poisson) robust eform

```
### Results

### Recommendations

### Limitations

### References
