---
title: "Machine Learning Toolbox"
author: "Ken Harmon"
date: "2019 August 08"
output:
  html_document:  
    keep_md: true
    code_folding: hide
    fig_height: 6
    fig_width: 12
    fig_align: 'center'
editor_options: 
  chunk_output_type: console
---

# {.tabset .tabset-fade}



## Background

https://campus.datacamp.com/courses/machine-learning-toolbox/regression-models-fitting-them-and-evaluating-their-performance?ex=1#

http://topepo.github.io/caret/index.html

## Regression models:

In-sample error



```r
# Fit a model to the mtcars data
data(mtcars)
model <- lm(mpg ~ hp, mtcars[1:20, ])

# Predict in-sample
predicted <- predict(model, mtcars[1:20, ], type = "response")

# Calculate RMSE
actual <- mtcars[1:20, "mpg"]
sqrt(mean((predicted - actual) ^ 2))
```

```
## [1] 3.172132
```

In-sample RMSE for linear regression on diamonds
As you saw in the video, included in the course is the diamonds dataset, which is a classic dataset from the ggplot2 package. The dataset contains physical attributes of diamonds as well as the price they sold for. One interesting modeling challenge is predicting diamond price based on their attributes using something like a linear regression.

Recall that to fit a linear regression, you use the lm() function in the following format:

mod <- lm(y ~ x, my_data)
To make predictions using mod on the original data, you call the predict() function:

pred <- predict(mod, my_data)


```r
# Fit lm model: model
data("diamonds")
model <- lm(price ~ ., diamonds)

# Predict on full data: p
p <- predict(model, diamonds, type = "response")

# Compute errors: error
actual <- diamonds$price
error <- p - actual

# Calculate RMSE
sqrt(mean((p - actual) ^ 2))
```

```
## [1] 1129.843
```

Out-of-sample error


```r
# Fit a model to the mtcars data
data(mtcars)
model <- lm(mpg ~ hp, mtcars[1:20, ])

# Predict in-sample
predicted <- predict(model, mtcars[21:32, ], type = "response")

# Calculate RMSE
actual <- mtcars[21:32, "mpg"]
sqrt(mean((predicted - actual) ^ 2))
```

```
## [1] 5.507236
```

Randomly order the data frame
One way you can take a train/test split of a dataset is to order the dataset randomly, then divide it into the two sets. This ensures that the training set and test set are both random samples and that any biases in the ordering of the dataset (e.g. if it had originally been ordered by price or size) are not retained in the samples we take for training and testing your models. You can think of this like shuffling a brand new deck of playing cards before dealing hands.

First, you set a random seed so that your work is reproducible and you get the same random split each time you run your script:

set.seed(42)
Next, you use the sample() function to shuffle the row indices of the diamonds dataset. You can later use these indices to reorder the dataset.

rows <- sample(nrow(diamonds))
Finally, you can use this random vector to reorder the diamonds dataset:

diamonds <- diamonds[rows, ]


```r
# Set seed
set.seed(42)

# Shuffle row indices: rows
rows <- sample(nrow(diamonds))

# Randomly order data
shuffled_diamonds <- diamonds[rows, ]
```

Try an 80/20 split
Now that your dataset is randomly ordered, you can split the first 80% of it into a training set, and the last 20% into a test set. You can do this by choosing a split point approximately 80% of the way through your data:

split <- round(nrow(mydata) * .80)
You can then use this point to break off the first 80% of the dataset as a training set:

mydata[1:split, ]
And then you can use that same point to determine the test set:

mydata[(split + 1):nrow(mydata), ]


```r
# Determine row to split on: split
split <- round(nrow(shuffled_diamonds) * .80)

# Create train
train <- shuffled_diamonds[1:split, ]

# Create test
test <- shuffled_diamonds[(split + 1):nrow(diamonds), ]
```

Predict on test set
Now that you have a randomly split training set and test set, you can use the lm() function as you did in the first exercise to fit a model to your training set, rather than the entire dataset. Recall that you can use the formula interface to the linear regression function to fit a model with a specified target variable using all other variables in the dataset as predictors:

mod <- lm(y ~ ., training_data)
You can use the predict() function to make predictions from that model on new data. The new dataset must have all of the columns from the training data, but they can be in a different order with different values. Here, rather than re-predicting on the training set, you can predict on the test set, which you did not use for training the model. This will allow you to determine the out-of-sample error for the model in the next exercise:

p <- predict(model, new_data)


```r
# Fit lm model on train: model
model <- lm(price ~ ., train)

# Predict on test: p
p <- predict(model, test)
```

Calculate test set RMSE by hand
Now that you have predictions on the test set, you can use these predictions to calculate an error metric (in this case RMSE) on the test set and see how the model performs out-of-sample, rather than in-sample as you did in the first exercise. You first do this by calculating the errors between the predicted diamond prices and the actual diamond prices by subtracting the predictions from the actual values.

Once you have an error vector, calculating RMSE is as simple as squaring it, taking the mean, then taking the square root:

sqrt(mean(error^2))


```r
# Compute errors: error
error <- p - test[["price"]]

# Calculate RMSE
sqrt(mean(error^2))
```

```
## [1] 1137.466
```

```r
# Compute errors: error
error <- p - test$price

# Calculate RMSE
sqrt(mean(error^2))
```

```
## [1] 1137.466
```

10-fold cross-validation
As you saw in the video, a better approach to validating models is to use multiple systematic test sets, rather than a single random train/test split. Fortunately, the caret package makes this very easy to do:

model <- train(y ~ ., my_data)
caret supports many types of cross-validation, and you can specify which type of cross-validation and the number of cross-validation folds with the trainControl() function, which you pass to the trControl argument in train():

model <- train(
  y ~ ., 
  my_data,
  method = "lm",
  trControl = trainControl(
    method = "cv", 
    number = 10,
    verboseIter = TRUE
  )
)
It's important to note that you pass the method for modeling to the main train() function and the method for cross-validation to the trainControl() function.


```r
# Fit lm model using 10-fold CV: model
model <- train(price ~ .,diamonds, method = "lm", trControl = trainControl(method = "cv",number = 10, verboseIter = TRUE))
```

```
## + Fold01: intercept=TRUE 
## - Fold01: intercept=TRUE 
## + Fold02: intercept=TRUE 
## - Fold02: intercept=TRUE 
## + Fold03: intercept=TRUE 
## - Fold03: intercept=TRUE 
## + Fold04: intercept=TRUE 
## - Fold04: intercept=TRUE 
## + Fold05: intercept=TRUE 
## - Fold05: intercept=TRUE 
## + Fold06: intercept=TRUE 
## - Fold06: intercept=TRUE 
## + Fold07: intercept=TRUE 
## - Fold07: intercept=TRUE 
## + Fold08: intercept=TRUE 
## - Fold08: intercept=TRUE 
## + Fold09: intercept=TRUE 
## - Fold09: intercept=TRUE 
## + Fold10: intercept=TRUE 
## - Fold10: intercept=TRUE 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print model to console
model
```

```
## Linear Regression 
## 
## 53940 samples
##     9 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 48547, 48546, 48546, 48547, 48545, 48547, ... 
## Resampling results:
## 
##   RMSE      Rsquared   MAE     
##   1131.015  0.9196398  740.6117
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```

5-fold cross-validation
In this course, you will use a wide variety of datasets to explore the full flexibility of the caret package. Here, you will use the famous Boston housing dataset, where the goal is to predict median home values in various Boston suburbs.

You can use exactly the same code as in the previous exercise, but change the dataset used by the model:

model <- train(
  medv ~ ., 
  Boston, # <- new!
  method = "lm",
  trControl = trainControl(
    method = "cv", 
    number = 10,
    verboseIter = TRUE
  )
)
Next, you can reduce the number of cross-validation folds from 10 to 5 using the number argument to the trainControl() argument:

trControl = trainControl(
  method = "cv", 
  number = 5,
  verboseIter = TRUE
)


```r
# Fit lm model using 10-fold CV: model
model <- train(medv ~ .,Boston, method = "lm", trControl = trainControl(method = "cv",number = 5, verboseIter = TRUE))
```

```
## + Fold1: intercept=TRUE 
## - Fold1: intercept=TRUE 
## + Fold2: intercept=TRUE 
## - Fold2: intercept=TRUE 
## + Fold3: intercept=TRUE 
## - Fold3: intercept=TRUE 
## + Fold4: intercept=TRUE 
## - Fold4: intercept=TRUE 
## + Fold5: intercept=TRUE 
## - Fold5: intercept=TRUE 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print model to console
model
```

```
## Linear Regression 
## 
## 506 samples
##  13 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 405, 405, 406, 403, 405 
## Resampling results:
## 
##   RMSE      Rsquared   MAE     
##   4.860247  0.7209221  3.398114
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```

5 x 5-fold cross-validation
You can do more than just one iteration of cross-validation. Repeated cross-validation gives you a better estimate of the test-set error. You can also repeat the entire cross-validation procedure. This takes longer, but gives you many more out-of-sample datasets to look at and much more precise assessments of how well the model performs.

One of the awesome things about the train() function in caret is how easy it is to run very different models or methods of cross-validation just by tweaking a few simple arguments to the function call. For example, you could repeat your entire cross-validation procedure 5 times for greater confidence in your estimates of the model's out-of-sample accuracy, e.g.:

trControl = trainControl(
  method = "repeatedcv", 
  number = 5,
  repeats = 5, 
  verboseIter = TRUE
)


```r
# Fit lm model using 5 x 5-fold CV: model
model <- train(
  medv ~ ., 
  Boston,
  method = "lm",
  trControl = trainControl(
    method = "repeatedcv", 
    number = 5,
    repeats = 5, 
    verboseIter = TRUE
  )
)
```

