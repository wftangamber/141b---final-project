# 141b final project
# Wufan Tang
In this project, I am going to explain and analyze the reported cases and deaths of the COVID-19 coronavirus worldwide. The website I am choosing to crawl is [woroldometer](https://www.worldometers.info/coronavirus/?utm_campaign=homeAdvegas1?#countries).

COVID-19 is a highly contagious disease which affect people's life worldwild. Comparing to other contagious disease, COVID-19 has a high fatality rate. Although most people infected with the COVID-19 only have mild to moderate illness, a lot of people will have severe illness which might lead to death. I am interested in the fatality situation across countries. And hopefully we can find the reasons behind the differences of death rate to avoid more deaths. So in this project, I am going to define the fatality rate as total death/total cases, and have some basic analyze toward the death situation.

(The detailed definition of the code is written within in the code after #)

I. crawl the website and get the data

> I am using the BeautifulSoup in the project because I need to scrape and parse HTML
```
import pandas as pd
import numpy as np
import requests
from bs4 import BeautifulSoup
```
```
def getHtml(url):
    # create a persistent session which maintains certain parameters across requests
    session = requests.Session()
    req=session.get(url)

    # return the response data
    html=req.text
    return html
```
```
# define the url I am going to crawl
url='https://www.worldometers.info/coronavirus/?utm_campaign=homeAdvegas1?#countries'
html=getHtml(url)
```
```
def getData(html):
    '''
    (str)->list
    
    html: URL request return value, str
    
    return data, list
    '''
    # use BeautifulSoup to scrape the website
    soup = BeautifulSoup(html,'lxml')
    # find the data with 'tr' in soup, cause the target data is in 'tr'
    divs = soup.find_all('tr', style='')
    
    # define the empty string, which for the panda dataframe later
    data=[]
    for div in divs:
        if '/a' in str(div):
            data_series=[]
            data_series.append(div.a.string)
            for idx,td in enumerate(div.find_all('td')):
                if idx >=2 and idx <=15:
                    if idx==7:
                        pass
                    elif idx ==14:
                        data_series.append(td.a.string) if td.a != None else data_series.append(np.nan)
                    else:
                        data_series.append(td.string)

            data.append(data_series)
    return data
```
```
# set column names of panda dataframe
columns=['Country,Other','Total Cases','New Cases','Total Deaths','New Deaths','Total Recovered',
         'Active Cases','Serious Critical','Tot Cases/1M pop','Deaths/1M pop','Total Tests',
         'Tests/1M pop','Population','Region']

data=getData(html)

pdata=pd.DataFrame(data,columns=columns)
pdata.head()
```
II. Manipulate the data
> The raw data is rough, so I am doing some manipulation for a better visualizatioin and analyz later.
> A new column of Total Death Ratio is created here, which define as the total deaths/total cases, and allow us to see the death rates more directly.
```
# remove the , and + signs in numbers
def cleanComma(pdata,columns):
    for column in columns:
        pdata[column]=pdata[column].str.replace(',','')
        pdata[column]=pdata[column].str.replace('+','')
    
    
cleanComma(pdata,columns)
pdata.head()
```
```
def change_to_numeric(pdata,columns):
    for column in columns:
        pdata[column]=pd.to_numeric(pdata[column], downcast='float', errors='coerce').fillna(np.nan)
    
    
columns_change2numeric=['Total Cases','New Cases','Total Deaths','New Deaths','Total Recovered',
         'Active Cases','Serious Critical','Tot Cases/1M pop','Deaths/1M pop','Total Tests',
         'Tests/1M pop','Population']

change_to_numeric(pdata,columns_change2numeric)

# create a new column - Total Deaths Ratio
pdata['Total Deaths Ratio']=pdata['Total Deaths']/pdata['Total Cases']
pdata.head()
```
```
# replace the missing values with 0
pdata.fillna(0,inplace=True)
pdata.info()
pdata.drop_duplicates(['Country,Other'],keep='first',inplace=True)
pdata.info()
# check
pdata[pdata['Country,Other']=='India']
```
```
#group with regions
pdata_group=pdata.groupby("Region")[['Total Cases','New Cases','Total Deaths','New Deaths','Total Recovered',
         'Active Cases','Serious Critical','Tot Cases/1M pop','Deaths/1M pop','Total Tests',
         'Tests/1M pop','Population']].sum()
pdata_group.index
```
III. Visualization
```
import matplotlib
import matplotlib.pyplot as plt
import seaborn as sns
#new death vs. region plot to find the rough situation
plt.figure(figsize=(10, 8))
plt.plot(pdata_group.index,pdata_group['New Deaths'],'o-',color='red',linewidth='1')
plt.bar(pdata_group.index,pdata_group['New Deaths'],width=0.6, align='center',color='skyblue')
plt.xlabel('Region')
plt.ylabel('New Deaths')
plt.title('New Deaths vs Region')
plt.show()
```
> The new death vs. region plot
