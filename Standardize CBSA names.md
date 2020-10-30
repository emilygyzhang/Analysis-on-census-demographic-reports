<a id=maincontent></a>
### Data Preparation Steps:
- [**Step 1: Load Libraries and Data Sets**](#step1)
- [**Step 2: Create A Cross-Walk File to Standardize CBSA Nmaes**](#step2)
- [**Step 3: Standardize CBSA Names In The ZHVI File**](#step3)
- [**Step 4: Transpose The ZHVI Data Set & Final Checkup**](#step4)
- [**Step 5: Export Data for Data Viz in Tableau**](#step5)

#### Step 1: Load Libraries and Data Sets
Load ZHVI file and a table that contains standardized CBSA names from the US Census Bureau website


```python
import pandas as pd
import numpy as np
from fuzzywuzzy import fuzz
```


```python
df = pd.read_csv("Metro_zhvi_uc_sfrcondo_tier_0.33_0.67_sm_sa_mon.csv")
cbsa = pd.read_csv("cbsa-est2019-alldata.csv",encoding='latin-1')
cbsa = cbsa[cbsa['LSAD'].str.contains('Metropolitan Statistical Area') | cbsa['LSAD'].str.contains('Micropolitan Statistical Area')][['NAME','LSAD']]
cbsa = cbsa.drop_duplicates()
```


```python
df.head()
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
      <th>RegionID</th>
      <th>SizeRank</th>
      <th>RegionName</th>
      <th>RegionType</th>
      <th>StateName</th>
      <th>1996-01-31</th>
      <th>1996-02-29</th>
      <th>1996-03-31</th>
      <th>1996-04-30</th>
      <th>1996-05-31</th>
      <th>...</th>
      <th>2019-12-31</th>
      <th>2020-01-31</th>
      <th>2020-02-29</th>
      <th>2020-03-31</th>
      <th>2020-04-30</th>
      <th>2020-05-31</th>
      <th>2020-06-30</th>
      <th>2020-07-31</th>
      <th>2020-08-31</th>
      <th>2020-09-30</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>102001</td>
      <td>0</td>
      <td>United States</td>
      <td>Country</td>
      <td>NaN</td>
      <td>107630.0</td>
      <td>107657.0</td>
      <td>107707.0</td>
      <td>107834.0</td>
      <td>107977.0</td>
      <td>...</td>
      <td>247737.0</td>
      <td>248625.0</td>
      <td>249639.0</td>
      <td>250802.0</td>
      <td>252042.0</td>
      <td>253216.0</td>
      <td>254423.0</td>
      <td>255872.0</td>
      <td>257736.0</td>
      <td>259906.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>394913</td>
      <td>1</td>
      <td>New York, NY</td>
      <td>Msa</td>
      <td>NY</td>
      <td>187842.0</td>
      <td>187403.0</td>
      <td>187125.0</td>
      <td>186592.0</td>
      <td>186274.0</td>
      <td>...</td>
      <td>479999.0</td>
      <td>480758.0</td>
      <td>481745.0</td>
      <td>482804.0</td>
      <td>484104.0</td>
      <td>485517.0</td>
      <td>487279.0</td>
      <td>489670.0</td>
      <td>492875.0</td>
      <td>497090.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>753899</td>
      <td>2</td>
      <td>Los Angeles-Long Beach-Anaheim, CA</td>
      <td>Msa</td>
      <td>CA</td>
      <td>183929.0</td>
      <td>184185.0</td>
      <td>184205.0</td>
      <td>184312.0</td>
      <td>184286.0</td>
      <td>...</td>
      <td>671796.0</td>
      <td>675183.0</td>
      <td>680320.0</td>
      <td>685503.0</td>
      <td>689705.0</td>
      <td>691229.0</td>
      <td>692332.0</td>
      <td>696613.0</td>
      <td>703740.0</td>
      <td>711361.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>394463</td>
      <td>3</td>
      <td>Chicago, IL</td>
      <td>Msa</td>
      <td>IL</td>
      <td>164647.0</td>
      <td>164345.0</td>
      <td>163946.0</td>
      <td>163493.0</td>
      <td>162886.0</td>
      <td>...</td>
      <td>245373.0</td>
      <td>245631.0</td>
      <td>246017.0</td>
      <td>246628.0</td>
      <td>247155.0</td>
      <td>247719.0</td>
      <td>248421.0</td>
      <td>249650.0</td>
      <td>251298.0</td>
      <td>253512.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>394514</td>
      <td>4</td>
      <td>Dallas-Fort Worth, TX</td>
      <td>Msa</td>
      <td>TX</td>
      <td>114406.0</td>
      <td>114471.0</td>
      <td>114634.0</td>
      <td>114962.0</td>
      <td>115314.0</td>
      <td>...</td>
      <td>260294.0</td>
      <td>260750.0</td>
      <td>261428.0</td>
      <td>262440.0</td>
      <td>263584.0</td>
      <td>264699.0</td>
      <td>265992.0</td>
      <td>267485.0</td>
      <td>269183.0</td>
      <td>270907.0</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 302 columns</p>
</div>




```python
cbsa.head()
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
      <th>NAME</th>
      <th>LSAD</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abilene, TX</td>
      <td>Metropolitan Statistical Area</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Akron, OH</td>
      <td>Metropolitan Statistical Area</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Albany, GA</td>
      <td>Metropolitan Statistical Area</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Albany-Lebanon, OR</td>
      <td>Metropolitan Statistical Area</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Albany-Schenectady-Troy, NY</td>
      <td>Metropolitan Statistical Area</td>
    </tr>
  </tbody>
</table>
</div>



<a id=step2></a>
#### Step 2: Create A Cross-Walk File to Standardize CBSA Nmaes
Some CBSA names in the ZHVI file cannot be recognized by Tableau and they need to be converted to the standardized CBSA names reported by the US Census Bureau so that Tableau can correctly map them
<br><br>
[Go back](#maincontent)


```python
# Get unique CBSA names from ZHVI data set
cbsa_zillow = df[['RegionName','StateName']].drop_duplicates().dropna(how = 'any')
# Create a dummy column for table merging purpose
cbsa_zillow['Dummy'] = 1
cbsa_zillow.head()
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
      <th>RegionName</th>
      <th>StateName</th>
      <th>Dummy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>New York, NY</td>
      <td>NY</td>
      <td>1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Los Angeles-Long Beach-Anaheim, CA</td>
      <td>CA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Chicago, IL</td>
      <td>IL</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Dallas-Fort Worth, TX</td>
      <td>TX</td>
      <td>1</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Philadelphia, PA</td>
      <td>PA</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Generate state names for CBSA names from the US Census Bureau file
cbsa['State'] = [x.split(', ')[-1] for x in cbsa['NAME']]
# Create a dummy column for table merging purpose
cbsa['Dummy'] = 1
cbsa.head()
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
      <th>NAME</th>
      <th>LSAD</th>
      <th>State</th>
      <th>Dummy</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Abilene, TX</td>
      <td>Metropolitan Statistical Area</td>
      <td>TX</td>
      <td>1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Akron, OH</td>
      <td>Metropolitan Statistical Area</td>
      <td>OH</td>
      <td>1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Albany, GA</td>
      <td>Metropolitan Statistical Area</td>
      <td>GA</td>
      <td>1</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Albany-Lebanon, OR</td>
      <td>Metropolitan Statistical Area</td>
      <td>OR</td>
      <td>1</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Albany-Schenectady-Troy, NY</td>
      <td>Metropolitan Statistical Area</td>
      <td>NY</td>
      <td>1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Create a cross walk table to convert CBSA names in ZHVI file to standardized CBSA names

    # Merge two tables
match = cbsa_zillow.merge(cbsa, how = 'left', on = 'Dummy')
    # Keep observations where partial match score of states is 100
match['StateMatchScore'] = [fuzz.partial_ratio(x,y) for (x,y) in zip(match['StateName'],match['State'])]
match = match[match['StateMatchScore'] == 100].reset_index()
    # Keep observations with the highest partial match score of 'CBSA minue state' for each CBSA from ZHVI
match['CBSAMatchScore'] = [fuzz.partial_ratio(x.split(", ")[0],y.split(", ")[0]) for (x,y) in zip(match['RegionName'],match['NAME'])]
match = match.loc[match.groupby(['RegionName'])['CBSAMatchScore'].idxmax()][['RegionName','NAME','CBSAMatchScore']]
match.head()
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
      <th>RegionName</th>
      <th>NAME</th>
      <th>CBSAMatchScore</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>19933</th>
      <td>Aberdeen, SD</td>
      <td>Aberdeen, SD</td>
      <td>100</td>
    </tr>
    <tr>
      <th>13422</th>
      <td>Aberdeen, WA</td>
      <td>Aberdeen, WA</td>
      <td>100</td>
    </tr>
    <tr>
      <th>7049</th>
      <td>Abilene, TX</td>
      <td>Abilene, TX</td>
      <td>100</td>
    </tr>
    <tr>
      <th>21152</th>
      <td>Ada, OK</td>
      <td>Ada, OK</td>
      <td>100</td>
    </tr>
    <tr>
      <th>10866</th>
      <td>Adrian, MI</td>
      <td>Adrian, MI</td>
      <td>100</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Check obeservations where the standardized CBSA names are different from the CBSA names from ZHVI file and CBSAMatchScore isn't 100
match[(match['RegionName'] != match['NAME']) & (match['CBSAMatchScore'] != 100)].sort_values(['CBSAMatchScore'],ascending = False)
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
      <th>RegionName</th>
      <th>NAME</th>
      <th>CBSAMatchScore</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1328</th>
      <td>Louisville-Jefferson County, KY</td>
      <td>Louisville/Jefferson County, KY-IN</td>
      <td>96</td>
    </tr>
    <tr>
      <th>487</th>
      <td>Minneapolis-St Paul, MN</td>
      <td>Minneapolis-St. Paul-Bloomington, MN-WI</td>
      <td>95</td>
    </tr>
    <tr>
      <th>17875</th>
      <td>Cañon City, CO</td>
      <td>Caon City, CO</td>
      <td>90</td>
    </tr>
    <tr>
      <th>20016</th>
      <td>Espa±ola, NM</td>
      <td>Espaola, NM</td>
      <td>88</td>
    </tr>
    <tr>
      <th>21514</th>
      <td>Logan, WV</td>
      <td>Morgantown, WV</td>
      <td>80</td>
    </tr>
    <tr>
      <th>14201</th>
      <td>Marshall, TX</td>
      <td>Pearsall, TX</td>
      <td>80</td>
    </tr>
    <tr>
      <th>23809</th>
      <td>Merrill, WI</td>
      <td>Chicago-Naperville-Elgin, IL-IN-WI</td>
      <td>71</td>
    </tr>
    <tr>
      <th>21291</th>
      <td>Canton, IL</td>
      <td>Bloomington, IL</td>
      <td>67</td>
    </tr>
    <tr>
      <th>19584</th>
      <td>Port Clinton, OH</td>
      <td>Celina, OH</td>
      <td>67</td>
    </tr>
    <tr>
      <th>15333</th>
      <td>Oxford, NC</td>
      <td>Sanford, NC</td>
      <td>67</td>
    </tr>
    <tr>
      <th>13476</th>
      <td>Greenfield Town, MA</td>
      <td>Springfield, MA</td>
      <td>64</td>
    </tr>
    <tr>
      <th>14555</th>
      <td>Ionia, MI</td>
      <td>Iron Mountain, MI-WI</td>
      <td>60</td>
    </tr>
    <tr>
      <th>22387</th>
      <td>Junction City, KS</td>
      <td>Dodge City, KS</td>
      <td>60</td>
    </tr>
    <tr>
      <th>24456</th>
      <td>Boone, IA</td>
      <td>Davenport-Moline-Rock Island, IA-IL</td>
      <td>60</td>
    </tr>
    <tr>
      <th>24117</th>
      <td>Bastrop, LA</td>
      <td>Baton Rouge, LA</td>
      <td>57</td>
    </tr>
    <tr>
      <th>22441</th>
      <td>Valley, AL</td>
      <td>Huntsville, AL</td>
      <td>55</td>
    </tr>
    <tr>
      <th>21494</th>
      <td>Newton, IA</td>
      <td>Burlington, IA-IL</td>
      <td>50</td>
    </tr>
    <tr>
      <th>13531</th>
      <td>Owosso, MI</td>
      <td>Grand Rapids-Kentwood, MI</td>
      <td>50</td>
    </tr>
    <tr>
      <th>9852</th>
      <td>Dunn, NC</td>
      <td>Durham-Chapel Hill, NC</td>
      <td>50</td>
    </tr>
    <tr>
      <th>21728</th>
      <td>Summit Park, UT</td>
      <td>Salt Lake City, UT</td>
      <td>45</td>
    </tr>
    <tr>
      <th>5793</th>
      <td>Claremont, NH</td>
      <td>Boston-Cambridge-Newton, MA-NH</td>
      <td>44</td>
    </tr>
  </tbody>
</table>
</div>




```python
# Based on the result above, I decided to only consider matches with CBSAMatchScore>90
threshold = 90
match['CBSA'] = [x if y>threshold else z for (x,y,z) in zip(match['NAME'],match['CBSAMatchScore'],match['RegionName'])]
```


```python
# For cases where multiple CBSA names from the ZHVI file match to the same CBSA names from the US Census Bureau, 
# keep the original CBSA names from the ZHVI file
dup_count = pd.DataFrame(match.groupby(['CBSA'])['RegionName'].count())
dup_count.columns = ['RegionCount']
match = match.merge(dup_count,how = 'left',left_on = 'CBSA',right_on='CBSA')
match['CBSA'] = [x if y>1 else z for (x,y,z) in zip(match['RegionName'],match['RegionCount'],match['CBSA'])]
```


```python
final_match = match.drop(['NAME','CBSAMatchScore','RegionCount'],axis = 1)
final_match
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
      <th>RegionName</th>
      <th>CBSA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Aberdeen, SD</td>
      <td>Aberdeen, SD</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Aberdeen, WA</td>
      <td>Aberdeen, WA</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Abilene, TX</td>
      <td>Abilene, TX</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Ada, OK</td>
      <td>Ada, OK</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Adrian, MI</td>
      <td>Adrian, MI</td>
    </tr>
    <tr>
      <th>...</th>
      <td>...</td>
      <td>...</td>
    </tr>
    <tr>
      <th>908</th>
      <td>Youngstown, OH</td>
      <td>Youngstown, OH</td>
    </tr>
    <tr>
      <th>909</th>
      <td>Yuba City, CA</td>
      <td>Yuba City, CA</td>
    </tr>
    <tr>
      <th>910</th>
      <td>Yuma, AZ</td>
      <td>Yuma, AZ</td>
    </tr>
    <tr>
      <th>911</th>
      <td>Zanesville, OH</td>
      <td>Zanesville, OH</td>
    </tr>
    <tr>
      <th>912</th>
      <td>Zapata, TX</td>
      <td>Zapata, TX</td>
    </tr>
  </tbody>
</table>
<p>913 rows × 2 columns</p>
</div>




```python
# Make sure that no CBSA names from the US Census Bureau is assigned to multiple CBSAs from the ZHVI file
(final_match.groupby('CBSA').count()>2).any()
```




    RegionName    False
    dtype: bool



<a id=step3></a>
#### Step 3: Standardize CBSA Names In The ZHVI File
Use The Cross-Walk File to Standardize CBSA Names In The ZHVI File
<br><br>
[Go back](#maincontent)


```python
# Standardize CBSA names from Zillow
df_s = df.merge(final_match,how = 'left',on = 'RegionName')
```


```python
df_s.head()
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
      <th>RegionID</th>
      <th>SizeRank</th>
      <th>RegionName</th>
      <th>RegionType</th>
      <th>StateName</th>
      <th>1996-01-31</th>
      <th>1996-02-29</th>
      <th>1996-03-31</th>
      <th>1996-04-30</th>
      <th>1996-05-31</th>
      <th>...</th>
      <th>2020-01-31</th>
      <th>2020-02-29</th>
      <th>2020-03-31</th>
      <th>2020-04-30</th>
      <th>2020-05-31</th>
      <th>2020-06-30</th>
      <th>2020-07-31</th>
      <th>2020-08-31</th>
      <th>2020-09-30</th>
      <th>CBSA</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>102001</td>
      <td>0</td>
      <td>United States</td>
      <td>Country</td>
      <td>NaN</td>
      <td>107630.0</td>
      <td>107657.0</td>
      <td>107707.0</td>
      <td>107834.0</td>
      <td>107977.0</td>
      <td>...</td>
      <td>248625.0</td>
      <td>249639.0</td>
      <td>250802.0</td>
      <td>252042.0</td>
      <td>253216.0</td>
      <td>254423.0</td>
      <td>255872.0</td>
      <td>257736.0</td>
      <td>259906.0</td>
      <td>NaN</td>
    </tr>
    <tr>
      <th>1</th>
      <td>394913</td>
      <td>1</td>
      <td>New York, NY</td>
      <td>Msa</td>
      <td>NY</td>
      <td>187842.0</td>
      <td>187403.0</td>
      <td>187125.0</td>
      <td>186592.0</td>
      <td>186274.0</td>
      <td>...</td>
      <td>480758.0</td>
      <td>481745.0</td>
      <td>482804.0</td>
      <td>484104.0</td>
      <td>485517.0</td>
      <td>487279.0</td>
      <td>489670.0</td>
      <td>492875.0</td>
      <td>497090.0</td>
      <td>New York, NY</td>
    </tr>
    <tr>
      <th>2</th>
      <td>753899</td>
      <td>2</td>
      <td>Los Angeles-Long Beach-Anaheim, CA</td>
      <td>Msa</td>
      <td>CA</td>
      <td>183929.0</td>
      <td>184185.0</td>
      <td>184205.0</td>
      <td>184312.0</td>
      <td>184286.0</td>
      <td>...</td>
      <td>675183.0</td>
      <td>680320.0</td>
      <td>685503.0</td>
      <td>689705.0</td>
      <td>691229.0</td>
      <td>692332.0</td>
      <td>696613.0</td>
      <td>703740.0</td>
      <td>711361.0</td>
      <td>Los Angeles-Long Beach-Anaheim, CA</td>
    </tr>
    <tr>
      <th>3</th>
      <td>394463</td>
      <td>3</td>
      <td>Chicago, IL</td>
      <td>Msa</td>
      <td>IL</td>
      <td>164647.0</td>
      <td>164345.0</td>
      <td>163946.0</td>
      <td>163493.0</td>
      <td>162886.0</td>
      <td>...</td>
      <td>245631.0</td>
      <td>246017.0</td>
      <td>246628.0</td>
      <td>247155.0</td>
      <td>247719.0</td>
      <td>248421.0</td>
      <td>249650.0</td>
      <td>251298.0</td>
      <td>253512.0</td>
      <td>Chicago-Naperville-Elgin, IL-IN-WI</td>
    </tr>
    <tr>
      <th>4</th>
      <td>394514</td>
      <td>4</td>
      <td>Dallas-Fort Worth, TX</td>
      <td>Msa</td>
      <td>TX</td>
      <td>114406.0</td>
      <td>114471.0</td>
      <td>114634.0</td>
      <td>114962.0</td>
      <td>115314.0</td>
      <td>...</td>
      <td>260750.0</td>
      <td>261428.0</td>
      <td>262440.0</td>
      <td>263584.0</td>
      <td>264699.0</td>
      <td>265992.0</td>
      <td>267485.0</td>
      <td>269183.0</td>
      <td>270907.0</td>
      <td>Dallas-Fort Worth-Arlington, TX</td>
    </tr>
  </tbody>
</table>
<p>5 rows × 303 columns</p>
</div>



<a id=step4></a>
#### Step 4: Transpose The ZHVI Data Set & Final Checkup
Transpose The ZHVI Data Set To Have Month-Year As One Of The Variables & Perform A Final Checkup
<br><br>
[Go back](#maincontent)


```python
# Transpose data from wide to long 
df_t = pd.melt(df_s,
               value_name='ZHVI',
               id_vars=['RegionID','SizeRank','CBSA','RegionName','RegionType','StateName'],
               var_name='YearMonth')
```


```python
# Spot-check some records
df_t.loc[df_t['RegionName'].str.contains('Wheeling')].head()
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
      <th>RegionID</th>
      <th>SizeRank</th>
      <th>CBSA</th>
      <th>RegionName</th>
      <th>RegionType</th>
      <th>StateName</th>
      <th>YearMonth</th>
      <th>ZHVI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>278</th>
      <td>395221</td>
      <td>278</td>
      <td>Wheeling, WV-OH</td>
      <td>Wheeling, OH</td>
      <td>Msa</td>
      <td>OH</td>
      <td>1996-01-31</td>
      <td>40605.0</td>
    </tr>
    <tr>
      <th>1192</th>
      <td>395221</td>
      <td>278</td>
      <td>Wheeling, WV-OH</td>
      <td>Wheeling, OH</td>
      <td>Msa</td>
      <td>OH</td>
      <td>1996-02-29</td>
      <td>40632.0</td>
    </tr>
    <tr>
      <th>2106</th>
      <td>395221</td>
      <td>278</td>
      <td>Wheeling, WV-OH</td>
      <td>Wheeling, OH</td>
      <td>Msa</td>
      <td>OH</td>
      <td>1996-03-31</td>
      <td>40651.0</td>
    </tr>
    <tr>
      <th>3020</th>
      <td>395221</td>
      <td>278</td>
      <td>Wheeling, WV-OH</td>
      <td>Wheeling, OH</td>
      <td>Msa</td>
      <td>OH</td>
      <td>1996-04-30</td>
      <td>40724.0</td>
    </tr>
    <tr>
      <th>3934</th>
      <td>395221</td>
      <td>278</td>
      <td>Wheeling, WV-OH</td>
      <td>Wheeling, OH</td>
      <td>Msa</td>
      <td>OH</td>
      <td>1996-05-31</td>
      <td>40824.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
df_t.loc[df_t['RegionName'].str.contains('Bastrop')].head()
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
      <th>RegionID</th>
      <th>SizeRank</th>
      <th>CBSA</th>
      <th>RegionName</th>
      <th>RegionType</th>
      <th>StateName</th>
      <th>YearMonth</th>
      <th>ZHVI</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>823</th>
      <td>394364</td>
      <td>838</td>
      <td>Bastrop, LA</td>
      <td>Bastrop, LA</td>
      <td>Msa</td>
      <td>LA</td>
      <td>1996-01-31</td>
      <td>25888.0</td>
    </tr>
    <tr>
      <th>1737</th>
      <td>394364</td>
      <td>838</td>
      <td>Bastrop, LA</td>
      <td>Bastrop, LA</td>
      <td>Msa</td>
      <td>LA</td>
      <td>1996-02-29</td>
      <td>25905.0</td>
    </tr>
    <tr>
      <th>2651</th>
      <td>394364</td>
      <td>838</td>
      <td>Bastrop, LA</td>
      <td>Bastrop, LA</td>
      <td>Msa</td>
      <td>LA</td>
      <td>1996-03-31</td>
      <td>25915.0</td>
    </tr>
    <tr>
      <th>3565</th>
      <td>394364</td>
      <td>838</td>
      <td>Bastrop, LA</td>
      <td>Bastrop, LA</td>
      <td>Msa</td>
      <td>LA</td>
      <td>1996-04-30</td>
      <td>25925.0</td>
    </tr>
    <tr>
      <th>4479</th>
      <td>394364</td>
      <td>838</td>
      <td>Bastrop, LA</td>
      <td>Bastrop, LA</td>
      <td>Msa</td>
      <td>LA</td>
      <td>1996-05-31</td>
      <td>25965.0</td>
    </tr>
  </tbody>
</table>
</div>



<a id=step5></a>
#### Step 5: Export Data for Data Viz in Tableau
Everything looks good! And the final FHVI data is ready for data visualization in Tableau
<br><br>
[Go back](#maincontent)


```python
df_t.to_csv('Metro_zhvi_uc_sfrcondo_tier_0.33_0.67_sm_sa_mon_clean.csv')
```
