# etl-project
ETL Project - Hurricane Sandy

For my project, I wanted to find data regarding Hurricane Sandy. I was born and raised at the Jersey Shore and I was living there at the time the storm hit the Jersey coast. I can still remember driving through the neighborhoods along the Barnegat Bay and Seaside Park. My objective for the extraction portion of the project was to find data related to the storms weather affects such as percipitation, wave length and height, wind speed etc. However, I will have to admit that I was not so successful in obtaining all the weather indicator's I intended to get. I think with some more time to search for accessible weather API's and dataset, I beleive I can build a significant profile of Superstorm Sandy.

Before we begin, the following dependencies installed and API keys will be required to execute the code I have written for the project.

API Keys:
New York Times Developer Portal - get your key from https://developer.nytimes.com/
NOAA Weather API Key - get your key from https://www.ncdc.noaa.gov/cdo-web/token

** please note, you will need to enable access to the New York Times Archive API and Article Search API in order to perform the API calls in the jupyter notebook.

** Please visit config.py before you begin. You will be able to enter your API keys and credentials for mongoDB Cloud. You will have to enter your NOAA API key in the variable token to perform the requests.

Dependencies:
1. Pandas
2. json
3. requests
4. pymongo
5. pprint ** (Optional - only if you want an easier time reading json)

Data Sources (Extraction):

1. SBA_Disaster_Loan_Data_Superstorm_Sandy_0.xlsx - from data.world
    
    This data was found on the data.world website. There were several results when searching for data on Hurricane Sandy. I chose the excel file named above as my first data source. My intent from this source was to understand the financial impact from the storm. What this dataset represents is the damage totals for both personal and commerical realestate. The document does not contain any personal information but does contain the claim ID numbers for people who filed claims as a result of Hurricane Sandy. I will explain the tranformation of the data below, but I was only interested in the locations (represted by town, county and zip code) and the money value of each claim.

2. National Centers For Environmental Information - API Call

    My focus of using NOAA's weather API was to collect any relevant data related to Hurricane Sandy. I discovered that the dataset that best summarized the data that I wanted for my final dataset was Global Historical Climate Network (GHCN) - Daily Summaries. I also needed to specify a specific time period in which to call the data. I chose to collect data from October 21, 2012 through November 2, 2012. Now, HS made landfall on the Jersey coast on October 29, 2012, however, the timeframe I chose represents the entire life cylce of Sandy. After some analyzing of the returned data, I chose to display percipitation totals for the purposes of the project. Future collection, would include wind speed and ocean data.

3. New York Times API

    The final part of my project looks to collect headlines that are related to Hurricane Sandy. This call required searching through the Article Search API and Archive API. My call to both of these API's only returned 20 rows of data and my next thought was to see what articles I could find from other news article API. Unfortunately, I was unable to get an access key in time to do any further calls. I think expanding this section to national websites from FLorida or the Carolinas would help show the signifigance of the storm and it's aftermath. And that was the end goal of this sections. I wanted to collect and store national headlines from the New Jersey/New York area that reported on the mainy issues that arose after Sandy left our region. It was also interesting to see how there were no articles about the storm leading up October 29, 2012, no one from the NYT reported on the storms impact until after it hit our area. I think expanding this section to a national level would allow me to further analyze what the our expectations for the storm were and if education on hurricanes.


Cleaning and Transformation:

    I used one jupyter notebook, named nyt_api_sandy.ipynb, to extract and transform all the data for my project. The notebook is seperated into 4 sections. The first 3 sections are responsible for extracting the data from their repective sources and building dataframes to display the data for analysis. The first section has to do with the SBS Disastier data obtained from data.world. The excel file that I downloaded from the website contained 5 spreadsheet within the workbook. I was only interested in sheet #4 (Damaged Property - Personal) and sheet #5 (Damaged Property - Business). In order to get these sheets into jupyter notebook, I chose to use pandas read.excel function and the parameter shee_name = [3] to get sheet # 4. Once I had it read in, I used a for loop to iterate through the dictionary's keys and values and used pd.DataFrame to create the dataframe. I was only interested in informtion from New Jersey, therefore, I used .loc to return a new dataframe holding claims and their values. Finally, chose the column names I was interested in saved it to a final dataframe for conversion into mongos for later. I performed the same steps described above for Sheet #5.

    Section 2 of the notebook is dedicated to requesting weather data from the NOAA API. After reviewing their documentation on their API calls, I decided to use their "data" API filtered with the paraments Global Historical Climate Network (GHCN), start=2012-10-21, end=2012-11-02. The final parameter for each call is to use a township's ZIP code as the locationid. I chose ZIP codes for townships that are located near the shore. However, after reviewing the data from the SBA excel spreadsheet, future ideas for this API call is to use a for loop with the ZIP codes from the excel spreadsheets and use it to get weather data for each township. What each dataframe returns is the amount of percipitation that occured for each day from October 21, 2012 and November 2, 2012. What you will see is that it did not rain in the days leading up to Ocotber 29, 2021 and then you values increase showing the amount of rain that occured during the storm. 

    The third section of the Jupyter Notebook is used to call news headlines from the New York Times API. I made 2 API requests to their website. The first call was to the Article Search API which took a specific keyword to search for any article with Hurricane Sandy. Using pprint to display the json data, I was able to set a variable to a list of articles. Using a for loop, I chose individual fields from the json and stored them into a dictionary. At the end of the loop before moving to the next article, I appended the dictionary back to a list which I would use to display the final dataframe for this API request. The second API request was to NYT's Achive API. The only parameters that it took was {Year}/{Month} which made finding articles related to Hurricane Sandy harder than my first request. In order to get the data from this API, I read in all the data from the articles, examined the the general layout and chose to use 2 .loc to first filter the dataframe by news_desk == New York and then by pub_date >= 2012-10-28T18:05:43+0000, which was the first article date I found that referenced Hurricane Sandy. the last step in this section was to comnine both dataframes using pd.concat. After reseting the indexes and making sure all column headings were correct, the datframe was ready to be stored into mongos db.

Load:

    The final portion of my jupyter notebook was to send the information to mongoDB. For this section, please ensure that your UserID and Password for MongoDB Cloud are loaded into config.py. My main database is called hurricane_sandy_db. I chose to create 3 collections to hold each datasets I extracted and transformed. I orginally dumped the data into one database but found that 2 of the datasets were unable to be seen behind the large SBA damage dataset. So, puting each into a collection which I could later add too was more ideal for the focus of my project. Once the collections were created, I converted the dataframes to dictionaries and used mongo's .insert_many to add the data to the database. I chose to also store the data to mongo's cloud database, if you entered your credentials into config.py, you should be able to run the code and log into the database on mongoDB compass to see the data. 