```
## + Fold1.Rep1: intercept=TRUE 
## - Fold1.Rep1: intercept=TRUE 
## + Fold2.Rep1: intercept=TRUE 
## - Fold2.Rep1: intercept=TRUE 
## + Fold3.Rep1: intercept=TRUE 
## - Fold3.Rep1: intercept=TRUE 
## + Fold4.Rep1: intercept=TRUE 
## - Fold4.Rep1: intercept=TRUE 
## + Fold5.Rep1: intercept=TRUE 
## - Fold5.Rep1: intercept=TRUE 
## + Fold1.Rep2: intercept=TRUE 
## - Fold1.Rep2: intercept=TRUE 
## + Fold2.Rep2: intercept=TRUE 
## - Fold2.Rep2: intercept=TRUE 
## + Fold3.Rep2: intercept=TRUE 
## - Fold3.Rep2: intercept=TRUE 
## + Fold4.Rep2: intercept=TRUE 
## - Fold4.Rep2: intercept=TRUE 
## + Fold5.Rep2: intercept=TRUE 
## - Fold5.Rep2: intercept=TRUE 
## + Fold1.Rep3: intercept=TRUE 
## - Fold1.Rep3: intercept=TRUE 
## + Fold2.Rep3: intercept=TRUE 
## - Fold2.Rep3: intercept=TRUE 
## + Fold3.Rep3: intercept=TRUE 
## - Fold3.Rep3: intercept=TRUE 
## + Fold4.Rep3: intercept=TRUE 
## - Fold4.Rep3: intercept=TRUE 
## + Fold5.Rep3: intercept=TRUE 
## - Fold5.Rep3: intercept=TRUE 
## + Fold1.Rep4: intercept=TRUE 
## - Fold1.Rep4: intercept=TRUE 
## + Fold2.Rep4: intercept=TRUE 
## - Fold2.Rep4: intercept=TRUE 
## + Fold3.Rep4: intercept=TRUE 
## - Fold3.Rep4: intercept=TRUE 
## + Fold4.Rep4: intercept=TRUE 
## - Fold4.Rep4: intercept=TRUE 
## + Fold5.Rep4: intercept=TRUE 
## - Fold5.Rep4: intercept=TRUE 
## + Fold1.Rep5: intercept=TRUE 
## - Fold1.Rep5: intercept=TRUE 
## + Fold2.Rep5: intercept=TRUE 
## - Fold2.Rep5: intercept=TRUE 
## + Fold3.Rep5: intercept=TRUE 
## - Fold3.Rep5: intercept=TRUE 
## + Fold4.Rep5: intercept=TRUE 
## - Fold4.Rep5: intercept=TRUE 
## + Fold5.Rep5: intercept=TRUE 
## - Fold5.Rep5: intercept=TRUE 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print model to console
model
```

```
## Linear Regression 
## 
## 506 samples
##  13 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold, repeated 5 times) 
## Summary of sample sizes: 405, 406, 405, 403, 405, 405, ... 
## Resampling results:
## 
##   RMSE      Rsquared   MAE     
##   4.845724  0.7277269  3.402735
## 
## Tuning parameter 'intercept' was held constant at a value of TRUE
```

Making predictions on new data
Finally, the model you fit with the train() function has the exact same predict() interface as the linear regression models you fit earlier in this chapter.

After fitting a model with train(), you can simply call predict() with new data, e.g:

predict(my_model, new_data)


```r
# Predict on full Boston dataset
predict(model, Boston)
```

```
##          1          2          3          4          5          6 
## 30.0038434 25.0255624 30.5675967 28.6070365 27.9435242 25.2562845 
##          7          8          9         10         11         12 
## 23.0018083 19.5359884 11.5236369 18.9202621 18.9994965 21.5867957 
##         13         14         15         16         17         18 
## 20.9065215 19.5529028 19.2834821 19.2974832 20.5275098 16.9114013 
##         19         20         21         22         23         24 
## 16.1780111 18.4061360 12.5238575 17.6710367 15.8328813 13.8062853 
##         25         26         27         28         29         30 
## 15.6783383 13.3866856 15.4639765 14.7084743 19.5473729 20.8764282 
##         31         32         33         34         35         36 
## 11.4551176 18.0592329  8.8110574 14.2827581 13.7067589 23.8146353 
##         37         38         39         40         41         42 
## 22.3419371 23.1089114 22.9150261 31.3576257 34.2151023 28.0205641 
##         43         44         45         46         47         48 
## 25.2038663 24.6097927 22.9414918 22.0966982 20.4232003 18.0365509 
##         49         50         51         52         53         54 
##  9.1065538 17.2060775 21.2815254 23.9722228 27.6558508 24.0490181 
##         55         56         57         58         59         60 
## 15.3618477 31.1526495 24.8568698 33.1091981 21.7753799 21.0849356 
##         61         62         63         64         65         66 
## 17.8725804 18.5111021 23.9874286 22.5540887 23.3730864 30.3614836 
##         67         68         69         70         71         72 
## 25.5305651 21.1133856 17.4215379 20.7848363 25.2014886 21.7426577 
##         73         74         75         76         77         78 
## 24.5574496 24.0429571 25.5049972 23.9669302 22.9454540 23.3569982 
##         79         80         81         82         83         84 
## 21.2619827 22.4281737 28.4057697 26.9948609 26.0357630 25.0587348 
##         85         86         87         88         89         90 
## 24.7845667 27.7904920 22.1685342 25.8927642 30.6746183 30.8311062 
##         91         92         93         94         95         96 
## 27.1190194 27.4126673 28.9412276 29.0810555 27.0397736 28.6245995 
##         97         98         99        100        101        102 
## 24.7274498 35.7815952 35.1145459 32.2510280 24.5802202 25.5941347 
##        103        104        105        106        107        108 
## 19.7901368 20.3116713 21.4348259 18.5399401 17.1875599 20.7504903 
##        109        110        111        112        113        114 
## 22.6482911 19.7720367 20.6496586 26.5258674 20.7732364 20.7154831 
##        115        116        117        118        119        120 
## 25.1720888 20.4302559 23.3772463 23.6904326 20.3357836 20.7918087 
##        121        122        123        124        125        126 
## 21.9163207 22.4710778 20.5573856 16.3666198 20.5609982 22.4817845 
##        127        128        129        130        131        132 
## 14.6170663 15.1787668 18.9386859 14.0557329 20.0352740 19.4101340 
##        133        134        135        136        137        138 
## 20.0619157 15.7580767 13.2564524 17.2627773 15.8784188 19.3616395 
##        139        140        141        142        143        144 
## 13.8148390 16.4488147 13.5714193  3.9888551 14.5949548 12.1488148 
##        145        146        147        148        149        150 
##  8.7282236 12.0358534 15.8208206  8.5149902  9.7184414 14.8045137 
##        151        152        153        154        155        156 
## 20.8385815 18.3010117 20.1228256 17.2860189 22.3660023 20.1037592 
##        157        158        159        160        161        162 
## 13.6212589 33.2598270 29.0301727 25.5675277 32.7082767 36.7746701 
##        163        164        165        166        167        168 
## 40.5576584 41.8472817 24.7886738 25.3788924 37.2034745 23.0874875 
##        169        170        171        172        173        174 
## 26.4027396 26.6538211 22.5551466 24.2908281 22.9765722 29.0719431 
##        175        176        177        178        179        180 
## 26.5219434 30.7220906 25.6166931 29.1374098 31.4357197 32.9223157 
##        181        182        183        184        185        186 
## 34.7244046 27.7655211 33.8878732 30.9923804 22.7182001 24.7664781 
##        187        188        189        190        191        192 
## 35.8849723 33.4247672 32.4119915 34.5150995 30.7610949 30.2893414 
##        193        194        195        196        197        198 
## 32.9191871 32.1126077 31.5587100 40.8455572 36.1277008 32.6692081 
##        199        200        201        202        203        204 
## 34.7046912 30.0934516 30.6439391 29.2871950 37.0714839 42.0319312 
##        205        206        207        208        209        210 
## 43.1894984 22.6903480 23.6828471 17.8544721 23.4942899 17.0058772 
##        211        212        213        214        215        216 
## 22.3925110 17.0604275 22.7389292 25.2194255 11.1191674 24.5104915 
##        217        218        219        220        221        222 
## 26.6033477 28.3551871 24.9152546 29.6865277 33.1841975 23.7745666 
##        223        224        225        226        227        228 
## 32.1405196 29.7458199 38.3710245 39.8146187 37.5860575 32.3995325 
##        229        230        231        232        233        234 
## 35.4566524 31.2341151 24.4844923 33.2883729 38.0481048 37.1632863 
##        235        236        237        238        239        240 
## 31.7138352 25.2670557 30.1001074 32.7198716 28.4271706 28.4294068 
##        241        242        243        244        245        246 
## 27.2937594 23.7426248 24.1200789 27.4020841 16.3285756 13.3989126 
##        247        248        249        250        251        252 
## 20.0163878 19.8618443 21.2883131 24.0798915 24.2063355 25.0421582 
##        253        254        255        256        257        258 
## 24.9196401 29.9456337 23.9722832 21.6958089 37.5110924 43.3023904 
##        259        260        261        262        263        264 
## 36.4836142 34.9898859 34.8121151 37.1663133 40.9892850 34.4463409 
##        265        266        267        268        269        270 
## 35.8339755 28.2457430 31.2267359 40.8395575 39.3179239 25.7081791 
##        271        272        273        274        275        276 
## 22.3029553 27.2034097 28.5116947 35.4767660 36.1063916 33.7966827 
##        277        278        279        280        281        282 
## 35.6108586 34.8399338 30.3519266 35.3098070 38.7975697 34.3312319 
##        283        284        285        286        287        288 
## 40.3396307 44.6730834 31.5968909 27.3565923 20.1017415 27.0420667 
##        289        290        291        292        293        294 
## 27.2136458 26.9139584 33.4356331 34.4034963 31.8333982 25.8178324 
##        295        296        297        298        299        300 
## 24.4298235 28.4576434 27.3626700 19.5392876 29.1130984 31.9105461 
##        301        302        303        304        305        306 
## 30.7715945 28.9427587 28.8819102 32.7988723 33.2090546 30.7683179 
##        307        308        309        310        311        312 
## 35.5622686 32.7090512 28.6424424 23.5896583 18.5426690 26.8788984 
##        313        314        315        316        317        318 
## 23.2813398 25.5458025 25.4812006 20.5390990 17.6157257 18.3758169 
##        319        320        321        322        323        324 
## 24.2907028 21.3252904 24.8868224 24.8693728 22.8695245 19.4512379 
##        325        326        327        328        329        330 
## 25.1178340 24.6678691 23.6807618 19.3408962 21.1741811 24.2524907 
##        331        332        333        334        335        336 
## 21.5926089 19.9844661 23.3388800 22.1406069 21.5550993 20.6187291 
##        337        338        339        340        341        342 
## 20.1609718 19.2849039 22.1667232 21.2496577 21.4293931 30.3278880 
##        343        344        345        346        347        348 
## 22.0473498 27.7064791 28.5479412 16.5450112 14.7835964 25.2738008 
##        349        350        351        352        353        354 
## 27.5420512 22.1483756 20.4594409 20.5460542 16.8806383 25.4025351 
##        355        356        357        358        359        360 
## 14.3248663 16.5948846 19.6370469 22.7180661 22.2021889 19.2054806 
##        361        362        363        364        365        366 
## 22.6661611 18.9319262 18.2284680 20.2315081 37.4944739 14.2819073 
##        367        368        369        370        371        372 
## 15.5428625 10.8316232 23.8007290 32.6440736 34.6068404 24.9433133 
##        373        374        375        376        377        378 
## 25.9998091  6.1263250  0.7777981 25.3071306 17.7406106 20.2327441 
##        379        380        381        382        383        384 
## 15.8333130 16.8351259 14.3699483 18.4768283 13.4276828 13.0617751 
##        385        386        387        388        389        390 
##  3.2791812  8.0602217  6.1284220  5.6186481  6.4519857 14.2076474 
##        391        392        393        394        395        396 
## 17.2122518 17.2988727  9.8911664 20.2212419 17.9418118 20.3044578 
##        397        398        399        400        401        402 
## 19.2955908 16.3363278  6.5516232 10.8901678 11.8814587 17.8117451 
##        403        404        405        406        407        408 
## 18.2612659 12.9794878  7.3781636  8.2111586  8.0662619 19.9829479 
##        409        410        411        412        413        414 
## 13.7075637 19.8526845 15.2230830 16.9607198  1.7185181 11.8057839 
##        415        416        417        418        419        420 
## -4.2813107  9.5837674 13.3666081  6.8956236  6.1477985 14.6066179 
##        421        422        423        424        425        426 
## 19.6000267 18.1242748 18.5217713 13.1752861 14.6261762  9.9237498 
##        427        428        429        430        431        432 
## 16.3459065 14.0751943 14.2575624 13.0423479 18.1595569 18.6955435 
##        433        434        435        436        437        438 
## 21.5272830 17.0314186 15.9609044 13.3614161 14.5207938  8.8197601 
##        439        440        441        442        443        444 
##  4.8675110 13.0659131 12.7060970 17.2955806 18.7404850 18.0590103 
##        445        446        447        448        449        450 
## 11.5147468 11.9740036 17.6834462 18.1269524 17.5183465 17.2274251 
##        451        452        453        454        455        456 
## 16.5227163 19.4129110 18.5821524 22.4894479 15.2800013 15.8208934 
##        457        458        459        460        461        462 
## 12.6872558 12.8763379 17.1866853 18.5124761 19.0486053 20.1720893 
##        463        464        465        466        467        468 
## 19.7740732 22.4294077 20.3191185 17.8861625 14.3747852 16.9477685 
##        469        470        471        472        473        474 
## 16.9840576 18.5883840 20.1671944 22.9771803 22.4558073 25.5782463 
##        475        476        477        478        479        480 
## 16.3914763 16.1114628 20.5348160 11.5427274 19.2049630 21.8627639 
##        481        482        483        484        485        486 
## 23.4687887 27.0988732 28.5699430 21.0839878 19.4551620 22.2222591 
##        487        488        489        490        491        492 
## 19.6559196 21.3253610 11.8558372  8.2238669  3.6639967 13.7590854 
##        493        494        495        496        497        498 
## 15.9311855 20.6266205 20.6124941 16.8854196 14.0132079 19.1085414 
##        499        500        501        502        503        504 
## 21.2980517 18.4549884 20.4687085 23.5333405 22.3757189 27.6274261 
##        505        506 
## 26.1279668 22.3442123
```

