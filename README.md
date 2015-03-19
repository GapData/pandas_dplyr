# Super-short super-basic Data Munging in R and Python

*Giuseppe Paleologo* <paleologo@gmail.com>

2015-04-01

In October 2012 I wrote a document to help translate between R and python pandas idioms for data manipulation. Back then, It was reasonable to cover a good fraction of the operations in the two languages. Today that is no longer the case. Pandas has tripled in size since I first covered its capabilities and is now nearing 200K lines of code. Meanwhile, R has seen intense development of data.table and the release of dplyr, tidyr and magrittr, steps toward a "grammar of data". Pandas and dplyr take different approaches to data manipulation. Pandas takes the "maximalist" approach: it works with 1, 2, and 3-dimensional arrays; and with hierarchical indices, one can represent tree-like data structures (e.g., nested lists); moreover, it allows one to align and operate on numerical arrays. Finally, it has plotting capabilities as well. dplyr takes the "minimalist" approach. It works with 2-D tables only, with a small set of verbs that are easily composable with each other (as well as with base R functions). As a result, the manipulations are easy to write and read. Plotting is delegated to other libraries; so is numerical algebra; so is time series analysis. To learn well the data manipulation capabilities alone of these two languages requires a significant time investment. To reflect these changes, I chose to focus on manipulation of tabular data alone, therefore using a subset of pandas for Python, and of dplyr + tidyr in R. The choice of  the latter is motivated by its balance of conceptual elegance, performance and fast rate of adoption. Pandas' documentation has a session comparing R to pandas, but the idioms presented here are more modern, regular and concise.

I make no claims of completeness, but hope that readers will find this useful to go back and forth between the languages.

## Preliminaries 
###### Python
```python
# python preliminaries
import numpy as np
randn = np.random.randn
from pandas import *
```
###### R
```R
# R preliminaries
library(dplyr)
library(tidyr)
library(data.table) # used only for fread()
```
## Reading a dataFrame from file or URL
The functions `read_csv` and  `fread` have similar performance. They attempt to infer from data field separator and field type. 
###### Python
```python
DF = read_csv(filepath)
```
###### R
```R
DF <- fread(filepath)
```
Both pandas and R offer functions to access DMBS (though SQLAlchemy for Pandas, specific packages for R) and a variety of formats (e.g., Json).

