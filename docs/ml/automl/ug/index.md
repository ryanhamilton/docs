---
title: Automated machine-learning user guide | Machine Learning | Documentation for kdb+ and q
author: Conor McCarthy
description: How to call the top-level functions in the automated machine-learning repository. 
date: March 2020
keywords: machine learning, automated, ml, feature extraction, feature selection, data cleansing
---
# User interface



The top-level functions in the repository are:

`.automl.run`

: Run the automated machine -learning pipeline on user-defined data and target.

`.automl.new`

: Using a previously fit model and set of instructions derived from `.automl.run`, return predicted values for new tabular data.

Both of these functions are modifiable by a user to suit specific use cases and have been designed where possible to cover a wide range of functional options and to be extensible to a users needs. Details regarding all available modifications which can be made are outlined in the [advanced section](options.md).

The following examples and function descriptions outline the most basic implementations of each of the above functions for each of the use cases to which this platform can currently be applied. Namely non-timeseries-specific machine-learning examples and implementations making use of the [FRESH algorithm](../../toolkit/fresh).


## `.automl.run`

_Apply automated machine learning based on user provided data and target values_

Syntax: `.automl.run[tab;tgt;ftype;ptype;dict]`

Where

-   `tab` is unkeyed tabular data from which the models will be created
-   `tgt` is the target vector
-   `ftype` type of feature extraction being completed on the dataset as a symbol (`` `fresh/`normal``)
-   `ptype` type of problem, regression/class, as a symbol (` `reg/`class ``)
-   `dict` is one of `::` for default behavior, a kdb+ dictionary or path to a user-defined flat file for modifying default parameters.

returns the date and time at which the run was initiated.

The default setup saves the following items from an individual run:

1. The best model, saved as a HDF5 file, or ‘pickled’ byte object.
2. A saved report indicating the procedure taken and scores achieved.
3. A saved binary encoded dictionary denoting, the procedure to be taken for reproducing results, running on new data and outlining all important information relating to a run.
4. Results from each step of the pipeline published to console.

The following shows the execution of the function `.automl.run` in a regression task for a non-time series application. Data and implementation code is provided for other problem types however for brevity, output is displayed in full for one example only.

```q
// Non time-series example table
q)tab:([]asc 100?0t;100?1f;desc 100?0b;100?1f;asc 100?1f)
// Regression target
q)reg_tgt:asc 100?1f
// Feature extraction type
q)ftype:`normal
// Problem type
q)ptype:`reg
// Use default system parameters
q)dict:(::)
// Run example
q).automl.run[tab;reg_tgt;ftype;ptype;dict]

The following is a breakdown of information for each of the relevant columns in the dataset

  | count unique mean      std       min         max       type   
--| --------------------------------------------------------------
x1| 100   100    0.5241723 0.2885251 0.002184472 0.9830794 numeric
x3| 100   100    0.4722098 0.2894009 0.00283353  0.9996082 numeric
x4| 100   100    0.493481  0.2763346 0.01392076  0.9877844 numeric
x | 100   100    ::        ::        ::          ::        time   
x2| 100   2      ::        ::        ::          ::        boolean

Data preprocessing complete, starting feature creation

Feature creation and significance testing complete
Starting initial model selection - allow ample time for large datasets

Total features being passed to the models = 3

Scores for all models, using .ml.mse
GradientBoostingRegressor| 0.0001987841
RandomForestRegressor    | 0.0002360554
AdaBoostRegressor        | 0.0003896983
LinearRegression         | 0.0004134209
KNeighborsRegressor      | 0.0005237273
Lasso                    | 0.03033336
MLPRegressor             | 0.1881076
RegKeras                 | 0.4328872

Best scoring model = GradientBoostingRegressor
Score for validation predictions using best model = 0.0002752211

Feature impact calculated for features associated with GradientBoostingRegressor model
Plots saved in /outputs/2020.01.02/run_11.21.47.763/images/

Continuing to grid-search and final model fitting on holdout set

Best model fitting now complete - final score on test set = 0.0001017797

Saving down procedure report to /outputs/2020.01.02/run_11.21.47.763/report/
Saving down GradientBoostingRegressor model to /outputs/2020.01.02/run_11.21.47.763/models/
Saving down model parameters to /outputs/2020.01.02/run_11.21.47.763/config/
2020.01.02
11:21:47.763

// Example data for various problem types
q)bin_target:asc 100?0b
q)multi_target:desc 100?3
q)fresh_data:([]5000?100?0p;asc 5000?1f;5000?1f;desc 5000?10f;5000?0b)
// FRESH regression example
q).automl.run[fresh_data;reg_tgt;`fresh;`reg;::]
// non-time series/FRESH binary classification example
q).automl.run[tab;bin_target;`normal;`class;::]
```


## `.automl.new`

_Apply the workflow and fitted model associated with a specified run to new data_

Syntax: `.automl.new[tab;dt;tm]`

Where

-   `tab` is an unkeyed tabular dataset which has the same schema as the input data from the run specified in `fpath`
-   `dt` is the date of a run as a q date, or string representation i.e. `"yyyy.mm.dd"`
-   `tm` is the time of a run as a q time, or string representation either in the form `"hh:mm:ss.xxx"/"hh.mm.ss.xxx"`.

returns the target predictions for new data based on a previously fitted model and workflow.

```q
// New dataset
q)new_tab:([]asc 10?0t;10?1f;desc 10?0b;10?1f;asc 10?1f)
// string date/time input
q).automl.new[new_tab;2020.01.02;"11.21.47.763"]
0.1404663 0.255114 0.255114 0.2683779 0.2773197 0.487862 0.6659926 0.8547356 ..
// q date/time input
q).automl.new[new_tab;"2020.01.02";11:21:47.763]
0.1953181 0.449196 0.6708352 0.5842918 0.230593 0.4713597 0.1953181 0.0576498..
```

