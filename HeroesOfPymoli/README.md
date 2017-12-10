

```python
# This pandas script reads an input file containing sales data, and produces eigth different reports summarizing these data.
# Data in the input file are assumed to be in json format.
# The input file is assumed to be in the same directory where this script resides.
#
# The code for each report is contained in a single cell and includes comments that detail its logic and operations.
#
# At a high level, each of these cells defines a data frame corresponding to the report it produces. The elements of these
#   data frames are calculated/filled-in as expalined in the comments throughout this script.
#
```


```python
# There is only one dependency: pandas
import pandas as pd
```


```python
# Read the input file into a data frame for subsequent processing and analysis
file_path='purchase_data.json'
data_df=pd.read_json(file_path)
data_df.head()   # output the first five rows of input data for future reference
```


```python
# Rename column 'Item ID' to 'Item_ID' to convert the space character to an underscore (_); this facilitates easier
#   reference to this column in code cells below.
data_df=data_df.rename(columns={"Item ID":"Item_ID"})
```


```python
# This first report produces a single data point: total number of players present in the input file.
# This value corresponds to the number of unique instances in the SN (screen name) column and is determined by using
#   method nunique().
#
total_unique_players=data_df['SN'].nunique()
Total_Players_df=pd.DataFrame({"Total Players":[total_unique_players]})
Total_Players_df.head()  # output is passed through head() to produce consistent output with other reports below
```


```python
# This code cell conducts an overall Purchasing Analysis (Total) and produces the following output:
#   Number of unique items, Avg purchase price, Total number of purchases and Total revenue
#
# Start with determining various high level data points, such as number of unique items and number of purchases.
#
num_unique_items = data_df['Item_ID'].nunique()  # Use nunique() on Item_ID column to determine no. of unique items
num_purchases = len(data_df.index)               # Number of purchases is simply the length of the overall data frame index
average_price= data_df['Price'].mean()           # Average price is determined using mean() method on the Price column
total_rev = average_price * num_purchases        # Total revenue is the product of average price and number of purchases

# Now store all the above values into a data frame for final formatting and output:
PurchaseAnalysisTotal_df = pd.DataFrame ({"Number of Unique Items" : [num_unique_items],
                               "average_price" : [average_price],
                               "Number of Purchases" : [num_purchases],
                               "total_revenue" : [total_rev]
                               })
# Fix the formatting and the order of columns, to match the format provided in the example:
PurchaseAnalysisTotal_df ['Average Price'] = PurchaseAnalysisTotal_df['average_price'].map('${:,.2f}'.format)
PurchaseAnalysisTotal_df ['Total Revenue'] = PurchaseAnalysisTotal_df['total_revenue'].map('${:,.2f}'.format)
PurchaseAnalysisTotal_df = PurchaseAnalysisTotal_df [['Number of Unique Items', 'Average Price', 'Number of Purchases', 'Total Revenue']]

# And print the resulting data frame:
PurchaseAnalysisTotal_df.head()
```


```python
# This code cell conducts an overall Geder Demographics analysis and produces the following output:
#   Percentage & count of male, female and other/non-disclosed players.
#
# Start with grouping the overall data by gender, using groupby(), and determining various high level data points such as 
#   the number of various geneder groups.
#
gender_groups = data_df.groupby('Gender')

# Capture the number of players of each gender using count(). The first element in the count() output corresponds to females,
#    the second element corresponds to males, and the third element corresponds to other/un-disclosed.
num_female_players, num_male_players, num_other_players = gender_groups['Gender'].count()
total_players = num_female_players + num_male_players + num_other_players

# Now store all the above data into a data frame for final formatting and output:
Gender_df = pd.DataFrame ({"Male" : [num_male_players, (num_male_players/total_players)*100],
                           "Female" : [num_female_players, (num_female_players/total_players)*100],
                           "Other / Non-Disclosed" : [num_other_players, (num_other_players/total_players)*100]
                              })
# Transpose the rows and columns of the report data frame, and fix the order of columns to match the format provided 
#    in the example:
Gender_df = Gender_df.T
Gender_df = Gender_df.rename(columns={0:"Total Count"})
Gender_df = Gender_df.rename(columns={1:"Percentage of Players"})
Gender_df = Gender_df [['Percentage of Players', 'Total Count']]
# And print the resulting data frame, passing it through head() for consistent formatting
Gender_df.sort_values(by='Total Count', ascending=False).head()
```


