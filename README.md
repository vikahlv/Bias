# Bias simulations
#### Last modified: 2021-06-02
#### Author: Viktor H. Ahlqvist

The simulations are performed as follows:
1. Create individual-level data
2. Perform regression
3. Store the estimate
4. Repeat 100 times and average
5. Modify input parameters

#### All simulation code is available under their respective folder as do-files for Stata version 15.1

The general code for the simulations looks as follows:

```
	version 15.1
	set seed 1
	clear*
	program myreg, rclass
	clear
	set obs 10000
	local a0 = 0.05
	local a1 = 1.5
	local a2 = 10
	local b0 = 0.01
	local b1 = 1.5
	local b2 = 1
	local b3 = 1.25
	gen z = .
	gen u = .
	gen x = .
	gen y = .
	replace z = rbinomial(1, .5)
	replace u = rbinomial(1, .5)
	replace x = rbinomial(1, `a0' * `a1'^u * `a2'^z)  
	replace y = rbinomial(1, `b0' * `b1'^u * `b2'^x * `b3'^z) 
	glm y x, family(binomial) link(log) eform
	return scalar y_b_x = _b[x]
	glm y x z, family(binomial) link(log) eform
	return scalar y_b_x_z = _b[x]
	glm y x z u, family(binomial) link(log) eform
	return scalar y_b_x_z_u = _b[x]
	glm y x if z==1, family(binomial) link(log) eform
	return scalar y_b_x_z1 = _b[x]
	glm y x if z==0, family(binomial) link(log) eform
	return scalar y_b_x_z0 = _b[x]
	glm y x u if z==1, family(binomial) link(log) eform
	return scalar y_b_x_z1_u = _b[x]
	end
```
In order to run the simulation, we can vary the input parameters by replacing an input parameter argument i
```	
	tempname memhold
	postfile `memhold' mean_crude_or_X mean_z1_or_x mean_truth_or_X mean_z0_or_x z_y_effect mean_crude_bias mean_z1_bias using "XXXXXXXXXXXXXXXXXXXXX" , replace
	forvalues i = 1(.01)2 {
	simulate y_b_x = r(y_b_x) y_b_x_z1 = r(y_b_x_z1) y_b_x_z_u = r(y_b_x_z_u) y_b_x_z0 = r(y_b_x_z0), ///
	    reps(100): sim `i'
	
	gen crude_or_X = exp(y_b_x)
	egen mean_crude_or_X = mean(crude_or_X)
	
	gen z1_or_x = exp(y_b_x_z1)
	egen mean_z1_or_x = mean(z1_or_x)
	
	gen truth_or_X = exp(y_b_x_z_u)
	egen mean_truth_or_X = mean(truth_or_X)
	
	gen z0_or_x = exp(y_b_x_z0)
	egen mean_z0_or_x = mean(z0_or_x)
	
	
	gen z_y_effect =  `i'
	
	gen z1_bias = (z1_or_x/truth_or_X)-1
	egen mean_z1_bias = mean(z1_bias)
	
	gen crude_bias = (crude_or_X/truth_or_X)-1
	egen mean_crude_bias = mean(crude_bias)
	
	keep in 1/1
	post `memhold' (mean_crude_or_X) (mean_z1_or_x) (mean_truth_or_X) (mean_z0_or_x) (z_y_effect) (mean_crude_bias) (mean_z1_bias)
	}
	postclose `memhold'	


```
