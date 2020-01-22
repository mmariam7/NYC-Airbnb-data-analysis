
## NYC Airbnb Data Analysis
## Mariam Mohamed
## January 21, 2020


## Introduction
    The purpose of this project is to identify which areas and zipcodes in New York City is the best to invest in a property that will be rented out through the Airbnb platform. This study is to help a real estate that has a niche in purchasing properties to rent out short-term as part of their business model specifically within New York City. The real estate company has already concluded that two bedroom properties are the most profitable; however, they do not know which zip codes are the best to invest in. Our analysis will aid in helping them make a decision based on the data. The data analysis was conducted primarily using Python.    

## Data
There are two diffrent sources of data that will be used in this project. 
The two sources of data are:
1.	AirBnB data:  Download http://data.insideairbnb.com/united-states/ny/new-york-city/2019-07-08/data/listings.csv.gz
2.	Zillow data:  Zip_Zhvi_2bedroom.csv.zip

## Airbnb Dataset
This data has information on the listing including location, number of bedrooms, room types (entire home/private home/shared home),reviews, zipcode and neighborhood. 

## Zillow Dataset

The Zillow data set has Cost data to determine the average property price for 2 bedrooms.

## Assumptions
* The occupancy rate is assumed to be 75%

* The investor will pay for the property in cash (i.e. no mortgage/interest rate will need to be accounted for). 

* The time value of money discount rate is 0% (i.e. $1 today is worth the same 100 years from now). 

* All properties and all square feet within each locale can be assumed to be homogeneous (i.e. a 1000 square foot property in a locale such as Bronx or Manhattan generates twice the revenue and costs twice as much as any other 500 square foot property within that same locale.)

* We are also assuming that the size of property does not have an affect on the cost of the property.

## Airbnb Data Cleaning ## 

## Importing all the packages will be used 
import pandas as pd
import numpy as nm
import seaborn as sns
import matplotlib.pyplot as plt
import warnings
warnings.filterwarnings('ignore')

## reading the initial Airbnb dataset
d1= pd.read_csv("listings.csv")

## viewing columns and the type of information that the raw data contains 
d1.head()

## To get the length and shape of the initial data frame 

***from this, we can tell that the data has 48895 rows and 106 columns*** 
print(d1.shape)

## The airbnb data contains information for properties with many diffrent number of bedrooms and it also containts informations of many different cities. We first have to filter the data to contain only the 2 bedroom properties in New York City 

df_filtered1 = d1[d1['bedrooms'] == 2]

## Checking to see how many diffrent states are in the data after filtering by 2 bedrooms

***from this, we can tell that the only states present in the data now is NY.  

df_filtered1.state.unique() 
df_filtered1.to_csv('filteredlistings.csv')
df= pd.read_csv("filteredlistings.csv")

## The new shape of the airbnb data after filtering by 2 bedrooms
print(df.shape)

## Removing unwanted variables that will not be used in analysis 

 ***seeing values of object types from the data set and choosing which object type variables to eliminate. After choosing which variables to elimate, they are assigned to the drop_object variable***
 
df.select_dtypes(include=['object']).columns
drop_object = ['listing_url',
               'last_scraped','space','description',
               'require_guest_profile_picture',
               'require_guest_phone_verification',
               'name','summary','experiences_offered',
               'neighborhood_overview','notes',
               'access','interaction',
               'house_rules',
               'picture_url','host_url','host_name',
               'host_since','host_location','host_about',
               'host_response_time','host_response_rate',
               'host_is_superhost','host_thumbnail_url',
               'host_picture_url','host_neighbourhood',
               'host_verifications','host_has_profile_pic','security_deposit','extra_people',
               'host_identity_verified','calendar_updated',
               'calendar_last_scraped','first_review','last_review','requires_license']

***seeing values of float types from the data set and choosing which float type variables to eliminate. After choosing which variables to elimate, they are assigned to the drop_float variable***    

df.select_dtypes(include=['float64']).columns
drop_float = ['thumbnail_url',
              'medium_url',
              'xl_picture_url',
              'host_acceptance_rate',
              'host_listings_count','host_total_listings_count',
              'review_scores_accuracy','review_scores_cleanliness',
              'review_scores_checkin','review_scores_communication','review_scores_value','reviews_per_month']

***seeing values of int types from the data set and choosing which int type variables to eliminate. After choosing which variables to elimate, they are assigned to the drop_int variable***

df.select_dtypes(include=['int64']).columns
drop_int = ['scrape_id','host_id','minimum_minimum_nights','guests_included',
            'maximum_minimum_nights','minimum_maximum_nights',
            'maximum_maximum_nights','calculated_host_listings_count',
            'calculated_host_listings_count_entire_homes',
            'calculated_host_listings_count_private_rooms',
            'calculated_host_listings_count_shared_rooms']

***Now combining all the columns inorder to drop them***  
drop_cols = drop_object + drop_float+ drop_int

***Dropping all the variables that are not needed and assigning them to a dataframe called df_clean***
df_clean = df.drop(columns=drop_cols) 

## Cleaning the data and handling the missing values in the data 
***a function is defined which will give the statistics regarding the missing values from the data set. Based on the statistics that we get, we can decide which variables are worth dropping and which ones can be imputed***

def missing_values(df_clean):
    missing_val= df_clean.isnull().sum().to_frame()
    missing_val.columns = ['number_missing']
    missing_val['percent_missing'] = nm.round(100 * (missing_val['number_missing'] / df_clean.shape[0]))
    missing_val.sort_values(by='number_missing', ascending=False, inplace=True)

    return missing_val
number_missing = missing_values(df_clean)
number_missing

