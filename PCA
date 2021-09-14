clear
capture log close

* Set path macros for saving graphs
global pathStata "C:/Users/tessam/Documents/GH_Manuscript"

* Read data from directory
import delimited "$pathStata/PCA_GH_paper.csv"
cd "$pathStata"


*global pathStata "U:/GlobalHealthIndicators/Stata/"
global gph "xline(0, lwidth(med) lpattern(tight_dot) lcolor(gs10)) yline(0, lwidth(med) lpattern(tight_dot) lcolor(gs10)) ylab(, nogrid) graphregion(fcolor(white) lcolor(white))"

* Label variables for graphs
la var hiv_prev_w_15 "HIV Prevalence"
la var unmetneedmm "Unmet Need"
la var hsv2prevalence2012  "HSV2 Prevalence"
la var hpv_prev15 "HPV Prevalence"

* Fix Congo names for graph tidying.
replace countryname ="CONGO-Brazzaville" if countryname == "CONGO (Brazzaville)"
replace countryname ="CONGO-Kinshasa" if countryname == "CONGO (Kinshasa)"

*Set a global variable for the graphics schemes
global gcustom "mlabs(tiny) mlabc(maroon) mc(maroon)"

* Create position offsets so names do not overlap when mapping
g pos = 12
replace pos = 6 if regexm(countryname, "(MALAWI|EQUATORIAL GUINEA|CONGO-Brazzaville|CHAD|RWANDA)")
replace pos = 3 if regexm(countryname, "(SOUTH SUDAN|CAMEROON|CONGO-Kinshasa)")
replace pos = 9 if regexm(countryname, "(ETHIOPIA|SWAZILAND|ERITREA)")
replace pos = 8 if regexm(countryname, "(ANGOLA)")
replace pos = 4 if regexm(countryname, "BURUNDI")

*Replace missing values with sample averages
foreach x of varlist hiv_prev_w_15 unmetneedmm hsv2prevalence2012 hpv_prev15 {
	egen tmp = mean(`x')
	replace `x' = tmp if `x' ==.
	drop tmp
	}
*end

	
* Produce correlation matrix and save to excel file called "sti.csv"
eststo clear
estpost correlate hiv_prev_w_15 unmetneedmm hsv2prevalence2012 hpv_prev15, matrix
esttab using sti.csv, not unstack compress noobs replace

* Run PCA on the four key variables, predict PC1 & PC2
pca hiv_prev_w_15 unmetneedmm hsv2prevalence2012 hpv_prev15, means
estat kmo

pca unmetneedmm hsv2prevalence2012 hpv_prev15
alpha unmetneedmm hsv2prevalence2012 hpv_prev15

* Run jackknifed pca to check robustness of results
jackknife: pca hiv_prev_w_15 unmetneedmm hsv2prevalence2012 hpv_prev15

* Predict the four principal components for mapping
predict PC1 PC2 PC3 PC4

* Plot the first two loadings to see how the variables cluster
loadingplot, mlabs(small) mlabc(maroon) mc(maroon) 

* Create a plot of the scores based on the first two components
scoreplot, mlabel(countryname) $gcustom mlabv(pos) $gph

* Now plot a subset of the data with PC1 > 1 and save to Stata folder. 
scoreplot if PC1>=0.5, mlabel(countryname) $gcustom mlabv(pos) title("PCA Scores Greater than 1") $gph
graph save "$pathStata/pca_subset.gph", replace

* Export the data to a .csv and save in the Stata folder
export delimited using "$pathStata/mpt_pca_results.csv", replace