```python
# This code cell conducts a gender-based Purchasing Analysis (Gender), producing the following output for each geneder group:
#   Count of Purchases, Average Selling price, Total Purchase Value and Normalized Total.
#
# Start with building data frames for each gender group, containing report data points for each group.
# Build each geneder-group data frame using loc operator, to find all rows for a given group in the overall data frame.
#
female_activity_df = data_df.loc[data_df['Gender'] == 'Female',:]
female_purchase_count = len(female_activity_df.index)
female_purcahse_avg_price = female_activity_df['Price'].mean()
female_purchase_total = female_purcahse_avg_price * female_purchase_count
female_purchase_normalized = female_purchase_total / num_female_players

male_activity_df = data_df.loc[data_df['Gender'] == 'Male',:]
male_purchase_count = len(male_activity_df.index)
male_purcahse_avg_price = male_activity_df['Price'].mean()
male_purchase_total = male_purcahse_avg_price * male_purchase_count
male_purchase_normalized = male_purchase_total / num_male_players

other_activity_df = data_df.loc[data_df['Gender'] == 'Other / Non-Disclosed',:]
other_purchase_count = len(other_activity_df.index)
other_purcahse_avg_price = other_activity_df['Price'].mean()
other_purchase_total = other_purcahse_avg_price * other_purchase_count
other_purchase_normalized = other_purchase_total / num_other_players

# Now store all the above data points into a data frame for final formatting and output:
PurchaseAnalysisGender_df = pd.DataFrame ({
                               "Male" : [male_purchase_count, male_purcahse_avg_price, male_purchase_total, male_purchase_normalized],
                               "Female" : [female_purchase_count, female_purcahse_avg_price, female_purchase_total, female_purchase_normalized],
                               "Other / Non-Disclosed" : [other_purchase_count, other_purcahse_avg_price, other_purchase_total, other_purchase_normalized]
                              })

# Transpose the rows and columns of the report data frame and rename columns for better code readability:
PurchaseAnalysisGender_df = PurchaseAnalysisGender_df.T
PurchaseAnalysisGender_df = PurchaseAnalysisGender_df.rename(columns={0:"Purchase Count"})
PurchaseAnalysisGender_df = PurchaseAnalysisGender_df.rename(columns={1:"avg_price"})
PurchaseAnalysisGender_df = PurchaseAnalysisGender_df.rename(columns={2:"tot_value"})
PurchaseAnalysisGender_df = PurchaseAnalysisGender_df.rename(columns={3:"normalized"})

# Note that columns containing currency data points are of type float; these columns need need to be reformatted (and 
#    effectvely converted to strings) so they are diaplayed in the same manner as in the sample file. Hence, the need for 
#    additional columns which will contain properly formatted currency figures:
PurchaseAnalysisGender_df ["Average Purchase Price"] = PurchaseAnalysisGender_df["avg_price"].map('${:,.2f}'.format)
PurchaseAnalysisGender_df ["Total Purchase Value"] = PurchaseAnalysisGender_df["tot_value"].map('${:,.2f}'.format)
PurchaseAnalysisGender_df ["Normalized Totals"] = PurchaseAnalysisGender_df["normalized"].map('${:,.2f}'.format)
# Now drop columns containing currency figures saved as type float, and produce the final report output
PurchaseAnalysisGender_df = PurchaseAnalysisGender_df [['Purchase Count', 'Average Purchase Price', 
                                                        'Total Purchase Value', 'Normalized Totals']]

PurchaseAnalysisGender_df.head()
```


```python
# This code cell conducts an age-based (Age Demographics) analysis and produces a report containing:
#   Purchase Count, Average Purchase Price, Total Purchase Value and Normalized Totals.
#
# Start the process by binning all the input data into 8 age groups each spanning four years.
#   This is done using the cut() method.  Aadd a new column which contains the corresponsing age group label for 
#   each record in the input data.
#
age_bins = [0, 10, 15, 20, 25, 30, 35, 40, 100]  # The last bin is set to a high value , 
bin_labels = ['<10', '10-15', '15-20', '20-25', '25-30', '30-35', '35-40', '40+']
data_df['Age Bin']=pd.cut(data_df['Age'], age_bins, labels=bin_labels)
age_group_purchases = data_df.groupby('Age Bin')  # Group all the input data using this freshly added coloumn

# Define the report data frame for final output, and initialize various fields to zero's.  Note that columns containing
#   currency info are of type float and will need to be converted/formatted for proper display.
# There are a total of 8 age groups, hence each column is initialized with a list comprised of 8 zeros:
PurchaseAnalysisAge_df = pd.DataFrame ({
                               "Purchase Count" : [0,0,0,0,0,0,0,0],
                               "avg_price" : [0,0,0,0,0,0,0,0],
                               "tot_purchase_val" : [0,0,0,0,0,0,0,0],
                               "normalized" : [0,0,0,0,0,0,0,0]
                              })
# Start calculating the values for various fields in the report data frame:
#
PurchaseAnalysisAge_df['avg_price'] = age_group_purchases['Price'].mean().values
PurchaseAnalysisAge_df['Purchase Count'] = age_group_purchases['Price'].count().values
PurchaseAnalysisAge_df['tot_purchase_val'] = PurchaseAnalysisAge_df['avg_price'] * PurchaseAnalysisAge_df['Purchase Count']
PurchaseAnalysisAge_df['Age Bracket Count'] = data_df['Age Bin'].value_counts().values
PurchaseAnalysisAge_df['normalized'] = PurchaseAnalysisAge_df['tot_purchase_val'] / PurchaseAnalysisAge_df['Age Bracket Count']

# create new columns which contain reformatted data of type float, to properly display currency info:
PurchaseAnalysisAge_df ['Average Price'] = PurchaseAnalysisAge_df['avg_price'].map('${:,.2f}'.format)
PurchaseAnalysisAge_df ['Total Purchase Value'] = PurchaseAnalysisAge_df['tot_purchase_val'].map('${:,.2f}'.format)
PurchaseAnalysisAge_df ['Normalized Totals'] = PurchaseAnalysisAge_df['normalized'].map('${:,.2f}'.format)

# Finally, select and set the order of columns, to match the sample file:
PurchaseAnalysisAge_df = PurchaseAnalysisAge_df [['Purchase Count', 'Average Price', 'Total Purchase Value', 'Normalized Totals']]
PurchaseAnalysisAge_df.index = bin_labels
# Output the report by sending it through head(10) since there are 8 actual records to display:
PurchaseAnalysisAge_df.head(10)
```


