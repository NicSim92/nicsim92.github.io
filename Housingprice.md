# HDB Price Linear Regression Analysis

<img src="images/hdbcover.png?raw=true" style="width:500px;"/><br>

**Project description:** For this project, it will be a simple demonstration on exploratory data analysis followed by training the model using scikit-learn's linear regression machine learning model. The data was obtained from this <a href="https://data.gov.sg/dataset/price-range-of-hdb-flats-offered">data.gov.sg</a> link on "Price Range of HDB Flats Offered". Data is sorted in "room type", "min and max selling price", "price after grants (AHG/SHG)" and the "town estate".

<br>
<br>
<br>


## 1. Imports

```python
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
```
<br>

The above are libraries are imported:
1. Pandas & Numpy for NumPy for large, multi-dimensional arrays and dataFrames manipulation.
2. Matplotlib & Seaborn for data visualization.

<br>

## 2. Exploratory Data Analysis

```python
df = pd.read_csv('Data Set/price-range-of-hdb-flats-offered.csv')
```
<br>

We first import our .CSV file dataset into our IDE.

<br>

```python
df.isnull().sum()
```
```
financial_year                    0
town                              0
room_type                         0
min_selling_price                 0
max_selling_price                 0
min_selling_price_less_ahg_shg    0
max_selling_price_less_ahg_shg    0
dtype: int64
```
```python
df[df['min_selling_price'] == 0]

df.drop(df[df['min_selling_price'] == 0].index, inplace=True)
```
<img src="images/zerovalue.png?raw=true" style="width:500px;"/><br>

We check for any null values and zero values. From the above we see that a few entries show "0". This could possibly mean that the houses in that region does not exist (e.g 2015, 5-Room flat in Sembawang). This would mean that we can drop the rows for this missing entries.

<br>

```python
sns.displot(data = df, x='min_selling_price',bins=50, hue='financial_year',palette='viridis',height=6)
sns.displot(data = df, x='max_selling_price',bins=50, hue='financial_year',palette='viridis',height=6)
sns.displot(data = df, x='min_selling_price',bins=50, hue='room_type',palette='viridis',height=6)
sns.displot(data = df, x='max_selling_price',bins=50, hue='room_type',palette='viridis',height=6)
```
<img src="images/price1.png?raw=true" style="width:500px;"/><br>
<img src="images/price2.png?raw=true" style="width:500px;"/><br>
<img src="images/price3.png?raw=true" style="width:500px;"/><br>
<img src="images/price4.png?raw=true" style="width:500px;"/><br>

We use seaborn to plot a distribution figure to see the market trend on the selling price due to financial year & room type. There is direct correlation from financial year and room-type, as we see that it increases over the years as well as when the HDB type gets larger.

<br>

## 3. Data Preprocessing
```python
df.insert(3, column='Price', value=df.iloc[:,3:5].median(axis=1))
df.drop(['min_selling_price','max_selling_price'],inplace=True,axis=1)
df.insert(4, column='ahg_shg_price', value=df.iloc[:,4:6].median(axis=1))
df.drop(['min_selling_price_less_ahg_shg','max_selling_price_less_ahg_shg'],inplace=True,axis=1)
df.insert(4, column='ahg_shg_reduce', value=df['Price'] - df['ahg_shg_price'])
df.drop(['ahg_shg_price'],inplace=True,axis=1)
df.drop(df[df['Price'] == df['ahg_shg_reduce']].index, inplace=True)
df[df['Price'] == df['ahg_shg_reduce']]
```
<img src="images/adjusted123.png?raw=true" style="width:500px;"/><br>
The above code appears to be inefficient and messy. The main purpose was to combine both the min and max selling prices (as well as the min and max AHG/SHG grant prices) to get a single median value per row. This column with the new median values are added into the dataframe and the original columns dropped. The amount of AHG / SHG grant is also obtained from the above tabulation. This reduction of columns allows us to identify the factor on price, without reviewing on two price columns that serves the same purpose.

<br>

```python
town_dummied = pd.get_dummies(df['town'], prefix='town', drop_first=True)
room_type_dummied = pd.get_dummies(df['room_type'], prefix='room_type', drop_first=True)

df = pd.concat([df, town_dummied], axis=1)
df = pd.concat([df, room_type_dummied], axis=1)
df.drop(['town','room_type'],axis=1,inplace=True)
```
<img src="images/adjusted456.png?raw=true" style="width:800px;"/><br>
Since the town and room columns are in categorical values we apply the get_dummies on these columns to convert categorical variable into dummy/indicator variables so that the linear regression model can be applied effectively on the dataset. 

The dataset columns have increased to a total of 15.

## 4. Training Model
```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression

X = df.drop('Price',axis=1)
y = df['Price']
X_train, X_test, y_train, y_test = train_test_split(X,y,test_size=0.3,random_state=101)
```
```python
 lm = LinearRegression()
 lm.fit(X_train,y_train)
 predictions = lm.predict(X_test)
```
<br>

We import the associated libraries for training, testing and splitting the data as well as the linear regression machine learning model from scikit-learn. Applying the following code to split the data for training and testing, then fitting the training set to the model.
The model is generated and we use the test set to perform our predictions.

<br>

## 5. Results
```python
plt.scatter(y_test,predictions)
plt.xlabel('Y Test')
plt.ylabel('Predicted Y')
plt.savefig('result2.png')
```
<img src="images/result2.png?raw=true" style="width:500px;"/><br>
We plot the figure above and the predicted results shows a direct linear relation with the actual data. 

<br>

```python
from sklearn import metrics
from sklearn.metrics import r2_score

print('MAE:', metrics.mean_absolute_error(y_test, predictions))
print('MSE:', metrics.mean_squared_error(y_test, predictions))
print('RMSE:', np.sqrt(metrics.mean_squared_error(y_test, predictions)))
print('R2:', r2_score(y_test,predictions))

MAE: 11690.954481118963
MSE: 222145056.78074107
RMSE: 14904.531417684391
R2: 0.9750582954996662
```
<br>

Results using mean absolute error,mean squared error, root mean square error and R2 score shows that the machine learning model is performing accordingly. The variance, based on the presented errors above, are relatively small as compared to the actual selling.

<br>

```python
sns.displot((y_test-predictions),bins=50,kde=True);
```
<img src="images/result.png?raw=true" style="width:500px;"/><br>
Above figure shows that residual error occurs largely within -10000 to 10000 range.

<br>

```python
coeffecients = pd.DataFrame(lm.coef_,X.columns)
coeffecients.columns = ['Coeffecient']
coeffecients
```
<img src="images/coeff.png?raw=true" style="width:200px;"/><br>
From the coeffecients, we can interpret that:

* e.g. Holding all other features fixed, a 1 unit increase in financial year is associated with an increase of 3,676 in selling price.

<br>
<br>
<br>

<a href="https://nicsim92.github.io/" style="font-size: 30px">Return to Main Selection Page</a>





