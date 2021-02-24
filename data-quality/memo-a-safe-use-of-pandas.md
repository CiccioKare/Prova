---
description: '#pandas #csv #read'
---

# Memo - A safe use of Pandas

This template has the purpose to give some advice about reading csv files with pandas without bumping into common "errors", or unexpected behaviours. 

For each problem, we describe the **task**, the **expected behaviour** \(what we expect pandas does while doing that task\), the **real behaviour** \(what pandas actually does, breaking the "common sense"\) and a **proposed solution** \(if the solution is to complicated to write in a template, we refer to one or two colleagues who have encountered that behaviour and handled it\).

#### 1. Reading Naples province abbreviation \(NA\)

* **Task**: read a csv file where a column contains the italian abbreviations of provinces \(for Naples it is NA\)
* **Expected behaviour**: read the Naples province as string NA \(not as null value\)
* **Actual behaviour**: in place of string NA, pandas reads NaN \(null value\)
* **Solution proposed**: in pandas.read\_csv\(\) set the optional arguments keep\_default\_na=False and na\_values=\[‘\#N/A’, ‘\#N/A N/A’, ‘\#NA’, ‘-1.\#IND’, ‘-1.\#QNAN’, ‘-NaN’, ‘-nan’, ‘1.\#IND’, ‘1.\#QNAN’, ‘’, ‘N/A’, ‘NULL’, ‘NaN’, ‘n/a’, ‘nan’, ‘null’\]

#### 2. Reading Integer column with missing values

* **Task**: read a csv file where a column contains integer values with some missing values
* **Expected behaviour**: the column has integer type \(numpy.int64 for example\) ad the missing values are handled
* **Actual behaviour**: numpy.int64 does not handle missing values; so, the column dtype is switched to float64
* **Solution proposed**: in pandas of version &gt;= 0.24, there is the "Int64" dtype \(pandas.Int64Dtype\(\), _Attention_: mind the initial capital "I"\) so, after having read the file, you can set the type df\[column\] = df\[column\].astype\('Int64'\). See more about pandas dtypes handling missing values [here](https://pandas.pydata.org/pandas-docs/stable/user_guide/gotchas.html#support-for-integer-na).

**3. Reading a csv \(possibly corrupted\) file with incomplete rows \(a lower number of separators in some rows\)**

* **Task**: read a csv file where some rows contains a number of separators \(and so, of fields\) lower than in the header \(less than expected\)
* **Expected behaviour**: an Exception will be raised
* **Actual behaviour**: no Errors or Exceptions are raised and the DataFrame will have missing \(NaN\) values in place of the not present fields
* **Solution proposed**: read the csv file line by line \(with csv.reader\(\)\) and for each line count the number of separators; if it is not what we expect, raise an Exception, that will be reported to the client \(or the owner of the csv file\)

**4. Reading a csv containing a date column using infer\_datetime\_format**

* **Task**: read a csv file containing a "date" column, trying to parse the date directly while reading, in order to get a datetime column and not an object column \(use pd.read\_csv\(file, parse\_date=\[date\_col\_name\], infer\_datetime\_format=True\)
* **Expected behaviour**: the same datetime format is used for each row; ie if in row 1 I have 01-10-2020 and in row 10 I have 22-10-2020, I expect to get a format of this kind: %d-%m-%Y. 
* **Actual behaviour**: I get the format %m-%d-%Y where possible \(for example in the first row of the example before\) but the format %d-%m-%Y where the first one is not possible, using two different formats for two different rows
* **Solution proposed**: read without inferring datetime \(as an object column\) and then use df\[date\_col\_name\] = pd.to\_datetime\(df\[date\_col\_name\], format = %d-%m-%Y"\). _Note_: in to\_datetime you need to specify the format, and cannot use the default \(it gives the same error as infer\_datetime\_format!\)

**5. Interpolate values in a column setting a limit**

* **Task**: you have a pandas Series with some values and some missing points \(example: a time series\) and you want to fill the missing values with a \(linear\) interpolation, but you are interested in filling only maximum N consecutive missing point at a time; if the missing points consecutively are more than N, you want to keep the missing values. For doing that, you decide to use pd.Series.interpolate\(limit = N\)
* **Expected behaviour**: if the consecutive missing points are more than N, I expect they \(ALL\) remain missing
* **Actual behaviour**: pandas makes the interpolation for the first N missing points, and keep the NaN value for the remaining
* **Solution proposed**: ask Andrea Giannetti and Livio Cascone!

Anyone who encounters any other bugs in the common use of pandas, is encouraged to expand this list!



Written by Filippo Miti