##Generating a dataframe from existing data
###### Python
```python
# From dictionary
x = {'city': ['Rome', 'New York', 'Moscow'], 
     'continent': ['Europe', 'America', 'Europe'], 
     'inhabitants': [8.4e6, 2.8e6, 11.5e6]}
DF = DataFrame(x)

# from recarray
m = np.zeros((2,), dtype=[('A', 'i4'),('B', 'f4'),('C', 'a10')])
m[:] = [(1,2.,'Hello'),(2,3.,"World")]
DataFrame(m)

# from ndarray
m = np.arange(12.).reshape((3, 4))        
DataFrame(m, columns=['a', 'b', 'c', 'd'])
```
###### R
```R
# native
DF <- data.frame(city=c('New York', 'Rome', 'Moscow'), 
                 continent = c('Europe', 'America', 'Europe'), 
                 inhabitants=c(8.4e6, 2.8e6, 11.5e6))
# From list
x <- list(city=c('New York', 'Rome', 'Moscow'), 
          continent = c('Europe', 'America', 'Europe'), 
          inhabitants=c(8.4e6, 2.8e6, 11.5e6))
DF <- as.data.frame(x)

# from matrix
m <- matrix(1:12, ncol=2)                         
colnames(m) <- letters[1:3]
as.data.frame(m)
```
## Filtering rows
###### Python
```python
DF.ix[DF.inhabitants > 5e6] 
DF.ix[[x in ['Europe', 'Asia'] for x in DF.continent]]
```
###### R
```R
filter(DF, inhabitants > 5e6)
filter(DF, continent %in% c('Europe', 'Asia'))
```
## Filtering columns 
###### Python
```python
DF[['city','continent']]
```
###### R
```R
select(DF, city, continent)
```
## Deleting columns
###### Python
```python
del DF['city']
```
###### R
```R
DF[, 'city'] <- NULL
select(DF, -city)
```
## Jointly filtering rows and columns 
###### Python
```python
DF.ix[[True, False, True], ['city', 'continent']]
```
###### R
```R
# base R; dplyr combines filter and select
DF[c(TRUE, FALSE, TRUE), c('city', 'continent')] 
```
## Setting Fields to to NA
###### Python
```python
DF.ix[[True, False, True], ['city', 'continent']] = np.nan
```
###### R
```R
DF[c(TRUE, FALSE, TRUE), c('city', 'continent')] = NA
```
## Checking for (non)missing values 
###### Python
```python
DF.isnull()      # missing
isnull(DF)       # alternative
DF.notnull()     # non missing
notnull(DF)      # alternative
```
###### R
```R
is.na(DF)        # missing
!is.na(DF)       # non missing
```
## dropping missing values
###### Python
```python
DF.dropna()
```
###### R
```R
na.omit(s)   
```
## Replacing missing values
###### Python
```python
DF.fillna(-1)
```
###### R
```R
DF[is.na(DF)] <- -1
```
## Head and Tail 
###### Python
```python
DF.head(2)  # default is 5 rows
DF.tail(2)
```
###### R
```R
head(DF, 2)  # default is 6 rows
tail(DF, 2)
```
## Updating/adding columns 
###### Python
```python
DF['birthdate'] = [-621, 1625, 1147]
```
###### R
```R
DF[,'birthdate'] <- c(-621, 1625, 1147)
DF %<>% mutate(birthdate = c(-621, 1625, 1147))
```
## Renaming columns 
###### Python
```python
DF.columns =[x.upper() for x in list(DF.columns)]
```
###### R
```R
names(DF) <- toupper(names(DF))       # base R
DF %<>% set_names(toupper(names(.)) 
```
## Concatenating rows 
###### Python
```python
DF_list = [DF, DF]
concat(DF_list)
```
###### R
```R
rbind_all(DF, DF)
rbind_list(list(DF, DF))
```
N.B.: Pandas allows data with different column names and types to be concatenated. Moreover, pandas allows the creation of a hierarchical index for the combined data frame. See the section "Introduction to Pandas indices".
## Remove duplicated rows
###### Python
```python
s = DataFrame({'a' : [0., .0, 1., 2.]}
s.duplicated()
s.drop_duplicates()
```
###### R
```R
s = data.frame(a = c(0., .0, 1., 2.))
distinct(s)   # in base R, unique(s)
```
## Joining
###### Python
```python
merge(DF1, DF2, how='left',  on=['colname1', 'colname2'])
merge(DF1, DF2, how='right', on=['colname1', 'colname2'])
merge(DF1, DF2, how='inner', on=['colname1', 'colname2'])
merge(DF1, DF2, how='outer', on=['colname1', 'colname2'])
```
###### R
```R
inner_join(DF1, DF2, by = c('colname1', 'colname2')) 
left_join (DF1, DF2, by = c('colname1', 'colname2'))
right_join(DF1, DF2, by = c('colname1', 'colname2'))
full_join (DF1, DF2, by = c('colname1', 'colname2'))
```
dplyr has also a semi_join and anti_join.
## Sorting
###### Python
```python
DF.sort('city', ascending=False)
DF.sort('city', ascending=True)
DF.sort(['city', 'continent'], ascending=True)
```
###### R
```R
DF %>% arrange(desc(city))
DF %>% arrange(city)
DF %>% arrange(city, continent)
```
## Summarizing
###### Python
```python
DF.describe()
```
###### R
```R
summary(DF)
```
## Converting to arrays for numerical computation
Pandas and R/dplyr differ substantially on this account. In R, it is strongly inadvisable to perform binary operations on data frames. The user should convert the data to a suitable n-way array  and then perform operations. Conversely, Pandas is fully able to correctly perform operations on the underlying numpy object, with the added benefit of automatic alignment. mplyr is a package that does alignment on n-way arrays in R.
###### Python
```python
DF[['inhabitants','birthdate']].as_matrix()

# an example of how Pandas takes care of automatic alignment
m1 = np.arange(12.).reshape((3, 4))        
df1 = DataFrame(m1, columns=list('abcd'))
m2 = (np.arange(20)+5).reshape((4, 5))               
df2 = DataFrame(m2, columns=list('abcde'))
df1 + df2   # takes union of indices and columns, NaN applied

as.matrix(DF[, c('inhabitants','birthdate')])
```
## Splitting, applying, combining
###### Python
```python
# converts the index to column fields
DF_grouped = DF.groupby('city',  as_index=False)
DF_grouped.agg({'C' : np.sum, 'D' : lambda x: np.std(x, ddof=1)})
DF_grouped['C'].agg(np.sum)
grouped['C'].agg({'result1' : np.sum, 'result2' : np.mean})
# sugared expression
DF_grouped.std()
```
## Converting a dataframe from wide to long format
###### Python
```python
cheese = DataFrame({'first' : ['John', 'Mary'], 
                    'last' : ['Doe', 'Bo'], 
                    'height' : [5.5, 6.0], 
                    'weight' : [130, 150]})
pd.melt(cheese, id_vars=['first', 'last'])
```
###### R
```R
cheese <- data.frame(first = c('John', 'Mary'), 
                     last = c('Doe', 'Bo'), 
                     height = c(5.5, 6.0), 
                     weight = c(130, 150))
cheese %>% gather(feature, value,-first,-last)
```
## Converting a dataframe from long to wide format
###### Python
```python
df = DataFrame({'Animal': ['Animal1', 'Animal2', 'Animal3', 'Animal2', 
                           'Animal1', 'Animal2', 'Animal3'], 
                'FeedType': ['A', 'B', 'A', 'A', 'B', 'B', 'A'], 
                'Amount': [10, 7, 4, 2, 5, 6, 2], })
df.pivot_table(values='Amount', index='Animal', columns='FeedType', aggfunc='sum')
```
###### R
```R
df <- data.frame(
  Animal = c('Animal1', 'Animal2', 'Animal3', 'Animal2', 'Animal1',
             'Animal2', 'Animal3'),
  FeedType = c('A', 'B', 'A', 'A', 'B', 'B', 'A'),
  Amount = c(10, 7, 4, 2, 5, 6, 2)
)
# with tidyr
df %>% spread(Animal, FeedType)
# using base R
with(df, tapply(Amount, list(Animal, FeedType), sum, na.rm = TRUE))
```
## Casting to an Array
###### Python
```python
df = pd.DataFrame({x': np.random.uniform(1., 168., 12), 
                  'y': np.random.uniform(7., 334., 12), 
                  'z': np.random.uniform(1.7, 20.7, 12), 
                  'month': [5,6,7]*4, 'week': [1,2]*6}) 
mdf = pd.melt(df, id_vars=['month', 'week'])
```
###### R
```R
pd.pivot_table(mdf, 
               values='value', 
               index=['variable','week'], 
               columns=['month'], aggfunc=np.mean)
df <- data.frame(x = runif(12, 1, 168),
                 y = runif(12, 7, 334),
                 z = runif(12, 1.7, 20.7),
                 month = rep(c(5,6,7),4),
                 week = rep(c(1,2), 6)
)
mdf <- melt(df, id=c("month", "week"))
acast(mdf, week ~ month ~ variable, mean)
```
## Applying functions to an entire data frame or column 
###### Python
```python
np.exp(df[['x']])  
```
###### R
```R
exp(df$x)  
```
## Applying elementwise functions
```python
s = Series(["little", "red", "fox"])     
s.map(len)          # notice that map(len, s) returns a list
```
###### R
```R
s <- c("little", "red", "fox")      
sapply(s, nchar)    # nchar(s) would have worked here
```
Notice the difference: pandas aligns by name, taking the union of indices. Base R does not take names into account. In R, the typical process would be to join the vectors in a data frame and then operating on the columns.
## A self-contained example: baby names

I close with a concrete example: the "baby names" data set made famous by Martin Wattenberg and Fernanda Viégas. The python code below is taken verbatim from Wes McKinney's book "Pandas for Data Analysis". The R code is a translation of the same analysis in R. A few distinguishing features stand out. First, python used method extensively whereas R uses functions. It is possible to chain methods but readability is not greatly enhanced. In R, all the operations can be performed by a single chain. Second, R delegates all the plotting functions to ggplot2, which uses a syntax similar to dplyr (but with a "+"; ggvis, the successor to ggplot, uses the familiar %>%"). Third, all R analysis uses "long data frame". This is an instance of "tidy data". Python uses wide format data frames for plotting (not unlike R's the ones matplot() would require. Lastly, the same few functions show up in R over and over: `group_by()`, `summarize()`, `is`, `reindex`, `arrange()`, `mutate()`, `filter()`. Pandas has a larger vocabulary, with `groupby`, `apply`, `sortindex`, `searchsorted`, `unstack`, `map`, `pivot_table` methods; many of which take further arguments; but it is at the same time slightly less verbose.
### Read Files