## Classification Models

Try a 60/40 split
As you saw in the video, you'll be working with the Sonar dataset in this chapter, using a 60% training set and a 40% test set. We'll practice making a train/test split one more time, just to be sure you have the hang of it. Recall that you can use the sample() function to get a random permutation of the row indices in a dataset, to use when making train/test splits, e.g.:

n_obs <- nrow(my_data)
permuted_rows <- sample(n_obs)
And then use those row indices to randomly reorder the dataset, e.g.:

my_data <- my_data[permuted_rows, ]
Once your dataset is randomly ordered, you can split off the first 60% as a training set and the last 40% as a test set.


```r
data(Sonar)

# Get the number of observations
n_obs <- nrow(Sonar)

# Shuffle row indices: permuted_rows
permuted_rows <- sample(n_obs)

# Randomly order data: Sonar
Sonar_shuffled <- Sonar[permuted_rows, ]

# Identify row to split on: split
split <- round(n_obs * 0.6)

# Create train
train <- Sonar_shuffled[1:split, ]

# Create test
test <- Sonar_shuffled[(split + 1):n_obs, ]

nrow(train)/nrow(Sonar)
```

```
## [1] 0.6009615
```

Fit a logistic regression model
Once you have your random training and test sets you can fit a logistic regression model to your training set using the glm() function. glm() is a more advanced version of lm() that allows for more varied types of regression models, aside from plain vanilla ordinary least squares regression.

Be sure to pass the argument family = "binomial" to glm() to specify that you want to do logistic (rather than linear) regression. For example:

glm(Target ~ ., family = "binomial", dataset)
Don't worry about warnings like glm.fit: algorithm did not converge or glm.fit: fitted probabilities numerically 0 or 1 occurred. These are common on smaller datasets and usually don't cause any issues. They typically mean your dataset is perfectly separable, which can cause problems for the math behind the model, but R's glm() function is almost always robust enough to handle this case with no problems.

Once you have a glm() model fit to your dataset, you can predict the outcome (e.g. rock or mine) on the test set using the predict() function with the argument type = "response":

predict(my_model, test, type = "response")


```r
# Fit glm model: model
model <- glm(Class ~ ., family = "binomial", train)

# Predict on test: p
p <- predict(model, test, type = "response")
```

Calculate a confusion matrix
As you saw in the video, a confusion matrix is a very useful tool for calibrating the output of a model and examining all possible outcomes of your predictions (true positive, true negative, false positive, false negative).

Before you make your confusion matrix, you need to "cut" your predicted probabilities at a given threshold to turn probabilities into a factor of class predictions. Combine ifelse() with factor() as follows:

pos_or_neg <- ifelse(probability_prediction > threshold, positive_class, negative_class)
p_class <- factor(pos_or_neg, levels = levels(test_values))
confusionMatrix() in caret improves on table() from base R by adding lots of useful ancillary statistics in addition to the base rates in the table. You can calculate the confusion matrix (and the associated statistics) using the predicted outcomes as well as the actual outcomes, e.g.:

confusionMatrix(p_class, test_values)


```r
# If p exceeds threshold of 0.5, M else R: m_or_r
m_or_r <- ifelse(p > .5, "M", "R")

# Convert to factor: p_class
p_class <- factor(m_or_r, levels = levels(test[["Class"]]))

# Create confusion matrix
confusionMatrix(p_class, test$Class)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction  M  R
##          M  7 31
##          R 32 13
##                                           
##                Accuracy : 0.241           
##                  95% CI : (0.1538, 0.3473)
##     No Information Rate : 0.5301          
##     P-Value [Acc > NIR] : 1               
##                                           
##                   Kappa : -0.5258         
##                                           
##  Mcnemar's Test P-Value : 1               
##                                           
##             Sensitivity : 0.17949         
##             Specificity : 0.29545         
##          Pos Pred Value : 0.18421         
##          Neg Pred Value : 0.28889         
##              Prevalence : 0.46988         
##          Detection Rate : 0.08434         
##    Detection Prevalence : 0.45783         
##       Balanced Accuracy : 0.23747         
##                                           
##        'Positive' Class : M               
## 
```

Try another threshold
In the previous exercises, you used a threshold of 0.50 to cut your predicted probabilities to make class predictions (rock vs mine). However, this classification threshold does not always align with the goals for a given modeling problem.

For example, pretend you want to identify the objects you are really certain are mines. In this case, you might want to use a probability threshold of 0.90 to get fewer predicted mines, but with greater confidence in each prediction.

The code pattern for cutting probabilities into predicted classes, then calculating a confusion matrix, was shown in Exercise 7 of this chapter.


```r
# If p exceeds threshold of 0.9, M else R: m_or_r
m_or_r <- ifelse(p > .9, "M", "R")

# Convert to factor: p_class
p_class <- factor(m_or_r, levels = levels(test[["Class"]]))

# Create confusion matrix
confusionMatrix(p_class, test$Class)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction  M  R
##          M  7 31
##          R 32 13
##                                           
##                Accuracy : 0.241           
##                  95% CI : (0.1538, 0.3473)
##     No Information Rate : 0.5301          
##     P-Value [Acc > NIR] : 1               
##                                           
##                   Kappa : -0.5258         
##                                           
##  Mcnemar's Test P-Value : 1               
##                                           
##             Sensitivity : 0.17949         
##             Specificity : 0.29545         
##          Pos Pred Value : 0.18421         
##          Neg Pred Value : 0.28889         
##              Prevalence : 0.46988         
##          Detection Rate : 0.08434         
##    Detection Prevalence : 0.45783         
##       Balanced Accuracy : 0.23747         
##                                           
##        'Positive' Class : M               
## 
```


```r
# If p exceeds threshold of 0.9, M else R: m_or_r
m_or_r <- ifelse(p > .1, "M", "R")

# Convert to factor: p_class
p_class <- factor(m_or_r, levels = levels(test[["Class"]]))

# Create confusion matrix
confusionMatrix(p_class, test$Class)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction  M  R
##          M  7 31
##          R 32 13
##                                           
##                Accuracy : 0.241           
##                  95% CI : (0.1538, 0.3473)
##     No Information Rate : 0.5301          
##     P-Value [Acc > NIR] : 1               
##                                           
##                   Kappa : -0.5258         
##                                           
##  Mcnemar's Test P-Value : 1               
##                                           
##             Sensitivity : 0.17949         
##             Specificity : 0.29545         
##          Pos Pred Value : 0.18421         
##          Neg Pred Value : 0.28889         
##              Prevalence : 0.46988         
##          Detection Rate : 0.08434         
##    Detection Prevalence : 0.45783         
##       Balanced Accuracy : 0.23747         
##                                           
##        'Positive' Class : M               
## 
```

Plot an ROC curve
As you saw in the video, an ROC curve is a really useful shortcut for summarizing the performance of a classifier over all possible thresholds. This saves you a lot of tedious work computing class predictions for many different thresholds and examining the confusion matrix for each.

My favorite package for computing ROC curves is caTools, which contains a function called colAUC(). This function is very user-friendly and can actually calculate ROC curves for multiple predictors at once. In this case, you only need to calculate the ROC curve for one predictor, e.g.:

colAUC(predicted_probabilities, actual, plotROC = TRUE)
The function will return a score called AUC (more on that later) and the plotROC = TRUE argument will return the plot of the ROC curve for visual inspection.


```r
# Predict on test: p
p <- predict(model, test, type = "response")

# Make ROC curve
colAUC(p, test$Class, plotROC = TRUE)
```

![](Machine-Learning-Toolbox_files/figure-html/PROCC-1.png)<!-- -->

```
##              [,1]
## M vs. R 0.8161422
```

Customizing trainControl
As you saw in the video, area under the ROC curve is a very useful, single-number summary of a model's ability to discriminate the positive from the negative class (e.g. mines from rocks). An AUC of 0.5 is no better than random guessing, an AUC of 1.0 is a perfectly predictive model, and an AUC of 0.0 is perfectly anti-predictive (which rarely happens).

This is often a much more useful metric than simply ranking models by their accuracy at a set threshold, as different models might require different calibration steps (looking at a confusion matrix at each step) to find the optimal classification threshold for that model.

You can use the trainControl() function in caret to use AUC (instead of acccuracy), to tune the parameters of your models. The twoClassSummary() convenience function allows you to do this easily.

When using twoClassSummary(), be sure to always include the argument classProbs = TRUE or your model will throw an error! (You cannot calculate AUC with just class predictions. You need to have class probabilities as well.)


```r
# Create trainControl object: myControl
myControl <- trainControl(
  method = "cv",
  number = 10,
  summaryFunction = twoClassSummary,
  classProbs = TRUE, # IMPORTANT!
  verboseIter = TRUE
)
```

Using custom trainControl
Now that you have a custom trainControl object, it's easy to fit caret models that use AUC rather than accuracy to tune and evaluate the model. You can just pass your custom trainControl object to the train() function via the trControl argument, e.g.:

train(<standard arguments here>, trControl = myControl)
This syntax gives you a convenient way to store a lot of custom modeling parameters and then use them across multiple different calls to train(). You will make extensive use of this trick in Chapter 5.


```r
# Train glm with custom trainControl: model
model <- train(
  Class ~ ., 
  Sonar, 
  method = "glm",
  trControl = myControl
)
```

```
## + Fold01: parameter=none 
## - Fold01: parameter=none 
## + Fold02: parameter=none 
## - Fold02: parameter=none 
## + Fold03: parameter=none 
## - Fold03: parameter=none 
## + Fold04: parameter=none 
## - Fold04: parameter=none 
## + Fold05: parameter=none 
## - Fold05: parameter=none 
## + Fold06: parameter=none 
## - Fold06: parameter=none 
## + Fold07: parameter=none 
## - Fold07: parameter=none 
## + Fold08: parameter=none 
## - Fold08: parameter=none 
## + Fold09: parameter=none 
## - Fold09: parameter=none 
## + Fold10: parameter=none 
## - Fold10: parameter=none 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print model to console
model
```

```
## Generalized Linear Model 
## 
## 208 samples
##  60 predictor
##   2 classes: 'M', 'R' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 187, 187, 188, 187, 188, 188, ... 
## Resampling results:
## 
##   ROC        Sens   Spec
##   0.7255051  0.775  0.66
```

## Tuning model

Fit a random forest
As you saw in the video, random forest models are much more flexible than linear models, and can model complicated nonlinear effects as well as automatically capture interactions between variables. They tend to give very good results on real world data, so let's try one out on the wine quality dataset, where the goal is to predict the human-evaluated quality of a batch of wine, given some of the machine-measured chemical and physical properties of that batch.

Fitting a random forest model is exactly the same as fitting a generalized linear regression model, as you did in the previous chapter. You simply change the method argument in the train function to be "ranger". The ranger package is a rewrite of R's classic randomForest package and fits models much faster, but gives almost exactly the same results. We suggest that all beginners use the ranger package for random forest modeling.


```r
wine <- read.csv("wine.csv")


# Fit random forest: model
model <- train(
  quality ~ .,
  tuneLength = 1,
  data = wine, 
  method = "ranger",
  trControl = trainControl(
    method = "cv", 
    number = 5, 
    verboseIter = TRUE
  )
)
```

```
## + Fold1: mtry=3, min.node.size=5, splitrule=variance 
## - Fold1: mtry=3, min.node.size=5, splitrule=variance 
## + Fold1: mtry=3, min.node.size=5, splitrule=extratrees 
## - Fold1: mtry=3, min.node.size=5, splitrule=extratrees 
## + Fold2: mtry=3, min.node.size=5, splitrule=variance 
## - Fold2: mtry=3, min.node.size=5, splitrule=variance 
## + Fold2: mtry=3, min.node.size=5, splitrule=extratrees 
## - Fold2: mtry=3, min.node.size=5, splitrule=extratrees 
## + Fold3: mtry=3, min.node.size=5, splitrule=variance 
## - Fold3: mtry=3, min.node.size=5, splitrule=variance 
## + Fold3: mtry=3, min.node.size=5, splitrule=extratrees 
## - Fold3: mtry=3, min.node.size=5, splitrule=extratrees 
## + Fold4: mtry=3, min.node.size=5, splitrule=variance 
## - Fold4: mtry=3, min.node.size=5, splitrule=variance 
## + Fold4: mtry=3, min.node.size=5, splitrule=extratrees 
## - Fold4: mtry=3, min.node.size=5, splitrule=extratrees 
## + Fold5: mtry=3, min.node.size=5, splitrule=variance 
## - Fold5: mtry=3, min.node.size=5, splitrule=variance 
## + Fold5: mtry=3, min.node.size=5, splitrule=extratrees 
## - Fold5: mtry=3, min.node.size=5, splitrule=extratrees 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 3, splitrule = variance, min.node.size = 5 on full training set
```

```r
# Print model to console
model
```

```
## Random Forest 
## 
## 100 samples
##  13 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 81, 80, 79, 80, 80 
## Resampling results across tuning parameters:
## 
##   splitrule   RMSE       Rsquared   MAE      
##   variance    0.6484721  0.3367117  0.4909785
##   extratrees  0.6855287  0.2620637  0.5086255
## 
## Tuning parameter 'mtry' was held constant at a value of 3
## Tuning
##  parameter 'min.node.size' was held constant at a value of 5
## RMSE was used to select the optimal model using the smallest value.
## The final values used for the model were mtry = 3, splitrule =
##  variance and min.node.size = 5.
```

Try a longer tune length
Recall from the video that random forest models have a primary tuning parameter of mtry, which controls how many variables are exposed to the splitting search routine at each split. For example, suppose that a tree has a total of 10 splits and mtry = 2. This means that there are 10 samples of 2 predictors each time a split is evaluated.

Use a larger tuning grid this time, but stick to the defaults provided by the train() function. Try a tuneLength of 3, rather than 1, to explore some more potential models, and plot the resulting model using the plot function.


```r
# Fit random forest: model
model <- train(
  quality~.,
  tuneLength = 3,
  data = wine, 
  method = "ranger",
  trControl = trainControl(
    method = "cv", 
    number = 5, 
    verboseIter = TRUE
  )
)
```

```
## + Fold1: mtry= 2, min.node.size=5, splitrule=variance 
## - Fold1: mtry= 2, min.node.size=5, splitrule=variance 
## + Fold1: mtry= 7, min.node.size=5, splitrule=variance 
## - Fold1: mtry= 7, min.node.size=5, splitrule=variance 
## + Fold1: mtry=13, min.node.size=5, splitrule=variance 
## - Fold1: mtry=13, min.node.size=5, splitrule=variance 
## + Fold1: mtry= 2, min.node.size=5, splitrule=extratrees 
## - Fold1: mtry= 2, min.node.size=5, splitrule=extratrees 
## + Fold1: mtry= 7, min.node.size=5, splitrule=extratrees 
## - Fold1: mtry= 7, min.node.size=5, splitrule=extratrees 
## + Fold1: mtry=13, min.node.size=5, splitrule=extratrees 
## - Fold1: mtry=13, min.node.size=5, splitrule=extratrees 
## + Fold2: mtry= 2, min.node.size=5, splitrule=variance 
## - Fold2: mtry= 2, min.node.size=5, splitrule=variance 
## + Fold2: mtry= 7, min.node.size=5, splitrule=variance 
## - Fold2: mtry= 7, min.node.size=5, splitrule=variance 
## + Fold2: mtry=13, min.node.size=5, splitrule=variance 
## - Fold2: mtry=13, min.node.size=5, splitrule=variance 
## + Fold2: mtry= 2, min.node.size=5, splitrule=extratrees 
## - Fold2: mtry= 2, min.node.size=5, splitrule=extratrees 
## + Fold2: mtry= 7, min.node.size=5, splitrule=extratrees 
## - Fold2: mtry= 7, min.node.size=5, splitrule=extratrees 
## + Fold2: mtry=13, min.node.size=5, splitrule=extratrees 
## - Fold2: mtry=13, min.node.size=5, splitrule=extratrees 
## + Fold3: mtry= 2, min.node.size=5, splitrule=variance 
## - Fold3: mtry= 2, min.node.size=5, splitrule=variance 
## + Fold3: mtry= 7, min.node.size=5, splitrule=variance 
## - Fold3: mtry= 7, min.node.size=5, splitrule=variance 
## + Fold3: mtry=13, min.node.size=5, splitrule=variance 
## - Fold3: mtry=13, min.node.size=5, splitrule=variance 
## + Fold3: mtry= 2, min.node.size=5, splitrule=extratrees 
## - Fold3: mtry= 2, min.node.size=5, splitrule=extratrees 
## + Fold3: mtry= 7, min.node.size=5, splitrule=extratrees 
## - Fold3: mtry= 7, min.node.size=5, splitrule=extratrees 
## + Fold3: mtry=13, min.node.size=5, splitrule=extratrees 
## - Fold3: mtry=13, min.node.size=5, splitrule=extratrees 
## + Fold4: mtry= 2, min.node.size=5, splitrule=variance 
## - Fold4: mtry= 2, min.node.size=5, splitrule=variance 
## + Fold4: mtry= 7, min.node.size=5, splitrule=variance 
## - Fold4: mtry= 7, min.node.size=5, splitrule=variance 
## + Fold4: mtry=13, min.node.size=5, splitrule=variance 
## - Fold4: mtry=13, min.node.size=5, splitrule=variance 
## + Fold4: mtry= 2, min.node.size=5, splitrule=extratrees 
## - Fold4: mtry= 2, min.node.size=5, splitrule=extratrees 
## + Fold4: mtry= 7, min.node.size=5, splitrule=extratrees 
## - Fold4: mtry= 7, min.node.size=5, splitrule=extratrees 
## + Fold4: mtry=13, min.node.size=5, splitrule=extratrees 
## - Fold4: mtry=13, min.node.size=5, splitrule=extratrees 
## + Fold5: mtry= 2, min.node.size=5, splitrule=variance 
## - Fold5: mtry= 2, min.node.size=5, splitrule=variance 
## + Fold5: mtry= 7, min.node.size=5, splitrule=variance 
## - Fold5: mtry= 7, min.node.size=5, splitrule=variance 
## + Fold5: mtry=13, min.node.size=5, splitrule=variance 
## - Fold5: mtry=13, min.node.size=5, splitrule=variance 
## + Fold5: mtry= 2, min.node.size=5, splitrule=extratrees 
## - Fold5: mtry= 2, min.node.size=5, splitrule=extratrees 
## + Fold5: mtry= 7, min.node.size=5, splitrule=extratrees 
## - Fold5: mtry= 7, min.node.size=5, splitrule=extratrees 
## + Fold5: mtry=13, min.node.size=5, splitrule=extratrees 
## - Fold5: mtry=13, min.node.size=5, splitrule=extratrees 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 13, splitrule = variance, min.node.size = 5 on full training set
```