***dropping variables with missing pct over 85%. While square feet is a very important variable, since more than 85% of the data is missing, imputing would not provide an accurate result***
df_clean.drop(columns=['license','jurisdiction_names','square_feet','monthly_price','weekly_price'], inplace=True)

## Zillow Data set initial cleaning ##

## Loading the Zillow dataset

dZillow = pd.read_csv("Zip_Zhvi_2bedroom.csv") 

## Viewing the top 10 rows of this data
dZillow.head(10)

## Seeing if there are any null variables in zillow data set 
dZillow.isna().sum()

***The variable called Metro is bring dropped as it is assumed that it wont be involved in price calculation. It also has a lot of null values***

dZillow.drop(columns=['Metro'],inplace=True)

dZillow[dZillow['City'] == 'New York'].isna().sum()

## Renaming columns in the Zillow Data Set

***The common values between the Airbnb and the Zillow dataset in the zipcode value. We will merge these data through the common zipcode value. In the Zillow dataset, the zipcode values are under a variable RegionName. We have to change RegionName variable to be called zipcode***

dZillow = dZillow.rename(columns = {"RegionName":"zipcode"})

##Removing the years in the data the we are not going to be using

***We are only using the year 2016 and 2017 values to calculate our average property price. So we will remove other years data***

dZillow.columns.get_loc("2016-01")
dz= dZillow.drop(list(dZillow)[6:243], axis=1)
dz.head()

## Merging Airbnb data set and Zillow data set using zipcode

listingdf = pd.merge(df_clean,dz,on=['zipcode'],how='inner')

listingdf.head()

## Cleaning the Listingdf Data which is a merge of the Airbnb and Zillow datesets

***Two main values that will be used in calculation are price and cleaning_fee. However, both of these have the $ symbol and they have to be switched to numeric

listingdf['price']=listingdf.price.str.replace('$','').str.replace(',','').astype(float)
listingdf['cleaning_fee']=listingdf.cleaning_fee.str.replace('$','').str.replace(',','').astype(float)

## Looking at null values for cleaning_fee and imputing those null values using median 

listingdf.cleaning_fee.isna().sum()
listingdf.cleaning_fee.fillna(listingdf.cleaning_fee.median(), inplace = True)

## Outlier analysis on the merged data. The variable price and cleaning_fee will be used for further calculation, so we need to get rid of any outlier values in those variables

listingdf[['price','cleaning_fee']].plot(kind='box') 
listingdf.price.describe()
listingdf.price.value_counts()

***Dropping any price over $400 value and any price with 0 value. THe number 400 was chosen because 75% of the data of the price are around $320***

listingdf.drop(listingdf[(listingdf.price>400) | (listingdf.price==0)].index,axis=0,inplace=True)

## Calculation to calculate new variables which will give us revenue in 5 years and 10 years
dz.columns.get_loc("2016-01")

***Getting median property price for the year 2016 and 2017***

propertypricecolumns = dz.iloc[:,6:].select_dtypes(include=['int64']).columns
listingdf['avg_property_price'] = listingdf[propertypricecolumns].median(axis=1)

## yearly price calculated based on the assumption that occupancy rate is 0.75
## yearly cleaning_fee has been calculated on the assumption that the booking rate is .40

***A new variable called annual_income_price is created. This is the annual income from the property based on the median price of year 2016 and 2017 and the cleaning fee

listingdf['annual_income_price'] = listingdf.price.apply(lambda x : x * (0.75 * 365)) + listingdf.cleaning_fee.apply(lambda x : x * (0.40 * 365))
listingdf[{'zipcode','avg_property_price','annual_income_price'}]

***We can use the annual_income_price for each property to figure out the number of years it will take to profit from the property***

listingdf['average_year_to_profit']= listingdf.avg_property_price/listingdf.annual_income_price
listingdf[{'zipcode','avg_property_price','annual_income_price','average_year_to_profit'}]

***We can then calculate the revene from the properties in 5 years and also in 10 years***

listingdf['revenue_5years'] = -(listingdf.avg_property_price) + 5*listingdf.annual_income_price
listingdf['revenue_10years'] = -(listingdf.avg_property_price) + 10*listingdf.annual_income_price
listingdf[{'zipcode','avg_property_price','annual_income_price','average_year_to_profit','revenue_5years', 'revenue_10years'}]

## Saving the final dataset with the newly added variables

listingdf.to_csv('airbnbzillowlisting.csv') 

## Data Vizualization and Analysis 

dfinal = pd.read_csv("airbnbzillowlisting.csv")
dfinal.head()

## Top zipcodes based on years to start profitting. The zipcodes with the lowest years to start profiting will be great to invest in

dfinal.groupby("zipcode").average_year_to_profit.mean().reset_index().sort_values('average_year_to_profit',ascending=True).head(5).plot(kind="bar",x="zipcode",y="average_year_to_profit")

## top zipcodes with the highest revenues in the next 5 years 

dfinal.groupby("zipcode").revenue_5years.sum().reset_index().sort_values('revenue_5years',ascending=False).head(5).plot(kind="bar",x="zipcode",y="revenue_5years")

## top zipcodes with the highest revenues in the next 10 years

dfinal.groupby("zipcode").revenue_10years.sum().reset_index().sort_values('revenue_10years',ascending=False).head(5).plot(kind="bar",x="zipcode",y="revenue_10years")

## Conclusions 

The zipcodes that can make the money that was invested back quickly are 11003, 10306, 10304, 10303, and 11434
 Zipcodes 11434, 10305, and 11234 are the ones that generate the most revenue

* If the company is willing to buy properties having high cost then they should invest in Zipcodes 10011, 10025 and 10036 because these zipcodes not only have high number of costly properties but also provide very high return as the rent is very high


