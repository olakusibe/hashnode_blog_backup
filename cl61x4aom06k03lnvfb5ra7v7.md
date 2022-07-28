## Solving AttributeError: 'float' object has no attribute 'rint'

#### Problem
Lately, I have been working on some ETL projects and during the transform stage I come across the error `"AttributeError: 'float' object has no attribute 'rint'"`, combined with `"TypeError: loop of ufunc does not support argument 0 of type float which has no callable rint method"`.

#### Explanation
It took a while but it became obvious that as I get/extract data from many sources (CSV, JSON, or XML), similar columns might have a mixture of the datatype (i.e a column may have a string and int data type mixed), and before I can proceed to the transform stage, I would need that column to have a designated datatype, especially if I want to perform other tasks on it (which was what I was trying to do and kept encountering the errors).

#### Solution
To convert the column with a mixed data type, you will need to cast it to the designated data type you intend to work with.

This can be achieved by using the `pandas.DataFrame.astype` function, as illustrated below

```python
import pandas as pd
 
# an example of extracted data
extracted_data = pd.DataFrame({
    'Brand': ['Gucci', 'Nike', 'Adidas', 'Hermes', 'Zara'],
    'year_of_manufacture': [2010, 1999, '2012', 2011, '2008'], # mixed data type
    'price': ['4034.203', 5000.00, '7450.17567', 3023.004, '4901.32345'] # mixed data type
})
 
# cast specific columns to a desired data type
cast_to_type = {
    'year_of_manufacture': int,
    'price': float
    }
extracted_data = extracted_data.astype(cast_to_type)

# do the transfrom
extracted_data['price'] = extracted_data['price'].round(2)

print(extracted_data)

```

#### For Further Reading
1. [pandas.DataFrame.astype](https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.astype.html) function
2. [Other ways to cast](https://www.geeksforgeeks.org/change-data-type-for-one-or-more-columns-in-pandas-dataframe/) mixed data type in a list to a specific data type