```python
# This code cell produces a report of Top 5 Spenders based on the total purchase value, including:
#   SN, Purchase Count, Average Purchase Price, Total Purchase Value.
#
# Start by grouping all the input data based on buyers screen name (SN), using groupby() method.
#
# Calculate various fields using count() and mean() methods applied to the groupby object:
#
sales_by_SN = data_df.groupby('SN')
SN_sales_df = pd.DataFrame({'Purchase Count' : sales_by_SN['Item_ID'].count(),
                            'avg_price' : sales_by_SN['Price'].mean(),
                            'tot_purchase' : sales_by_SN['Item_ID'].count() * sales_by_SN['Price'].mean()
})

# Create new columns which contain reformatted data of type float, to properly display currency info:
SN_sales_df ['Average Purchase Price'] = SN_sales_df['avg_price'].map('${:,.2f}'.format)
SN_sales_df ['Total Purchase Value'] = SN_sales_df['tot_purchase'].map('${:,.2f}'.format)

# Select and set the order of columns, to match the sample file:
SN_sales_df = SN_sales_df [['Purchase Count', 'Average Purchase Price', 'Total Purchase Value']]

# Produce the final report by sorting the report data frame based on Total Purchase Value, and outputing the first
#   five rows using head():
SN_sales_df.sort_values(by='Total Purchase Value', ascending=False).head()
```


```python
# This code cell produces the 5 Most Popular Items based on purchase count; the report includes:
#   Item ID, Item Name, Purchase Count, Item Price, Total Purchasse Value
#
# Start the process by grouping all the input data by Item_ID, using groupby() method:
item_sales = data_df.groupby('Item_ID')

# The code below produces a dictionary which maps Item_ID to Price; this dictionary is then used in building the report data frame.
# First define a function which accepts and item id and returns its corresponding price. Price is looked up in the
#   input data, which returns a Series. The first element of this series contains the price.
# Build the dictionary by iterating over all the unique instances of Item_ID in the input data, and retreive its corresponding
#   price using the function described above:
item_prices = []  # define empty dictionary; key is item_id and value is its price
def return_price(itemid):
    return data_df.loc[data_df.Item_ID==itemid]['Price'].iloc[0]
for item in data_df['Item_ID'].unique():
    item_prices.append(return_price(item))

# Define the report data frame and populate its various fields using unique() and count() methods:
item_sales_df = pd.DataFrame ({'Item Name' : item_sales['Item Name'].unique(),
                               'Purchase Count' : item_sales['Item_ID'].count().values,
                               'price' : item_prices,
                               'tot_pvalue' : item_sales['Item_ID'].count().values * item_prices
                              })
# Save the data frame with numeric values for the next report below
item_sales_numeric = item_sales_df

# Create new columns which contain reformatted data of type float, to properly display currency info:
item_sales_df ['Item Price'] = item_sales_df['price'].map('${:,.2f}'.format)
item_sales_df ['Total Purchase Value'] = item_sales_df['tot_pvalue'].map('${:,.2f}'.format)

# Select and set the order of columns, to match the sample file:
item_sales_df = item_sales_df [['Item Name', 'Purchase Count', 'Item Price', 'Total Purchase Value']]

# Produce the final report by sorting the report data frame based on Purchase Count, and passing the result to head() which
#   outputs the first 5 rows.
item_sales_df.sort_values(by='Purchase Count', ascending=False).head()
```


```python
# This code cell produces the 5 most profitable items by total purchase value; the report includes:
#   Item ID, Item Name, Purchase Count, Item Price, Total Purchase Value
# This report uses the data frame saved above:
#
# First sort this data frame which contains numeric values for total purchase values:
item_sales_df = item_sales_numeric.sort_values(by='tot_pvalue', ascending=False)

# Select and set the order of columns, to match the sample file:
item_sales_df = item_sales_df [['Item Name', 'Purchase Count', 'Item Price', 'Total Purchase Value']]

# Produce the output by passing the report data frame to head() (the data frame is already sorted):
item_sales_df.head()
```
