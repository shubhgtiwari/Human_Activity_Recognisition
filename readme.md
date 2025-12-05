# Load required libraries
2 library ( dplyr )
3 library ( earth )
4 library ( caret )
5 library ( corrplot )
6
7 # Load training and test datasets
8 train <- read . table ( " human + activity + recognition + using + smartphones /
UCI HAR Dataset / train / X _ train . txt " )
9 train _ labels <- read . table ( " human + activity + recognition + using +
smartphones / UCI HAR Dataset / train / y _ train . txt " )
10 train _ subject <- read . table ( " human + activity + recognition + using +
smartphones / UCI HAR Dataset / train / subject _ train . txt " )
11
12 test <- read . table ( " human + activity + recognition + using + smartphones /
UCI HAR Dataset / test / X _ test . txt " )
13 test _ labels <- read . table ( " human + activity + recognition + using +
smartphones / UCI HAR Dataset / test / y _ test . txt " )
14 test _ subject <- read . table ( " human + activity + recognition + using +
smartphones / UCI HAR Dataset / test / subject _ test . txt " )
15
16 # Load feature names
17 features <- read . table ( " human + activity + recognition + using +
smartphones / UCI HAR Dataset / features . txt " )
18
19 # Assign feature names to columns
20 colnames ( train ) <- features $ V2
21 colnames ( test ) <- features $ V2
22
23 # Add activity and subject columns
24 train $ Activity <- train _ labels $ V1
25 train $ Subject <- train _ subject $ V1
26 test $ Activity <- test _ labels $ V1
27 test $ Subject <- test _ subject $ V1
28
29 # Merge training and test datasets
30 har _ data <- rbind ( train , test )
31
32 # Convert activity labels to descriptive names
33 activity _ labels <- read . table ( " human + activity + recognition + using +
smartphones / UCI HAR Dataset / activity _ labels . txt " )
34 har _ data $ Activity <- factor ( har _ data $ Activity , levels = activity _
labels $ V1 , labels = activity _ labels $ V2 )
35
35
36 # Remove duplicate column names ( if any )
37 har _ data <- har _ data [ , ! duplicated ( colnames ( har _ data ) ) ]
38
39 # Clean column names
40 colnames ( har _ data ) <- gsub ( " [\\(\\) -] " , " " , colnames ( har _ data ) )
41 head ( har _ data , 1)
42
43 # Boxplot of first 10 predictors
44 boxplot ( har _ data [ , 1:10] ,
45 main = " Boxplots for the First 10 Predictors " ,
46 xlab = " Predictors " ,
47 ylab = " Values " ,
48 col = " lightpink " ,
49 las = 2 ,
50 cex . axis = 0.7)
51
52 # Identify numeric columns
53 numeric _ data <- har _ data [ , sapply ( har _ data , is . numeric ) ]
54
55 # Ensure all values are positive for Box - Cox
56 numeric _ data _ positive <- numeric _ data + abs ( min ( numeric _ data , na .
rm = TRUE ) ) + 1
57
58 # Apply Box - Cox Transformation
59 preProc _ bc <- preProcess ( numeric _ data _ positive , method = " BoxCox " )
60 transformed _ data _ bc <- predict ( preProc _ bc , numeric _ data _ positive )
61
62 # Apply Spatial Sign Transformation
63 preProc _ ss <- preProcess ( transformed _ data _ bc , method = "
spatialSign " )
64 transformed _ data _ final <- predict ( preProc _ ss , transformed _ data _ bc )
65
66 # Replace original numeric columns with transformed data
67 har _ data _ transformed <- har _ data $ Activity
68 har _ data _ transformed [ , sapply ( har _ data , is . numeric ) ] <-
transformed _ data _ final
69
70 # Summary of transformed data
71 summary ( transformed _ data _ final )
72 transformed _ data _ final $ Activity <- har _ data $ Activity
73
74 # Boxplots after both transformations
75 boxplot ( transformed _ data _ final [ , 1:10] ,
76 main = " Boxplots After Box - Cox and Spatial Sign
Transformation for First 10 Predictors " ,
77 xlab = " Predictors " ,
78 ylab = " Values " ,
79 col = " lightpink " ,
80 las = 2 ,
36
81 cex . axis = 0.7)
82
83 har _ data <- transformed _ data _ final
84
85 # Emptying subject as it has no significance further
86 har _ data $ Subject <- NULL
87
88 ncol ( har _ data ) # 478
89
90 # Correlation Matrix
91 # Extract only numeric columns from har _ data
92 numeric _ data <- har _ data [ , sapply ( har _ data , is . numeric ) ]
93
94 # Compute the correlation matrix
95 cor _ matrix <- cor ( numeric _ data , use = " pairwise . complete . obs " )
96
97 # Highly Correlated Predictors
98 high _ cor <- findCorrelation ( cor _ matrix , cutoff = 0.9)
99 har _ data _ filtered <- har _ data [ , - high _ cor ]
100
101 ncol ( har _ data _ filtered ) # 185
102
103 corrplot ( cor _ matrix , method = " color " , order = " hclust " ,
104 tl . col = " black " , tl . cex = 0.8 ,
105 title = " Correlation Heatmap ( Before Removal ) " ,
106 mar = c (0 , 0 , 2 , 0) )
107
108 # Correlation heatmap after removal
109 cor _ matrix _ filtered <- cor ( har _ data _ filtered , use = " pairwise .
complete . obs " )
110 corrplot ( cor _ matrix _ filtered , method = " color " , order = " hclust " ,
111 tl . col = " black " , tl . cex = 0.8 ,
112 title = " Correlation Heatmap ( After Removal ) " ,
113 mar = c (0 , 0 , 2 , 0) )
114
115 # Extract the first 10 predictors ( numeric columns )
116 first _ 10 _ predictors <- har _ data _ filtered [ , sapply ( har _ data _
filtered , is . numeric ) ][ , 1:10]
117
118 # Plot histograms for each predictor
119 par ( mfrow = c (3 , 3) ) # Arrange plots in a grid (2 rows , 5 columns
)
120 for ( i in 1: ncol ( first _ 10 _ predictors ) ) {
121 hist (
122 first _ 10 _ predictors [ , i ] ,
123 main = paste ( " Histogram of " , colnames ( first _ 10 _ predictors ) [ i ])
,
124 xlab = colnames ( first _ 10 _ predictors ) [ i ] ,
125 col = " lightpink " ,
37
126 border = " white "
127 )
128 }
129 par ( mfrow = c (1 , 1) ) # Reset plotting layout
130
131 pcaObject <- prcomp ( har _ data _ filtered [ , sapply ( har _ data _ filtered ,
is . numeric ) ] , center = TRUE , scale . = TRUE )
132 # Extract PCA scores
133 pca _ scores <- as . data . frame ( pcaObject $ x )
134
135 # Combine PCA scores with Activity labels
136 pca _ data <- cbind ( pca _ scores , Activity = har _ data _ filtered $
Activity )
137 nrow ( pca _ scores )
138 length ( har _ data _ filtered $ Activity )
139
140 # Ensure Activity is a factor
141 pca _ data $ Activity <- as . factor ( pca _ data $ Activity )
142
143 # View the first few rows of the resulting data frame
144 head ( pca _ data )
145
146 set . seed (123)
147 # Stratified sampling being used to split the data
148 trainIndex <- c r e at e D at a P ar t i ti o n ( pca _ data $ Activity , p = 0.8 , list
= FALSE )
149 # Splitting the data
150 trainData <- pca _ data [ trainIndex , ]
151 testData <- pca _ data [ - trainIndex , ]
152
153 # Extract labels for train and test
154 trainLabels <- pca _ data $ Activity
155 testLabels <- pca _ data $ Activity
156
157 # Remove the label column from the data
158 trainData <- pca _ data [ , - which ( names ( pca _ data ) == " Activity " ) ]
159 testData <- pca _ data [ , - which ( names ( pca _ data ) == " Activity " ) ]
160
161 # Model Training
162 tr _ control <- trainControl ( method = " repeatedcv " , number = 3 ,
repeats = 3 , summaryFunction = multiClassSummary , classProbs =
TRUE )
163 grid _ glm <- expand . grid ( alpha = c (0 , .2 , .4 , .6 , .8 , 1) , lambda =
seq (0.01 , 0.1 , by = 0.01) )
164
165 # Linear Discriminant Analysis ( LDA )
166 set . seed (123)
167 ldaModel <- train ( trainData , trainLabels , method = " lda " ,
trControl = tr _ control , metric = " Kappa " )
38
168 print ( ldaModel )
169 # Confusion Matrix
170 ldaModelTest <- confusionMatrix ( predict ( ldaModel , testData ) ,
testLabels )
171 print ( ldaModelTest )
172
173 # Partial least squares discriminant analysis ( PLSDA )
174 set . seed (123)
175 plsdaModel <- train ( trainData , trainLabels , method = " pls " ,
trControl = tr _ control , tuneLength = 10 , metric = " Kappa " )
176 print ( plsdaModel )
177 plot ( plsdaModel )
178
179 # Confusion Matrix
180 plsdaModelTest <- confusionMatrix ( predict ( plsdaModel , testData ) ,
testLabels )
181 print ( plsdaModelTest )
182
183 # Penalized Model
184 set . seed (123)
185 penModel <- train ( trainData , trainLabels , method = " glmnet " ,
trControl = tr _ control , tuneGrid = grid _ glm , metric = " Kappa " )
186 print ( penModel )
187 plot ( penModel )
188
189 # Confusion Matrix
190 penModelTest <- confusionMatrix ( predict ( penModel , testData ) ,
testLabels )
191
192 # Creating results table
193 results <- data . frame (
194 Model = c ( " LDA " , " PLSDA " , " Penalized " ) ,
195 Kappa _ Train = c ( max ( ldaModel $ results $ Kappa ) , max ( plsdaModel $
results $ Kappa ) , max ( penModel $ results $ Kappa ) ) ,
196 Kappa _ Test = c ( ldaModelTest $ overall [ ’ Kappa ’] , plsdaModelTest $
overall [ ’ Kappa ’] , penModelTest $ overall [ ’ Kappa ’ ])
197 )
198 print ( results )
199
200 # Important Variable
201 varImp ( penModel )
202 importance _ sorted <- varImp ( penModel ) $ importance [ order ( - varImp (
penModel ) $ importance [ , 1]) , ]
203 top _ 20 _ vars <- head ( importance _ sorted , 20)
204 plot ( top _ 20 _ vars )
205
206 barplot ( top _ 20 _ vars [ , 1] ,
207 names . arg = rownames ( top _ 20 _ vars ) ,
208 col = " lightpink " ,
39
209 las = 2 ,
210 main = " Top 20 Important Variables from glmnet Model " ,
211 ylab = " Importance " ,
212 cex . names = 0.7 , # Smaller x - axis labels
213 border = " white " )
214
215 # Assuming ’ penModel ’ is your model and you have the variable
importance
216 importance _ data <- varImp ( penModel ) $ importance
217
218 # Neural Networks
219 set . seed (123)
220 nnModel <- train ( trainData , trainLabels , method = " nnet " ,
trControl = tr _ control , tuneLength = 5 , metric = " Kappa " , trace
= FALSE )
221 print ( nnModel )
222 plot ( nnModel )
223 # Confusion Matrix
224 nnModelTest <- confusionMatrix ( predict ( nnModel , testData ) ,
testLabels )
225 print ( nnModelTest )
226
227 # Flexible Discriminant Analysis ( FDA )
228 set . seed (123)
229 fdaModel <- train ( trainData , trainLabels , method = " fda " ,
trControl = tr _ control , tuneLength = 10 , metric = " Kappa " )
230 print ( fdaModel )
231 plot ( fdaModel )
232 # Confusion Matrix
233 fdaModelTest <- confusionMatrix ( predict ( fdaModel , testData ) ,
testLabels )
234 print ( fdaModelTest )
235
236 # Support Vector Machines ( SVM )
237 set . seed (123)
238 svmModel <- train ( trainData , trainLabels , method = " svmRadial " ,
trControl = tr _ control , tuneLength = 10 , metric = " Kappa " )
239 print ( svmModel )
240 plot ( svmModel )
241 # Confusion Matrix
242 svmModelTest <- confusionMatrix ( predict ( svmModel , testData ) ,
testLabels )
243 print ( svmModelTest )
244
245 # K - Nearest Neighbors ( KNN )
246 set . seed (123)
247 knnModel <- train ( trainData , trainLabels , method = " knn " ,
trControl = tr _ control , tuneLength = 10 , metric = " Kappa " )
248 print ( knnModel )
40
249 plot ( knnModel )
250 # Confusion Matrix
251 knnModelTest <- confusionMatrix ( predict ( knnModel , testData ) ,
testLabels )
252 print ( knnModelTest )
253
254 # Naive Bayes
255 set . seed (123)
256 nbModel <- train ( trainData , trainLabels , method = " nb " , trControl
= tr _ control , metric = " Kappa " )
257 print ( nbModel )
258 # Confusion Matrix
259 nbModelTest <- confusionMatrix ( predict ( nbModel , testData ) ,
testLabels )
260 print ( nbModelTest )
261
262 # Combine results for all models
263 results <- rbind (
264 data . frame (
265 Model = c ( " Neural Networks " , " FDA " , " SVM " , " KNN " , " Naive
Bayes " ) ,
266 Kappa _ Train = c (
267 max ( nnModel $ results $ Kappa ) ,
268 max ( fdaModel $ results $ Kappa ) ,
269 max ( svmModel $ results $ Kappa ) ,
270 max ( knnModel $ results $ Kappa ) ,
271 max ( nbModel $ results $ Kappa )
272 ) ,
273 Kappa _ Test = c (
274 nnModelTest $ overall [ ’ Kappa ’] ,
275 fdaModelTest $ overall [ ’ Kappa ’] ,
276 svmModelTest $ overall [ ’ Kappa ’] ,
277 knnModelTest $ overall [ ’ Kappa ’] ,
278 nbModelTest $ overall [ ’ Kappa ’]
279 )
280 )
281 )
282 print ( results )
283
284 # Plot Variable Importance for NN
285 varImp ( nnModel )
286 importance _ data <- varImp ( nnModel ) $ importance [ , " Overall " ]
287 # Create a bar plot
288 barplot (
289 sort ( importance _ data , decreasing = TRUE ) [1:20] ,
290 las = 2 , # Rotate x - axis labels
291 col = " lightpink " ,
292 main = " Overall Variable Importance ( Top 20 Predictors ) " ,
293 xlab = " Predictors " ,
41
294 ylab = " Importance "
295 )
42