```r
# Print model to console
model
```

```
## Random Forest 
## 
## 100 samples
##  13 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 79, 80, 81, 80, 80 
## Resampling results across tuning parameters:
## 
##   mtry  splitrule   RMSE       Rsquared   MAE      
##    2    variance    0.6527076  0.3404939  0.4963580
##    2    extratrees  0.6847222  0.2517920  0.5046799
##    7    variance    0.6364057  0.3508543  0.4838560
##    7    extratrees  0.6823771  0.2345015  0.5111203
##   13    variance    0.6229532  0.3864590  0.4830788
##   13    extratrees  0.6815077  0.2505217  0.5134303
## 
## Tuning parameter 'min.node.size' was held constant at a value of 5
## RMSE was used to select the optimal model using the smallest value.
## The final values used for the model were mtry = 13, splitrule =
##  variance and min.node.size = 5.
```

```r
# Plot model
plot(model)
```

![](Machine-Learning-Toolbox_files/figure-html/Tltl-1.png)<!-- -->

Fit a random forest with custom tuning
Now that you've explored the default tuning grids provided by the train() function, let's customize your models a bit more.

You can provide any number of values for mtry, from 2 up to the number of columns in the dataset. In practice, there are diminishing returns for much larger values of mtry, so you will use a custom tuning grid that explores 2 simple models (mtry = 2 and mtry = 3) as well as one more complicated model (mtry = 7).


```r
# Define the tuning grid: tuneGrid
tuneGrid <- data.frame(
  .mtry = c(2,3,7),
  .splitrule = "variance",
  .min.node.size = 5
)


# Fit random forest: model
model <- train(
  quality~.,
  tuneGrid = tuneGrid,
  data = wine, 
  method = "ranger",
  trControl = trainControl(
    method = "cv", 
    number = 5, 
    verboseIter = TRUE
  )
)
```

```
## + Fold1: mtry=2, splitrule=variance, min.node.size=5 
## - Fold1: mtry=2, splitrule=variance, min.node.size=5 
## + Fold1: mtry=3, splitrule=variance, min.node.size=5 
## - Fold1: mtry=3, splitrule=variance, min.node.size=5 
## + Fold1: mtry=7, splitrule=variance, min.node.size=5 
## - Fold1: mtry=7, splitrule=variance, min.node.size=5 
## + Fold2: mtry=2, splitrule=variance, min.node.size=5 
## - Fold2: mtry=2, splitrule=variance, min.node.size=5 
## + Fold2: mtry=3, splitrule=variance, min.node.size=5 
## - Fold2: mtry=3, splitrule=variance, min.node.size=5 
## + Fold2: mtry=7, splitrule=variance, min.node.size=5 
## - Fold2: mtry=7, splitrule=variance, min.node.size=5 
## + Fold3: mtry=2, splitrule=variance, min.node.size=5 
## - Fold3: mtry=2, splitrule=variance, min.node.size=5 
## + Fold3: mtry=3, splitrule=variance, min.node.size=5 
## - Fold3: mtry=3, splitrule=variance, min.node.size=5 
## + Fold3: mtry=7, splitrule=variance, min.node.size=5 
## - Fold3: mtry=7, splitrule=variance, min.node.size=5 
## + Fold4: mtry=2, splitrule=variance, min.node.size=5 
## - Fold4: mtry=2, splitrule=variance, min.node.size=5 
## + Fold4: mtry=3, splitrule=variance, min.node.size=5 
## - Fold4: mtry=3, splitrule=variance, min.node.size=5 
## + Fold4: mtry=7, splitrule=variance, min.node.size=5 
## - Fold4: mtry=7, splitrule=variance, min.node.size=5 
## + Fold5: mtry=2, splitrule=variance, min.node.size=5 
## - Fold5: mtry=2, splitrule=variance, min.node.size=5 
## + Fold5: mtry=3, splitrule=variance, min.node.size=5 
## - Fold5: mtry=3, splitrule=variance, min.node.size=5 
## + Fold5: mtry=7, splitrule=variance, min.node.size=5 
## - Fold5: mtry=7, splitrule=variance, min.node.size=5 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 7, splitrule = variance, min.node.size = 5 on full training set
```

```r
# Print model to console
model
```

```
## Random Forest 
## 
## 100 samples
##  13 predictor
## 
## No pre-processing
## Resampling: Cross-Validated (5 fold) 
## Summary of sample sizes: 80, 79, 80, 81, 80 
## Resampling results across tuning parameters:
## 
##   mtry  RMSE       Rsquared   MAE      
##   2     0.6751737  0.2774521  0.5188755
##   3     0.6614311  0.3099066  0.5094203
##   7     0.6557326  0.3076607  0.5002893
## 
## Tuning parameter 'splitrule' was held constant at a value of
##  variance
## Tuning parameter 'min.node.size' was held constant at a value
##  of 5
## RMSE was used to select the optimal model using the smallest value.
## The final values used for the model were mtry = 7, splitrule =
##  variance and min.node.size = 5.
```

```r
# Plot model
plot(model)
```

![](Machine-Learning-Toolbox_files/figure-html/frfwct-1.png)<!-- -->

Make a custom trainControl
The wine quality dataset was a regression problem, but now you are looking at a classification problem. This is a simulated dataset based on the "don't overfit" competition on Kaggle a number of years ago.

Classification problems are a little more complicated than regression problems because you have to provide a custom summaryFunction to the train() function to use the AUC metric to rank your models. Start by making a custom trainControl, as you did in the previous chapter. Be sure to set classProbs = TRUE, otherwise the twoClassSummary for summaryFunction will break.


```r
# Create custom trainControl: myControl
myControl <- trainControl(method = "cv", number = 10, summaryFunction = twoClassSummary, classProbs = TRUE, # IMPORTANT!
  verboseIter = TRUE)
```

Fit glmnet with custom trainControl
Now that you have a custom trainControl object, fit a glmnet model to the "don't overfit" dataset. Recall from the video that glmnet is an extension of the generalized linear regression model (or glm) that places constraints on the magnitude of the coefficients to prevent overfitting. This is more commonly known as "penalized" regression modeling and is a very useful technique on datasets with many predictors and few values.

glmnet is capable of fitting two different kinds of penalized models, controlled by the alpha parameter:

Ridge regression (or alpha = 0)
Lasso regression (or alpha = 1)
You'll now fit a glmnet model to the "don't overfit" dataset using the defaults provided by the caret package.


```r
overfit <- read.csv("overfit.csv")

# Fit glmnet model: model
model <- train(
  y ~ ., 
  overfit,
  method = "glmnet",
  trControl = myControl
)
```

```
## + Fold01: alpha=0.10, lambda=0.01013 
## - Fold01: alpha=0.10, lambda=0.01013 
## + Fold01: alpha=0.55, lambda=0.01013 
## - Fold01: alpha=0.55, lambda=0.01013 
## + Fold01: alpha=1.00, lambda=0.01013 
## - Fold01: alpha=1.00, lambda=0.01013 
## + Fold02: alpha=0.10, lambda=0.01013 
## - Fold02: alpha=0.10, lambda=0.01013 
## + Fold02: alpha=0.55, lambda=0.01013 
## - Fold02: alpha=0.55, lambda=0.01013 
## + Fold02: alpha=1.00, lambda=0.01013 
## - Fold02: alpha=1.00, lambda=0.01013 
## + Fold03: alpha=0.10, lambda=0.01013 
## - Fold03: alpha=0.10, lambda=0.01013 
## + Fold03: alpha=0.55, lambda=0.01013 
## - Fold03: alpha=0.55, lambda=0.01013 
## + Fold03: alpha=1.00, lambda=0.01013 
## - Fold03: alpha=1.00, lambda=0.01013 
## + Fold04: alpha=0.10, lambda=0.01013 
## - Fold04: alpha=0.10, lambda=0.01013 
## + Fold04: alpha=0.55, lambda=0.01013 
## - Fold04: alpha=0.55, lambda=0.01013 
## + Fold04: alpha=1.00, lambda=0.01013 
## - Fold04: alpha=1.00, lambda=0.01013 
## + Fold05: alpha=0.10, lambda=0.01013 
## - Fold05: alpha=0.10, lambda=0.01013 
## + Fold05: alpha=0.55, lambda=0.01013 
## - Fold05: alpha=0.55, lambda=0.01013 
## + Fold05: alpha=1.00, lambda=0.01013 
## - Fold05: alpha=1.00, lambda=0.01013 
## + Fold06: alpha=0.10, lambda=0.01013 
## - Fold06: alpha=0.10, lambda=0.01013 
## + Fold06: alpha=0.55, lambda=0.01013 
## - Fold06: alpha=0.55, lambda=0.01013 
## + Fold06: alpha=1.00, lambda=0.01013 
## - Fold06: alpha=1.00, lambda=0.01013 
## + Fold07: alpha=0.10, lambda=0.01013 
## - Fold07: alpha=0.10, lambda=0.01013 
## + Fold07: alpha=0.55, lambda=0.01013 
## - Fold07: alpha=0.55, lambda=0.01013 
## + Fold07: alpha=1.00, lambda=0.01013 
## - Fold07: alpha=1.00, lambda=0.01013 
## + Fold08: alpha=0.10, lambda=0.01013 
## - Fold08: alpha=0.10, lambda=0.01013 
## + Fold08: alpha=0.55, lambda=0.01013 
## - Fold08: alpha=0.55, lambda=0.01013 
## + Fold08: alpha=1.00, lambda=0.01013 
## - Fold08: alpha=1.00, lambda=0.01013 
## + Fold09: alpha=0.10, lambda=0.01013 
## - Fold09: alpha=0.10, lambda=0.01013 
## + Fold09: alpha=0.55, lambda=0.01013 
## - Fold09: alpha=0.55, lambda=0.01013 
## + Fold09: alpha=1.00, lambda=0.01013 
## - Fold09: alpha=1.00, lambda=0.01013 
## + Fold10: alpha=0.10, lambda=0.01013 
## - Fold10: alpha=0.10, lambda=0.01013 
## + Fold10: alpha=0.55, lambda=0.01013 
## - Fold10: alpha=0.55, lambda=0.01013 
## + Fold10: alpha=1.00, lambda=0.01013 
## - Fold10: alpha=1.00, lambda=0.01013 
## Aggregating results
## Selecting tuning parameters
## Fitting alpha = 0.1, lambda = 0.0101 on full training set
```

