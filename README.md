# Aims and Process
The function simulate.trajectory simulates longitudinal data for one variable for a series of individuals. The data takes the form of a pattern or trajectory with two sections joined by a step change. The two sections can take a range of forms parameterised in terms of time. Individual parameters for trajectories are randomly sampled from a population distribution and recorded so they are known. A first order autocorrelation structure can be specified for within-individual error terms. Between individual correlations can also be specified so the function can be re-run several times to form clustered data. Measurement times can be sampled to fall in waves or to be completely irregular.

There are three main elements the function uses to generate the data required:

A.	Population distributions for trajectory parameters

B.	Measurement time sampling specifications

C.	Error distribution parameters including within and between person correlations

A is used to randomly sample individual trajectory parameter values. These are used to generate individual values that exactly follow individual parameter values at each time point. B is used to generate times at which each individual is measured. A and B are combined to generate trajectory values for individuals with no error included. C is used to create an observation by observation covariance matrix for error terms. This reflects the time between measurements and the distance between individuals if spatial groups are specified, or alternatively the correlation specified within individuals. The correlation matrix is used to generate error terms which follow a normal distribution and these are added to the individual values for the data at each time point. 

# Required libraries
The function `simulate.trajectory` requires the libraries `ggplot2`, `gridExtra`, `Matrix`, `matrixcalc`, `MASS` and dependencies.

# Arguments
|Argument|Form|Default|Description|
|---|---|---|---|
|`n.obs`|numeric, length=1|`1`|Number of trajectories to generate
|`max.time`|numeric, length=1|`10`|Maximum time of window to simulate data in (minimum is always zero)|
|`record.times`|numeric, length>=1|`1:n.measures`|Two options. For wave measurements, specify the times at which the waves occur. Recording times will vary around the specified times. For irregular measurements, specify the number of records collected and a uniform distribution will be used to choose this many recording times for each trajectory.|
|`record.times.vary`|numeric, length=1|`1`|Only required for wave measurements. Amount of variation of actual measurement times around those specified in record.times.|
|`part.1.average`|character, 3<=length<=4|no default|Population distribution of the average value of the first trajectory section. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2`. Two of `part.1.average`, `part.2.average` and `value.step.change` must be specified. If all three are specified, `part.2.average` is ignored.|
|`part.2.average`|character, 3<=length<=4|no default|Population distribution of the average value of the second trajectory section. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2`. Two of `part.1.average`, `part.2.average` and `value.step.change` must be specified. If all three are specified, `part.2.average` is ignored.|
|`part.1.slope`|character, length=3|`c("rnorm", "0", "1", "1")`|Population distribution of the slope parameter for the first trajectory section. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2, multiplier`. The `multiplier` defaults to 1 and can be specified in terms of `t` (for time) if a non-linear pattern is required, for example, specifying `t` will give a quadratic curve and `t^2` cubic.|
|`part.2.slope`|character, length=3|`c("rnorm", "0", "1", "1")`|Population distribution of the slope parameter for the second trajectory section. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2, multiplier`. The `multiplier` defaults to 1 and can be specified in terms of `t` (for time) if a non-linear pattern is required, for example, specifying `t` will give a quadratic curve and `t^2` cubic.|
|`time.step.change`|character, length=3|`c("rnorm", "0", "1", "1")`|Population distribution of the time at which the step change between trajectory sections occurs. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2`.|
|`value.step.change`|character, length=3|no default|Population distribution of the average value of the step change between trajectory sections. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2`. Two of `part.1.average`, `part.2.average` and `value.step.change` must be specified. If all three are specified, `part.2.average` is ignored.|
|`sd.error`|character, length=3|`sd.error=c("rlnorm","0.5", "0.5")`|Population distribution of the standard deviation of total random error terms for each individual. Individual averages will be sampled from this distribution. Takes the form `distribution, parameter1, parameter2`. The absolute value of any negative values will be used.|
|`between.cor`|numeric, length=1|`0`|Correlation of error terms between individuals at the same time point. Must be between -1 and 1.|
|`within.cor`|numeric, length=1|`0`|Correlation of error terms within individuals at adjacent time points. An order 1 autocorrelation structure is specified within individuals. Must be between -1 and 1.|
|`dayreps`|numeric, length=1|`1`|Routes can be measured over the time specified on multiple occasions (for example, different days). This specifies the number of these occasions.|
|`dayreps.cor`|numeric, length=1|`0`|Error correlation between data measured at the same time on the same route on consecutive days.|
|`spacegroups`|numeric, length=1|no default|Number of spatial groups or clusters of individuals to be included in the dataset.|
|`spacegroups.size`|numeric, length=1|no default|The average deviation of the x and y coordinates for an individual in a spatial group from its centre (the distribution of points follows a bivariate normal distribution, this is the standard deviation of both distributions that comprise this).|
|`origin.dest`|logical, length=1|`FALSE`|If `TRUE`, data will be associated with a 'route' between two spatial points.|
|`area.x`|numeric, length=1|no default|Upper bound for x-coordinates of the centre of each spatial group. These will be sampled from a uniform distribution spanning from zero to this value.|
|`area.y`|numeric, length=1|no default|Upper bound for y-coordinates of the centre of each spatial group. These will be sampled from a uniform distribution spanning from zero to this value.|
|`sd.ratio`|numeric, length=1|`0.5`|Random variation in parameters in different spatial or route groups as a fraction of the random variation in parameters specified in `part.1.average`, `part.1.slope` etc.|
|`sd.ratio.day`|numeric, length=1|`0.5`|Random variation in parameters on different `dayreps` as a fraction of the random variation in parameters specified in `part.1.average`, `part.1.slope` etc.|
|`plot`|logical, length=1|`TRUE`|Whether the summary plot is produced.|
# Output
The function outputs a list containing 5 elements:

