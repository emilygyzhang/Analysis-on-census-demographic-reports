<a id = main></a>
# Scraping website with multiple pages
This notebook is created to scrap 2020 FFIEC Census Demographic Reports from FFIEC website
- [**Import Packages**](#packages)
- [**Create Functions to Retrieve Census Demographic Reports by County**](#function)
- [**An Example of Application of Functions Crated**](#application)
    - [**Retrieve All FIPs From Wikipedia**](#fips)
    - [**Retrieve Census Demographic Reports of All Counties**](#censusreport)

<a id = packages></a>
## Import Packages
[**Go Back to Main Contents**](#main)


```python
from lxml import html
import requests
import pandas as pd
from time import sleep
from random import randint
import dill
import re
```


```python
fips = pd.read_pickle('fips.pkl')
all_states = pd.read_pickle('all_states.pkl')
```

<a id = function></a>
## Create Functions to Retrieve Census Demographic Reports by County
Create functions to find all census demographic reports for all states and counties. 
<br><br>
All functions take a 3-digit county FIP code padded with leading zero(s) and a 2-digit state FIP code padded with leading zero. For example, if you want to retrieve the census demographic report for Cuyahoga County, Ohio State, use county = '035' and state = '39'
<br><br>
[**Go Back to Main Contents**](#main)

#### Function 1 
A function that allows us to pass different parameters to change the url
<br><br>
[**Go Back**](#function)


```python
def create_url(county,state,page=1):
    url="https://www.ffiec.gov/census/report.aspx?year=2020&county={}&tract=ALL&state={}&report=demographic&page={}".format(county,state,page)
    return url
```

#### Function 2
A function that searches for the maximum number of pages of this report
<br><br>
[**Go Back**](#function)


```python
def find_max_page(url):
    # Parse the main url and find the section says "Page"
    r = requests.get(url)
    parser = html.fromstring(r.content)
    
    find_page = parser.xpath("//*[contains(text(),'Page')]")

    # Set pages = 1 if there is only one page for the county, otherwise search for the maximum page number
    try:
        find_page[0].text_content()
        # Take the last string component from the string with "Page" since that is the maximum page number
        pages = int(find_page[0].text_content().split()[-1])
    except:
        pages = 1
    
    return pages
```

#### Function 3
A function that loops through all pages of the census demographic report and store data in one data frame
<br><br>
[**Go Back**](#function)


```python
def parse_and_save(county,state):
    # Check if url is valid
    url_invalid = create_url(county,state)
    r_invalid = requests.get(url_invalid)
    parser_invalid = html.fromstring(r_invalid.content)
    invalid = parser_invalid.xpath("//*[@id='Report1_lblERR']")
    try:
        if 'Invalid' in str(invalid[0].text_content()): # If url is invalid, then return 'Invalid'
            Invalid = 'Invalid'
            return Invalid
    except:
        pages = find_max_page(create_url(county,state)) # Find the maximum number of pages

        for page in range(1,int(pages)+1):
            url = create_url(county,state,page) # Change url as we move from page to page
            r = requests.get(url)
            parser = html.fromstring(r.content)
            tb = parser.xpath("//table[@id='Report1_dgReportDemographic']//tr") # Find the section where the demographic table locates
            # Paser table header if we are on the first page and initial the output data frame
            if (page == 1):
                cols = ['State','County','Page']
                for col in tb[0]:
                    cols.append(col.text_content())
                df = pd.DataFrame([cols])

            for i in range(1,len(tb)): # Loop through each row since the second row to exclude the header
                if len(tb[i]) != 12:
                    break
                row = [state,county,page] # Initialize a list to store elements in a row
                for j in range(0,len(tb[i])): # Loop through each element in a row
                    element = str(tb[i][j].text_content())
                    try:
                        element = float(element.replace(',','').replace('$',''))
                    except:
                        pass
                    row.append(element)
                df.loc[len(df)] = row

            sleep(randint(5,10)) # Control the scrapping rate - avoid stressing out the server and being banned

        df.columns = df.loc[0] # Set the first row as table header
        df = df[1:] # Remove the first row
        df.reset_index(drop = True,inplace = True)
        return df
```

<a id = application></a>
## An Example of Application of Functions Created
In this example, I am going to loop through all states and counties in the United States and retrieve their census demographic reports, and eventually store them in a data frame called 'all_states'
<br><br>
[**Go Back to Main Contents**](#main)

<a id = fips></a>
#### Step 1: Retrieve All FIPs from Wikipedia
Retrieve FIPs for states and counties in the U.S. from Wikipedia. Data is stored in a data frame called 'fips'
<br><br>
[**Go Back**](#application)


```python
# Retrieve and parse Wikipedia page, and find the table to be stored
fips_url = "https://en.wikipedia.org/wiki/List_of_United_States_FIPS_codes_by_county"
fips_r = requests.get(fips_url)
fips_parser = html.fromstring(fips_r.content)

fips_tb = fips_parser.xpath("//*[@id='mw-content-text']/div[1]/table[2]//tr")
```


```python
# Loop through all rows in the table and store them in a data frame

fips = pd.DataFrame([['FIPS','County','State']]) # Create an empty data frame with headers
state = "Alabama" # Initial state is Alabama

for i in range(1,len(fips_tb)):
    row = fips_tb[i].text_content().replace('\xa0','').split('\n') 
    row = [re.sub("[\(\[].*?[\)\]]","",str(i)) for i in row if i] # Remove empty strings in the list
    
    # If there are 3 or more elements in a row, then state is updated
    if len(row) >= 3:
        state = row[2]
    else:
        state = str(state)
        row.append(state)
    
    fips.loc[len(fips)] = row
    
fips.columns = fips.loc[0] # Set the first row as table header
fips = fips[1:] # Remove the first row
fips.reset_index(drop = True,inplace = True)
```


```python
fips
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FIPS</th>
      <th>County</th>
      <th>State</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01001</td>
      <td>Autauga County</td>
      <td>Alabama</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01003</td>
      <td>Baldwin County</td>
      <td>Alabama</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01005</td>
      <td>Barbour County</td>
      <td>Alabama</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01007</td>
      <td>Bibb County</td>
      <td>Alabama</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01009</td>
      <td>Blount County</td>
      <td>Alabama</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>3237</th>
      <td>56037</td>
      <td>Sweetwater County</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <th>3238</th>
      <td>56039</td>
      <td>Teton County</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <th>3239</th>
      <td>56041</td>
      <td>Uinta County</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <th>3240</th>
      <td>56043</td>
      <td>Washakie County</td>
      <td>Wyoming</td>
    </tr>
    <tr>
      <th>3241</th>
      <td>56045</td>
      <td>Weston County</td>
      <td>Wyoming</td>
    </tr>
  </tbody>
</table>
<p>3242 rows × 3 columns</p>
</div>



<a id = censusreport></a>
#### Step 2: Retrieve Census Demographic Reports of All Counties
Loop through all states and counties and retrieve the corresponding census demographic reports, and store them in 'all_states' data frame
<br><br>
[**Go Back**](#application)


```python
# Initial the data frame to store all census reports
all_states=pd.DataFrame()
```


```python
# Loop through all states and counties in fips data frame and retrieve the corresponding reports
for i in range(0,len(fips)):
    state = fips.loc[i][0][0:2]
    county = fips.loc[i][0][2:5]
    state_name = fips.loc[i][2]
    county_name = fips.loc[i][1]
    
    print(str(i) + ', ' + state + ' ' + state_name + ', ' + county + ' ' + county_name)
    try:
        if parse_and_save(county = county,state = state) == 'Invalid':
            print(parse_and_save(county = county,state = state))
            continue
    except:
        pass
    df = parse_and_save(county = county,state = state)
    df['State_Name'] = state_name
    df['County_Name'] = county_name
    all_states = pd.concat([all_states,df],ignore_index = True)
    
    sleep(randint(5,10)) # Control the scrapping rate - avoid stressing out the server and being banned
```


```python
all_states
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>State</th>
      <th>County</th>
      <th>Page</th>
      <th>Tract Code</th>
      <th>Tract Income Level</th>
      <th>Distressed or Under  -served Tract</th>
      <th>Tract Median Family Income %</th>
      <th>2020 FFIEC Est. MSA/MD non-MSA/MD Median Family Income</th>
      <th>2020 Est. Tract Median Family Income</th>
      <th>2015 Tract Median Family Income</th>
      <th>Tract Population</th>
      <th>Tract Minority %</th>
      <th>Minority Population</th>
      <th>Owner Occupied Units</th>
      <th>1- to 4- Family Units</th>
      <th>State_Name</th>
      <th>County_Name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>01</td>
      <td>001</td>
      <td>1</td>
      <td>201</td>
      <td>Upper</td>
      <td>No</td>
      <td>122.93</td>
      <td>65700</td>
      <td>80765</td>
      <td>72727</td>
      <td>1948</td>
      <td>12.58</td>
      <td>245</td>
      <td>507</td>
      <td>724</td>
      <td>Alabama</td>
      <td>Autauga County</td>
    </tr>
    <tr>
      <th>1</th>
      <td>01</td>
      <td>001</td>
      <td>1</td>
      <td>202</td>
      <td>Middle</td>
      <td>No</td>
      <td>82.4</td>
      <td>65700</td>
      <td>54137</td>
      <td>48750</td>
      <td>2156</td>
      <td>59.55</td>
      <td>1284</td>
      <td>433</td>
      <td>785</td>
      <td>Alabama</td>
      <td>Autauga County</td>
    </tr>
    <tr>
      <th>2</th>
      <td>01</td>
      <td>001</td>
      <td>1</td>
      <td>203</td>
      <td>Middle</td>
      <td>No</td>
      <td>94.26</td>
      <td>65700</td>
      <td>61929</td>
      <td>55766</td>
      <td>2968</td>
      <td>25.47</td>
      <td>756</td>
      <td>828</td>
      <td>1327</td>
      <td>Alabama</td>
      <td>Autauga County</td>
    </tr>
    <tr>
      <th>3</th>
      <td>01</td>
      <td>001</td>
      <td>1</td>
      <td>204</td>
      <td>Middle</td>
      <td>No</td>
      <td>116.82</td>
      <td>65700</td>
      <td>76751</td>
      <td>69114</td>
      <td>4423</td>
      <td>17.21</td>
      <td>761</td>
      <td>1345</td>
      <td>1806</td>
      <td>Alabama</td>
      <td>Autauga County</td>
    </tr>
    <tr>
      <th>4</th>
      <td>01</td>
      <td>001</td>
      <td>1</td>
      <td>205</td>
      <td>Upper</td>
      <td>No</td>
      <td>127.74</td>
      <td>65700</td>
      <td>83925</td>
      <td>75574</td>
      <td>10763</td>
      <td>31.54</td>
      <td>3395</td>
      <td>2255</td>
      <td>3237</td>
      <td>Alabama</td>
      <td>Autauga County</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>75878</th>
      <td>56</td>
      <td>043</td>
      <td>1</td>
      <td>3.02</td>
      <td>Middle</td>
      <td>Yes*</td>
      <td>91.93</td>
      <td>79700</td>
      <td>73268</td>
      <td>66958</td>
      <td>2566</td>
      <td>23.81</td>
      <td>611</td>
      <td>799</td>
      <td>1077</td>
      <td>Wyoming</td>
      <td>Washakie County</td>
    </tr>
    <tr>
      <th>75879</th>
      <td>56</td>
      <td>043</td>
      <td>1</td>
      <td>9999.99</td>
      <td>Middle</td>
      <td>No</td>
      <td>90.77</td>
      <td>79700</td>
      <td>72344</td>
      <td>66113</td>
      <td>8400</td>
      <td>17.61</td>
      <td>1479</td>
      <td>2590</td>
      <td>3743</td>
      <td>Wyoming</td>
      <td>Washakie County</td>
    </tr>
    <tr>
      <th>75880</th>
      <td>56</td>
      <td>045</td>
      <td>1</td>
      <td>9511</td>
      <td>Upper</td>
      <td>No</td>
      <td>120.81</td>
      <td>79700</td>
      <td>96286</td>
      <td>87994</td>
      <td>3442</td>
      <td>6.36</td>
      <td>219</td>
      <td>1103</td>
      <td>1724</td>
      <td>Wyoming</td>
      <td>Weston County</td>
    </tr>
    <tr>
      <th>75881</th>
      <td>56</td>
      <td>045</td>
      <td>1</td>
      <td>9513</td>
      <td>Middle</td>
      <td>No</td>
      <td>106.78</td>
      <td>79700</td>
      <td>85104</td>
      <td>77775</td>
      <td>3710</td>
      <td>9.14</td>
      <td>339</td>
      <td>1227</td>
      <td>1621</td>
      <td>Wyoming</td>
      <td>Weston County</td>
    </tr>
    <tr>
      <th>75882</th>
      <td>56</td>
      <td>045</td>
      <td>1</td>
      <td>9999.99</td>
      <td>Middle</td>
      <td>No</td>
      <td>109.84</td>
      <td>79700</td>
      <td>87542</td>
      <td>80000</td>
      <td>7152</td>
      <td>7.8</td>
      <td>558</td>
      <td>2330</td>
      <td>3345</td>
      <td>Wyoming</td>
      <td>Weston County</td>
    </tr>
  </tbody>
</table>
<p>75883 rows × 17 columns</p>
</div>




```python
# Save important data frames in this workspace
fips.to_pickle("./fips.pkl")
all_states.to_pickle("./all_states.pkl")
```