```r
# Print model to console
model
```

```
## glmnet 
## 
## 250 samples
## 201 predictors
##   2 classes: 'class1', 'class2' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 225, 225, 225, 224, 225, 225, ... 
## Resampling results across tuning parameters:
## 
##   alpha  lambda        ROC        Sens  Spec     
##   0.10   0.0001012745  0.4646739  0.1   0.9574275
##   0.10   0.0010127448  0.4652174  0.1   0.9701087
##   0.10   0.0101274483  0.4699275  0.0   0.9875000
##   0.55   0.0001012745  0.4074275  0.1   0.9530797
##   0.55   0.0010127448  0.4223732  0.1   0.9532609
##   0.55   0.0101274483  0.4527174  0.1   0.9831522
##   1.00   0.0001012745  0.4052536  0.1   0.9317029
##   1.00   0.0010127448  0.4119565  0.1   0.9403986
##   1.00   0.0101274483  0.4498188  0.1   0.9748188
## 
## ROC was used to select the optimal model using the largest value.
## The final values used for the model were alpha = 0.1 and lambda
##  = 0.01012745.
```

```r
# Print maximum ROC statistic
max(model[["results"]][["ROC"]])
```

```
## [1] 0.4699275
```


```r
# Train glmnet with custom trainControl and tuning: model
model <- train(y~., data = overfit,
  tuneGrid = expand.grid(alpha = 0:1,lambda = seq(0.0001, 1, length = 20)),
  method = "glmnet",
  trControl = myControl
)
```

```
## + Fold01: alpha=0, lambda=1 
## - Fold01: alpha=0, lambda=1 
## + Fold01: alpha=1, lambda=1 
## - Fold01: alpha=1, lambda=1 
## + Fold02: alpha=0, lambda=1 
## - Fold02: alpha=0, lambda=1 
## + Fold02: alpha=1, lambda=1 
## - Fold02: alpha=1, lambda=1 
## + Fold03: alpha=0, lambda=1 
## - Fold03: alpha=0, lambda=1 
## + Fold03: alpha=1, lambda=1 
## - Fold03: alpha=1, lambda=1 
## + Fold04: alpha=0, lambda=1 
## - Fold04: alpha=0, lambda=1 
## + Fold04: alpha=1, lambda=1 
## - Fold04: alpha=1, lambda=1 
## + Fold05: alpha=0, lambda=1 
## - Fold05: alpha=0, lambda=1 
## + Fold05: alpha=1, lambda=1 
## - Fold05: alpha=1, lambda=1 
## + Fold06: alpha=0, lambda=1 
## - Fold06: alpha=0, lambda=1 
## + Fold06: alpha=1, lambda=1 
## - Fold06: alpha=1, lambda=1 
## + Fold07: alpha=0, lambda=1 
## - Fold07: alpha=0, lambda=1 
## + Fold07: alpha=1, lambda=1 
## - Fold07: alpha=1, lambda=1 
## + Fold08: alpha=0, lambda=1 
## - Fold08: alpha=0, lambda=1 
## + Fold08: alpha=1, lambda=1 
## - Fold08: alpha=1, lambda=1 
## + Fold09: alpha=0, lambda=1 
## - Fold09: alpha=0, lambda=1 
## + Fold09: alpha=1, lambda=1 
## - Fold09: alpha=1, lambda=1 
## + Fold10: alpha=0, lambda=1 
## - Fold10: alpha=0, lambda=1 
## + Fold10: alpha=1, lambda=1 
## - Fold10: alpha=1, lambda=1 
## Aggregating results
## Selecting tuning parameters
## Fitting alpha = 1, lambda = 0.0527 on full training set
```

```r
# Print model to console
model
```

```
## glmnet 
## 
## 250 samples
## 201 predictors
##   2 classes: 'class1', 'class2' 
## 
## No pre-processing
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 225, 224, 226, 224, 225, 225, ... 
## Resampling results across tuning parameters:
## 
##   alpha  lambda      ROC        Sens  Spec     
##   0      0.00010000  0.4624094  0.0   0.9742754
##   0      0.05272632  0.4536232  0.0   0.9958333
##   0      0.10535263  0.4601449  0.0   1.0000000
##   0      0.15797895  0.4688406  0.0   1.0000000
##   0      0.21060526  0.4751812  0.0   1.0000000
##   0      0.26323158  0.4795290  0.0   1.0000000
##   0      0.31585789  0.4817935  0.0   1.0000000
##   0      0.36848421  0.4839674  0.0   1.0000000
##   0      0.42111053  0.4796196  0.0   1.0000000
##   0      0.47373684  0.4817935  0.0   1.0000000
##   0      0.52636316  0.4859601  0.0   1.0000000
##   0      0.57898947  0.4859601  0.0   1.0000000
##   0      0.63161579  0.4816123  0.0   1.0000000
##   0      0.68424211  0.4836957  0.0   1.0000000
##   0      0.73686842  0.4836957  0.0   1.0000000
##   0      0.78949474  0.4858696  0.0   1.0000000
##   0      0.84212105  0.4815217  0.0   1.0000000
##   0      0.89474737  0.4815217  0.0   1.0000000
##   0      0.94737368  0.4815217  0.0   1.0000000
##   0      1.00000000  0.4815217  0.0   1.0000000
##   1      0.00010000  0.4250906  0.1   0.9443841
##   1      0.05272632  0.5086957  0.0   1.0000000
##   1      0.10535263  0.5000000  0.0   1.0000000
##   1      0.15797895  0.5000000  0.0   1.0000000
##   1      0.21060526  0.5000000  0.0   1.0000000
##   1      0.26323158  0.5000000  0.0   1.0000000
##   1      0.31585789  0.5000000  0.0   1.0000000
##   1      0.36848421  0.5000000  0.0   1.0000000
##   1      0.42111053  0.5000000  0.0   1.0000000
##   1      0.47373684  0.5000000  0.0   1.0000000
##   1      0.52636316  0.5000000  0.0   1.0000000
##   1      0.57898947  0.5000000  0.0   1.0000000
##   1      0.63161579  0.5000000  0.0   1.0000000
##   1      0.68424211  0.5000000  0.0   1.0000000
##   1      0.73686842  0.5000000  0.0   1.0000000
##   1      0.78949474  0.5000000  0.0   1.0000000
##   1      0.84212105  0.5000000  0.0   1.0000000
##   1      0.89474737  0.5000000  0.0   1.0000000
##   1      0.94737368  0.5000000  0.0   1.0000000
##   1      1.00000000  0.5000000  0.0   1.0000000
## 
## ROC was used to select the optimal model using the largest value.
## The final values used for the model were alpha = 1 and lambda = 0.05272632.
```

```r
# Print maximum ROC statistic
max(model[["results"]][["ROC"]])
```

```
## [1] 0.5086957
```

## Preprocessing

Apply median imputation
In this chapter, you'll be using a version of the Wisconsin Breast Cancer dataset. This dataset presents a classic binary classification problem: 50% of the samples are benign, 50% are malignant, and the challenge is to identify which are which.

This dataset is interesting because many of the predictors contain missing values and most rows of the dataset have at least one missing value. This presents a modeling challenge, because most machine learning algorithms cannot handle missing values out of the box. For example, your first instinct might be to fit a logistic regression model to this data, but prior to doing this you need a strategy for handling the NAs.

Fortunately, the train() function in caret contains an argument called preProcess, which allows you to specify that median imputation should be used to fill in the missing values. In previous chapters, you created models with the train() function using formulas such as y ~ .. An alternative way is to specify the x and y arguments to train(), where x is an object with samples in rows and features in columns and y is a numeric or factor vector containing the outcomes. Said differently, x is a matrix or data frame that contains the whole dataset you'd use for the data argument to the lm() call, for example, but excludes the response variable column; y is a vector that contains just the response variable column.

For this exercise, the argument x to train() is loaded in your workspace as breast_cancer_x and y as breast_cancer_y.


```r
breast_cancer_x <- read.csv("breast_cancer_x.csv") %>% select(-1)
breast_cancer_y <- read.csv("breast_cancer_y.csv") %>% select(-1)
breast_cancer_y <- as.vector(breast_cancer_y$x)


# Apply median imputation: median_model
median_model <- train(x = breast_cancer_x, y = breast_cancer_y, method = "glm", trControl = myControl,
  preProcess = "medianImpute")
```

```
## + Fold01: parameter=none 
## - Fold01: parameter=none 
## + Fold02: parameter=none 
## - Fold02: parameter=none 
## + Fold03: parameter=none 
## - Fold03: parameter=none 
## + Fold04: parameter=none 
## - Fold04: parameter=none 
## + Fold05: parameter=none 
## - Fold05: parameter=none 
## + Fold06: parameter=none 
## - Fold06: parameter=none 
## + Fold07: parameter=none 
## - Fold07: parameter=none 
## + Fold08: parameter=none 
## - Fold08: parameter=none 
## + Fold09: parameter=none 
## - Fold09: parameter=none 
## + Fold10: parameter=none 
## - Fold10: parameter=none 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print median_model to console
median_model
```

```
## Generalized Linear Model 
## 
## 699 samples
##   9 predictor
##   2 classes: 'benign', 'malignant' 
## 
## Pre-processing: median imputation (9) 
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 629, 629, 629, 628, 629, 630, ... 
## Resampling results:
## 
##   ROC        Sens       Spec     
##   0.9913969  0.9694203  0.9461667
```

Use KNN imputation
In the previous exercise, you used median imputation to fill in missing values in the breast cancer dataset, but that is not the only possible method for dealing with missing data.