|Element|Description|
|---|---|
|`data`|Data frame containing simulated data in long format|
|`simulation.summary`|Data frame containing variables showing individual input parameters, values expected from a simulation with no error (given discrete measurement times), and actual parameters from simulated data.|
|`plot`|Graphical object containing spaghetti plot of trajectories and differences of average values and step change values from inputs and expected values. Can be displayed using `plot()` function.|
|`section1.function`|Individual functions specified for first trajectory section (including multiplier)|
|`section2.function`|Individual functions specified for second trajectory section (including multiplier)|
|`coordinates.info`|Contains two parts. The first contains central coordinates for each spatial group. The second contains individual level coordinates and spatial group membership.|
# Warning messages
Common warning messages include the following:

`In max(recordtime[which(recordtime[, i] <= prejumpt[i]), i]) :`
  `no non-missing arguments to max; returning –Inf`

`In min(recordtime[which(recordtime[, i] >= postjumpt[i]), i]) :`
  `no non-missing arguments to max; returning Inf`

`Removed 1 rows containing non-finite values (stat_density).`

These occur if wave or irregular sampling is specified and an individual has no recording times either before or after the step change. This means some summary values cannot be calculated but does not cause any other issues.

# Example of use
The following function call simulates data for 50 individuals over 30 time points. The first function section follows a log(time) curve and the second follows a cos(time)*time curve. The coefficients for these are from a lognormal distribution with parameters 3 and 0.2 and normal distribution with parameters 1 and 0.5, respectively. Individual step-change times are chosen from a uniform distribution spanning from time=10 to time=20 and the step change value is chosen from a normal distribution with mean -100 and standard deviation 3. While error terms are normally distributed, individual level error standard deviations are chosen from a lognormal distribution. Adjacent measurements within individuals have an error correlation of 0.5. Simultaneous measurements for different individuals have an error correlation of 0.5. Measurements are taken irregularly, 10 times for each individual. The figure below shows the plots produced by the function.

`B <- simulate_trajectory(n.obs=50 , max.time=30, record.times=10,
part.1.average=c("rnorm","100", "3"), part.1.slope=c("rlnorm", "3", "0.2", "log(t)/t") , 
part.2.slope=c("rnorm", "1", "0.5", "cos(t)") , time.step.change=c("runif","10", "20") , value.step.change=c("rnorm","-100", "3"), sd.error=c("rlnorm","1", "0.5"), 
between.cor=0.3, within.cor=0.5, spacegroups=2, spacegroup.size=1, area.x=30, area.y=30)`

![Graphs produced by example code](https://user-images.githubusercontent.com/25984118/48132614-1e82fd80-e28c-11e8-87fd-2c759ff75593.png)


# Limitations
A number of limitations have been identified to date, however, more will likely become apparent as the function is used more frequently.

•	A step change must be specified for the distribution to run. As the first and second section slope parameters (at the individual level) are randomly selected from orthogonal distributions, the function cannot be manipulated to follow exactly the same shape throughout the measurement period, but the step change can be set to have a value of 0 for all individuals by specifying a distribution with no variation. It would also be possible discard data on one side of the step change if desired. If the step change is required to occur at the same time for all individuals (for example, to represent an external event), a distribution with no variation may be specified for this time.

•	Two parameter values must be specified for each distribution involved which excludes distributions with other numbers of parameters, such as the exponential distribution.