An alternative to median imputation is k-nearest neighbors, or KNN, imputation. This is a more advanced form of imputation where missing values are replaced with values from other rows that are similar to the current row. While this is a lot more complicated to implement in practice than simple median imputation, it is very easy to explore in caret using the preProcess argument to train(). You can simply use preProcess = "knnImpute" to change the method of imputation used prior to model fitting.


```r
# Apply KNN imputation: knn_model
knn_model <- train(x = breast_cancer_x, y = breast_cancer_y, method = "glm", trControl = myControl,
  preProcess = "knnImpute")
```

```
## + Fold01: parameter=none 
## - Fold01: parameter=none 
## + Fold02: parameter=none 
## - Fold02: parameter=none 
## + Fold03: parameter=none 
## - Fold03: parameter=none 
## + Fold04: parameter=none 
## - Fold04: parameter=none 
## + Fold05: parameter=none 
## - Fold05: parameter=none 
## + Fold06: parameter=none 
## - Fold06: parameter=none 
## + Fold07: parameter=none 
## - Fold07: parameter=none 
## + Fold08: parameter=none 
## - Fold08: parameter=none 
## + Fold09: parameter=none 
## - Fold09: parameter=none 
## + Fold10: parameter=none 
## - Fold10: parameter=none 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print knn_model to console
knn_model
```

```
## Generalized Linear Model 
## 
## 699 samples
##   9 predictor
##   2 classes: 'benign', 'malignant' 
## 
## Pre-processing: nearest neighbor imputation (9), centered (9), scaled (9) 
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 630, 629, 629, 629, 630, 628, ... 
## Resampling results:
## 
##   ROC        Sens       Spec     
##   0.9909863  0.9715459  0.9376667
```

Combining preprocessing methods
The preProcess argument to train() doesn't just limit you to imputing missing values. It also includes a wide variety of other preProcess techniques to make your life as a data scientist much easier. You can read a full list of them by typing ?preProcess and reading the help page for this function.

One set of preprocessing functions that is particularly useful for fitting regression models is standardization: centering and scaling. You first center by subtracting the mean of each column from each value in that column, then you scale by dividing by the standard deviation.

Standardization transforms your data such that for each column, the mean is 0 and the standard deviation is 1. This makes it easier for regression models to find a good solution.


```r
# Fit glm with median imputation
model <- train(x = breast_cancer_x, y = breast_cancer_y, method = "glm", trControl = myControl,
  preProcess = "medianImpute")
```

```
## + Fold01: parameter=none 
## - Fold01: parameter=none 
## + Fold02: parameter=none 
## - Fold02: parameter=none 
## + Fold03: parameter=none 
## - Fold03: parameter=none 
## + Fold04: parameter=none 
## - Fold04: parameter=none 
## + Fold05: parameter=none 
## - Fold05: parameter=none 
## + Fold06: parameter=none 
## - Fold06: parameter=none 
## + Fold07: parameter=none 
## - Fold07: parameter=none 
## + Fold08: parameter=none 
## - Fold08: parameter=none 
## + Fold09: parameter=none 
## - Fold09: parameter=none 
## + Fold10: parameter=none 
## - Fold10: parameter=none 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print knn_model to console
model
```

```
## Generalized Linear Model 
## 
## 699 samples
##   9 predictor
##   2 classes: 'benign', 'malignant' 
## 
## Pre-processing: median imputation (9) 
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 629, 629, 629, 628, 629, 630, ... 
## Resampling results:
## 
##   ROC       Sens       Spec     
##   0.992494  0.9695169  0.9458333
```

```r
# Update model with standardization
model <- train(x = breast_cancer_x, y = breast_cancer_y, method = "glm", trControl = myControl,
  preProcess = c("medianImpute","center","scale"))
```

```
## + Fold01: parameter=none 
## - Fold01: parameter=none 
## + Fold02: parameter=none 
## - Fold02: parameter=none 
## + Fold03: parameter=none 
## - Fold03: parameter=none 
## + Fold04: parameter=none 
## - Fold04: parameter=none 
## + Fold05: parameter=none 
## - Fold05: parameter=none 
## + Fold06: parameter=none 
## - Fold06: parameter=none 
## + Fold07: parameter=none 
## - Fold07: parameter=none 
## + Fold08: parameter=none 
## - Fold08: parameter=none 
## + Fold09: parameter=none 
## - Fold09: parameter=none 
## + Fold10: parameter=none 
## - Fold10: parameter=none 
## Aggregating results
## Fitting final model on full training set
```

```r
# Print updated model
model
```

```
## Generalized Linear Model 
## 
## 699 samples
##   9 predictor
##   2 classes: 'benign', 'malignant' 
## 
## Pre-processing: median imputation (9), centered (9), scaled (9) 
## Resampling: Cross-Validated (10 fold) 
## Summary of sample sizes: 629, 628, 629, 629, 629, 630, ... 
## Resampling results:
## 
##   ROC        Sens       Spec     
##   0.9910105  0.9716425  0.9291667
```

Remove near zero variance predictors
As you saw in the video, for the next set of exercises, you'll be using the blood-brain dataset. This is a biochemical dataset in which the task is to predict the following value for a set of biochemical compounds:

log((concentration of compound in brain) /
      (concentration of compound in blood))
This gives a quantitative metric of the compound's ability to cross the blood-brain barrier, and is useful for understanding the biological properties of that barrier.

One interesting aspect of this dataset is that it contains many variables and many of these variables have extemely low variances. This means that there is very little information in these variables because they mostly consist of a single value (e.g. zero).

Fortunately, caret contains a utility function called nearZeroVar() for removing such variables to save time during modeling.

nearZeroVar() takes in data x, then looks at the ratio of the most common value to the second most common value, freqCut, and the percentage of distinct values out of the number of total samples, uniqueCut. By default, caret uses freqCut = 19 and uniqueCut = 10, which is fairly conservative. I like to be a little more aggressive and use freqCut = 2 and uniqueCut = 20 when calling nearZeroVar().


```r
bloodbrain_x <- read.csv("bloodbrain_x.csv") %>% select(-1)
bloodbrain_y <- read.csv("bloodbrain_y.csv") %>% select(-1)
bloodbrain_y <- as.vector(bloodbrain_y$x)

# Identify near zero variance predictors: remove_cols
remove_cols <- nearZeroVar(bloodbrain_x, names = TRUE, 
                           freqCut = 2, uniqueCut = 20)

# Get all column names from bloodbrain_x: all_cols
all_cols <- names(bloodbrain_x)

# Remove from data: bloodbrain_x_small
bloodbrain_x_small <- bloodbrain_x[ , setdiff(all_cols, remove_cols)]
```

Fit model on reduced blood-brain data
Now that you've reduced your dataset, you can fit a glm model to it using the train() function. This model will run faster than using the full dataset and will yield very similar predictive accuracy.

Furthermore, zero variance variables can cause problems with cross-validation (e.g. if one fold ends up with only a single unique value for that variable), so removing them prior to modeling means you are less likely to get errors during the fitting process.

bloodbrain_x, bloodbrain_y, remove, and bloodbrain_x_small are loaded in your workspace.

Fit a glm model using the train() function and the reduced blood-brain dataset you created in the previous exercise.
Print the result to the console.


```r
remove <- remove_cols

# Fit model on reduced data: model
model <- train(x = bloodbrain_x_small, y = bloodbrain_y, method = "glm")

# Print model to console
model
```

```
## Generalized Linear Model 
## 
## 208 samples
## 112 predictors
## 
## No pre-processing
## Resampling: Bootstrapped (25 reps) 
## Summary of sample sizes: 208, 208, 208, 208, 208, 208, ... 
## Resampling results:
## 
##   RMSE      Rsquared   MAE     
##   1082.142  0.1091276  257.9763
```

Using PCA as an alternative to nearZeroVar()
An alternative to removing low-variance predictors is to run PCA on your dataset. This is sometimes preferable because it does not throw out all of your data: many different low variance predictors may end up combined into one high variance PCA variable, which might have a positive impact on your model's accuracy.

This is an especially good trick for linear models: the pca option in the preProcess argument will center and scale your data, combine low variance variables, and ensure that all of your predictors are orthogonal. This creates an ideal dataset for linear regression modeling, and can often improve the accuracy of your models.


```r
# Fit glm model using PCA: model
model <- train(x = bloodbrain_x, y = bloodbrain_y, method = "glm", preProcess = "pca")

# Print model to console
model
```

```
## Generalized Linear Model 
## 
## 208 samples
## 132 predictors
## 
## Pre-processing: principal component signal extraction (132),
##  centered (132), scaled (132) 
## Resampling: Bootstrapped (25 reps) 
## Summary of sample sizes: 208, 208, 208, 208, 208, 208, ... 
## Resampling results:
## 
##   RMSE       Rsquared   MAE      
##   0.6095159  0.4174699  0.4614212
```

## Selecting Models

Make custom train/test indices
As you saw in the video, for this chapter you will focus on a real-world dataset that brings together all of the concepts discussed in the previous chapters.

The churn dataset contains data on a variety of telecom customers and the modeling challenge is to predict which customers will cancel their service (or churn).

In this chapter, you will be exploring two different types of predictive models: glmnet and rf, so the first order of business is to create a reusable trainControl object you can use to reliably compare them.


```r
churn_y <- read.csv("churn_y.csv")
churn_y <- as.vector(churn_y$x)

# Create custom indices: myFolds
myFolds <- createFolds(churn_y, k = 5)

# Create reusable trainControl object: myControl
myControl <- trainControl(summaryFunction = twoClassSummary, classProbs = TRUE, # IMPORTANT!
  verboseIter = TRUE, savePredictions = TRUE, index = myFolds)
```

Fit the baseline model
Now that you have a reusable trainControl object called myControl, you can start fitting different predictive models to your churn dataset and evaluate their predictive accuracy.

You'll start with one of my favorite models, glmnet, which penalizes linear and logistic regression models on the size and number of coefficients to help prevent overfitting.


```r
churn_x <- read.csv("churn_x.csv") %>% select(-1)

# Fit glmnet model: model_glmnet
model_glmnet <- train(x = churn_x, y = churn_y, metric = "ROC", method = "glmnet", trControl = myControl)
```

```
## + Fold1: alpha=0.10, lambda=0.01821 
## - Fold1: alpha=0.10, lambda=0.01821 
## + Fold1: alpha=0.55, lambda=0.01821 
## - Fold1: alpha=0.55, lambda=0.01821 
## + Fold1: alpha=1.00, lambda=0.01821 
## - Fold1: alpha=1.00, lambda=0.01821 
## + Fold2: alpha=0.10, lambda=0.01821 
## - Fold2: alpha=0.10, lambda=0.01821 
## + Fold2: alpha=0.55, lambda=0.01821 
## - Fold2: alpha=0.55, lambda=0.01821 
## + Fold2: alpha=1.00, lambda=0.01821 
## - Fold2: alpha=1.00, lambda=0.01821 
## + Fold3: alpha=0.10, lambda=0.01821 
## - Fold3: alpha=0.10, lambda=0.01821 
## + Fold3: alpha=0.55, lambda=0.01821 
## - Fold3: alpha=0.55, lambda=0.01821 
## + Fold3: alpha=1.00, lambda=0.01821 
## - Fold3: alpha=1.00, lambda=0.01821 
## + Fold4: alpha=0.10, lambda=0.01821 
## - Fold4: alpha=0.10, lambda=0.01821 
## + Fold4: alpha=0.55, lambda=0.01821 
## - Fold4: alpha=0.55, lambda=0.01821 
## + Fold4: alpha=1.00, lambda=0.01821 
## - Fold4: alpha=1.00, lambda=0.01821 
## + Fold5: alpha=0.10, lambda=0.01821 
## - Fold5: alpha=0.10, lambda=0.01821 
## + Fold5: alpha=0.55, lambda=0.01821 
## - Fold5: alpha=0.55, lambda=0.01821 
## + Fold5: alpha=1.00, lambda=0.01821 
## - Fold5: alpha=1.00, lambda=0.01821 
## Aggregating results
## Selecting tuning parameters
## Fitting alpha = 0.55, lambda = 0.0182 on full training set
```

Random forest with custom trainControl
Another one of my favorite models is the random forest, which combines an ensemble of non-linear decision trees into a highly flexible (and usually quite accurate) model.

Rather than using the classic randomForest package, you'll be using the ranger package, which is a re-implementation of randomForest that produces almost the exact same results, but is faster, more stable, and uses less memory. I highly recommend it as a starting point for random forest modeling in R.


```r
# Fit random forest: model_rf
model_rf <- train(x = churn_x, y = churn_y, metric = "ROC", method = "ranger", trControl = myControl)
```

```
## + Fold1: mtry= 2, min.node.size=1, splitrule=gini 
## - Fold1: mtry= 2, min.node.size=1, splitrule=gini 
## + Fold1: mtry=36, min.node.size=1, splitrule=gini 
## - Fold1: mtry=36, min.node.size=1, splitrule=gini 
## + Fold1: mtry=70, min.node.size=1, splitrule=gini 
## - Fold1: mtry=70, min.node.size=1, splitrule=gini 
## + Fold1: mtry= 2, min.node.size=1, splitrule=extratrees 
## - Fold1: mtry= 2, min.node.size=1, splitrule=extratrees 
## + Fold1: mtry=36, min.node.size=1, splitrule=extratrees 
## - Fold1: mtry=36, min.node.size=1, splitrule=extratrees 
## + Fold1: mtry=70, min.node.size=1, splitrule=extratrees 
## - Fold1: mtry=70, min.node.size=1, splitrule=extratrees 
## + Fold2: mtry= 2, min.node.size=1, splitrule=gini 
## - Fold2: mtry= 2, min.node.size=1, splitrule=gini 
## + Fold2: mtry=36, min.node.size=1, splitrule=gini 
## - Fold2: mtry=36, min.node.size=1, splitrule=gini 
## + Fold2: mtry=70, min.node.size=1, splitrule=gini 
## - Fold2: mtry=70, min.node.size=1, splitrule=gini 
## + Fold2: mtry= 2, min.node.size=1, splitrule=extratrees 
## - Fold2: mtry= 2, min.node.size=1, splitrule=extratrees 
## + Fold2: mtry=36, min.node.size=1, splitrule=extratrees 
## - Fold2: mtry=36, min.node.size=1, splitrule=extratrees 
## + Fold2: mtry=70, min.node.size=1, splitrule=extratrees 
## - Fold2: mtry=70, min.node.size=1, splitrule=extratrees 
## + Fold3: mtry= 2, min.node.size=1, splitrule=gini 
## - Fold3: mtry= 2, min.node.size=1, splitrule=gini 
## + Fold3: mtry=36, min.node.size=1, splitrule=gini 
## - Fold3: mtry=36, min.node.size=1, splitrule=gini 
## + Fold3: mtry=70, min.node.size=1, splitrule=gini 
## - Fold3: mtry=70, min.node.size=1, splitrule=gini 
## + Fold3: mtry= 2, min.node.size=1, splitrule=extratrees 
## - Fold3: mtry= 2, min.node.size=1, splitrule=extratrees 
## + Fold3: mtry=36, min.node.size=1, splitrule=extratrees 
## - Fold3: mtry=36, min.node.size=1, splitrule=extratrees 
## + Fold3: mtry=70, min.node.size=1, splitrule=extratrees 
## - Fold3: mtry=70, min.node.size=1, splitrule=extratrees 
## + Fold4: mtry= 2, min.node.size=1, splitrule=gini 
## - Fold4: mtry= 2, min.node.size=1, splitrule=gini 
## + Fold4: mtry=36, min.node.size=1, splitrule=gini 
## - Fold4: mtry=36, min.node.size=1, splitrule=gini 
## + Fold4: mtry=70, min.node.size=1, splitrule=gini 
## - Fold4: mtry=70, min.node.size=1, splitrule=gini 
## + Fold4: mtry= 2, min.node.size=1, splitrule=extratrees 
## - Fold4: mtry= 2, min.node.size=1, splitrule=extratrees 
## + Fold4: mtry=36, min.node.size=1, splitrule=extratrees 
## - Fold4: mtry=36, min.node.size=1, splitrule=extratrees 
## + Fold4: mtry=70, min.node.size=1, splitrule=extratrees 
## - Fold4: mtry=70, min.node.size=1, splitrule=extratrees 
## + Fold5: mtry= 2, min.node.size=1, splitrule=gini 
## - Fold5: mtry= 2, min.node.size=1, splitrule=gini 
## + Fold5: mtry=36, min.node.size=1, splitrule=gini 
## - Fold5: mtry=36, min.node.size=1, splitrule=gini 
## + Fold5: mtry=70, min.node.size=1, splitrule=gini 
## - Fold5: mtry=70, min.node.size=1, splitrule=gini 
## + Fold5: mtry= 2, min.node.size=1, splitrule=extratrees 
## - Fold5: mtry= 2, min.node.size=1, splitrule=extratrees 
## + Fold5: mtry=36, min.node.size=1, splitrule=extratrees 
## - Fold5: mtry=36, min.node.size=1, splitrule=extratrees 
## + Fold5: mtry=70, min.node.size=1, splitrule=extratrees 
## - Fold5: mtry=70, min.node.size=1, splitrule=extratrees 
## Aggregating results
## Selecting tuning parameters
## Fitting mtry = 36, splitrule = extratrees, min.node.size = 1 on full training set
```

Create a resamples object
Now that you have fit two models to the churn dataset, it's time to compare their out-of-sample predictions and choose which one is the best model for your dataset.

You can compare models in caret using the resamples() function, provided they have the same training data and use the same trainControl object with preset cross-validation folds. resamples() takes as input a list of models and can be used to compare dozens of models at once (though in this case you are only comparing two models).


```r
# Create model_list
model_list <- list(item1 = model_glmnet, item2 = model_rf)

# Pass model_list to resamples(): resamples
resamples <- resamples(model_list)

# Summarize the results
summary(resamples)
```

```
## 
## Call:
## summary.resamples(object = resamples)
## 
## Models: item1, item2 
## Number of resamples: 5 
## 
## ROC 
##            Min.   1st Qu.    Median      Mean   3rd Qu.      Max. NA's
## item1 0.4489390 0.4832007 0.5296198 0.5440319 0.6178286 0.6405714    0
## item2 0.6731874 0.7002286 0.7095491 0.7060488 0.7180571 0.7292219    0
## 
## Sens 
##            Min.   1st Qu.    Median      Mean   3rd Qu.      Max. NA's
## item1 0.9195402 0.9482759 0.9542857 0.9552644 0.9657143 0.9885057    0
## item2 0.9712644 0.9885714 0.9942529 0.9896749 0.9942857 1.0000000    0
## 
## Spec 
##             Min.    1st Qu.     Median       Mean 3rd Qu. Max. NA's
## item1 0.03846154 0.03846154 0.07692308 0.09476923    0.08 0.24    0
## item2 0.00000000 0.00000000 0.00000000 0.01600000    0.00 0.08    0
```

Create a box-and-whisker plot
caret provides a variety of methods to use for comparing models. All of these methods are based on the resamples() function. My favorite is the box-and-whisker plot, which allows you to compare the distribution of predictive accuracy (in this case AUC) for the two models.

In general, you want the model with the higher median AUC, as well as a smaller range between min and max AUC.

You can make this plot using the bwplot() function, which makes a box and whisker plot of the model's out of sample scores. Box and whisker plots show the median of each distribution as a line and the interquartile range of each distribution as a box around the median line. You can pass the metric = "ROC" argument to the bwplot() function to show a plot of the model's out-of-sample ROC scores and choose the model with the highest median ROC.

If you do not specify a metric to plot, bwplot() will automatically plot 3 of them.


```r
# Create bwplot
bwplot(resamples, metric = "ROC")
```

![](Machine-Learning-Toolbox_files/figure-html/cbwp-1.png)<!-- -->

Create a scatterplot
Another useful plot for comparing models is the scatterplot, also known as the xy-plot. This plot shows you how similar the two models' performances are on different folds.

It's particularly useful for identifying if one model is consistently better than the other across all folds, or if there are situations when the inferior model produces better predictions on a particular subset of the data.


```r
# Create xyplot
xyplot(resamples, metric = "ROC")
```

![](Machine-Learning-Toolbox_files/figure-html/cs-1.png)<!-- -->

Ensembling models
That concludes the course! As a teaser for a future course on making ensembles of caret models, I'll show you how to fit a stacked ensemble of models using the caretEnsemble package.

caretEnsemble provides the caretList() function for creating multiple caret models at once on the same dataset, using the same resampling folds. You can also create your own lists of caret models.

In this exercise, I've made a caretList for you, containing the glmnet and ranger models you fit on the churn dataset. Use the caretStack() function to make a stack of caret models, with the two sub-models (glmnet and ranger) feeding into another (hopefully more accurate!) caret model.



