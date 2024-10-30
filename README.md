# Introduction

Ever wonder how race charts are created?

![covid-19 race chart](https://raw.githubusercontent.com/dexplo/bar_chart_race/gh-pages/images/covid19_horiz.gif)

We will be following the idea of ETL (Extract, Transform, and Load) to create our race chart.

# Step 1

In the first step, we download our data.  For the purpose of this tutorial, we will use the 
'BC open data Catalogue' (​​https://catalogue.data.gov.bc.ca/) to download the dataset for BC 
tuition fees:  
https://catalogue.data.gov.bc.ca/dataset/tuition-fees-for-arts-programs-at-b-c-public-post-secondary-institutions 

The direct download link to the csv file is 
https://catalogue.data.gov.bc.ca/dataset/670516e0-6cb0-4d3b-ab2b-b82ed9ee59db/resource/8a4c39f0-0986-43eb-bc61-63e3fa4d9472/download/tui1_tuition_fees_at_public_post_secondary_institutions_by_economic_development_region.csv, 
so we can use wget to download it into our working directory. 

```console
$ wget https://catalogue.data.gov.bc.ca/dataset/670516e0-6cb0-4d3b-ab2b-b82ed9ee59db/resource/8a4c39f0-0986-43eb-bc61-63e3fa4d9472/download/tui1_tuition_fees_at_public_post_secondary_institutions_by_economic_development_region.csv
--2024-07-18 19:22:38--  https://catalogue.data.gov.bc.ca/dataset/670516e0-6cb0-4d3b-ab2b-b82ed9ee59db/resource/8a4c39f0-0986-43eb-bc61-63e3fa4d9472/download/tui1_tuition_fees_at_public_post_secondary_institutions_by_economic_development_region.csv
Resolving catalogue.data.gov.bc.ca (catalogue.data.gov.bc.ca)... 142.34.194.118
Connecting to catalogue.data.gov.bc.ca (catalogue.data.gov.bc.ca)|142.34.194.118|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 20845 (20K) [text/csv]
Saving to: ‘tui1_tuition_fees_at_public_post_secondary_institutions_by_economic_development_region.csv’

tui1_tuition_fees_at_public 100%[=========================================>]  20.36K  --.-KB/s    in 0.02s   

2024-07-18 19:22:38 (1.14 MB/s) - ‘tui1_tuition_fees_at_public_post_secondary_institutions_by_economic_development_region.csv’ saved [20845/20845]
```

Take a look at our csv file:

```console
$ head tui1_tuition_fees_at_public_post_secondary_institutions_by_economic_development_region.csv 
Annual Tuition Fees for Arts Program - Full-Time Domestic Students by Economic Development Region,,,,
"BC Public Post-Secondary Institutions, Academic Year (AY) 2014/15 to 2023/24",,,,
,,,,
Year,Sector,Economic Development Region,Institution,Value
AY 2014/15,College,Vancouver Island/Coast,Camosun College,"$3,155"
AY 2014/15,College,North,Coast Mountain College,"$2,581"
AY 2014/15,College,Cariboo,College of New Caledonia,"$2,516"
AY 2014/15,College,Kootenay,College of the Rockies,"$2,540"
AY 2014/15,College,Mainland/Southwest,Douglas College,"$2,874"
AY 2014/15,College,Mainland/Southwest,Langara College,"$2,702"
```

You may want to rename your data file to tuition.csv to reduce the amount of typing you need to do later on.

# Step 2

If you look at the python package we will be using to create our chart 
(https://github.com/dexplo/bar_chart_race#quickstart), 
you will see that it expects our data in the following format:

![race chart expected format](https://raw.githubusercontent.com/dexplo/bar_chart_race/gh-pages/images/wide_data.png)

So we need to transform our tuition data into this format, with the first row listing the institution names and 
the first column being a date.  The following script can perform such a transformation (assuming we rename the 
data file to tuition.csv):

```bash
#!/bin/bash

# header
echo -n "date,"
grep -P '(?!AY )2014\/15,.*"' tuition.csv | sort | cut -f 4 -d "," | paste -d ',' -s

# data
for i in {14..23}
do
    data=$(grep -P "20${i}\/$(( i + 1 )),.*\"" tuition.csv | sort | grep -oP '".*"'  |tr -d '$",'| paste -d ',' -s)
    echo "20${i},$data"
done
```

I saved the above script into a file called transform.sh, when executed properly, it will write the transformed 
data to stdout:

```
date,College of New Caledonia,College of the Rockies,Selkirk College,Douglas College,Langara College,Vancouver Community College,Coast Mountain College,Northern Lights College,Okanagan College,Camosun College,North Island College,British Columbia Institute of Technology,Justice Institute of British Columbia,Nicola Valley Institute of Technology,University of Northern British Columbia,Simon Fraser University,University of British Columbia,University of Victoria,Capilano University,Emily Carr University of Art and Design,Kwantlen Polytechnic University,University of the Fraser Valley,Thompson Rivers University,Royal Roads University,Vancouver Island University
2014,2516,2540,2644,2874,2702,2477,2581,3002,3267,3155,2738,5247,4714,2376,4913,5217,4890,5159,3610,3788,3932,4020,3907,6660,4095
2015,2565,2591,2697,2931,2756,2526,2633,3060,3332,3218,2793,5350,4808,2423,5011,5322,4988,5262,3683,3864,4010,4100,3985,6790,4177
2016,2616,2643,2751,2990,2811,2577,2686,3119,3277,3283,2849,5455,4904,2472,5111,5428,5088,5368,3756,3942,4089,4182,4064,6925,4261
2017,2669,2695,2802,3050,2867,2628,2739,3181,3343,3348,2905,5563,5002,2521,5213,5537,5190,5475,3831,4021,4170,4266,4145,7060,4346
2018,2722,2749,2855,3110,2924,2681,2794,3244,3410,3415,2963,5674,5102,2572,5318,5648,5294,5585,3908,4101,4253,4351,4228,7200,4433
2019,2776,2804,2907,3171,2983,2735,2850,3309,3478,3484,3022,5787,5204,2623,5424,5761,5399,5696,3986,4183,4339,4438,4313,7340,4521
2020,2831,2860,2960,3234,3042,2789,2907,3375,3547,3553,3082,5901,5308,2675,5533,5876,5507,5810,4066,4267,4425,4527,4399,7487,4612
2021,2887,2917,3019,3299,3103,2845,2965,3443,3618,3624,3144,5570,5414,2729,5644,5994,5617,5926,4147,4267,4514,4618,4487,7637,4704
2022,2944,2975,3077,3365,3165,2902,3024,3511,3690,3697,3206,5681,5523,2783,5756,6114,5729,6045,4230,4352,4604,4710,4576,7789,4798
2023,3003,3035,3136,3431,3229,2960,3085,3581,3764,3771,3269,5795,5633,2839,5872,6236,5843,6166,4315,4439,4696,4804,4668,7945,4894
```

# Step 3

The final step, we need to ingest the transformed data into our python module.  Below is the python script 
called `make_chart.py` that can make it happen (based on the bar-chart-race quick start documentation 
https://github.com/dexplo/bar_chart_race#quickstart):

```python
#!/usr/bin/env python

import bar_chart_race as bcr
import pandas
import sys

df = pandas.read_csv(sys.stdin)
# Need to make date an index so it turns into a label
df = df.set_index('date')
bcr.bar_chart_race(df = df, title='test', filename='tuition.gif')
```

The file `tuition.gif` will be created in the current directory.

```shell
@env3d ➜ /workspaces/python-data-transformation-project (main) $ ./transform.sh | python make_chart.py 
/usr/local/python/3.12.1/lib/python3.12/site-packages/bar_chart_race/_make_chart.py:889: FutureWarning: Series.fillna with 'method' is deprecated and will raise in a future version. Use obj.ffill() or obj.bfill() instead.
  df_values.iloc[:, 0] = df_values.iloc[:, 0].fillna(method='ffill')
/usr/local/python/3.12.1/lib/python3.12/site-packages/bar_chart_race/_make_chart.py:286: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
  ax.set_yticklabels(self.df_values.columns)
/usr/local/python/3.12.1/lib/python3.12/site-packages/bar_chart_race/_make_chart.py:287: UserWarning: set_ticklabels() should only be used with a fixed number of ticks, i.e. after set_ticks() or using a FixedLocator.
  ax.set_xticklabels([max_val] * len(ax.get_xticks()))
MovieWriter imagemagick unavailable; using Pillow instead.
@env3d ➜ /workspaces/python-data-transformation-project (main) $ ls -lh tuition.gif
-rw-rw-rw- 1 codespace codespace 1.5M Oct 30 21:44 tuition.gif
```

## Note
My example above uses bash script for transformation and python for loading data into bar_chart_race.  
Practice by creating a python script named transform.py that performs both the transformation and the 
chart creation  within the same script using Pandas.  At a high level, you will perform the following steps:

  1. Ingest tuition CSV file into a dataframe
  1. Using various dataframe and python functions such as column and row slices, query selection, etc.
     To create a dataframe with the shape that fits the race chart module
  1. Create the race chart and output to mp4 video file
  1. Make sure you study all the bar_chart_myrace customization options so your chart has proper labels, etc.  

Here’s a pandas cheat sheet to help you: https://pandas.pydata.org/Pandas_Cheat_Sheet.pdf.  While
there may be more simple solution to this problem, it's good practice to  
create a new dataframe by systemically extracting various columns and rows from the original dataframe, using 
techniques such as unique, subset rows, subset columns, and creating dataframe.

# Project (Idea 1)

After you are familiar with the process, create your own animated chart using your own dataset that you find 
online.  You will need to download the dataset to your working directory and named your python script `project.py` 
with the chart output going to the directory ${HOME}/public_html/

If you don’t have any datasets in mind, one interesting dataset is the vancouver crime data, which you can 
download it from https://geodash.vpd.ca/opendata/#.  A relatively simple project would be to create a race chart 
of all crimes for each vancouver neighborhood between 2003 to present (on a yearly basis).  Other popular websites 
with lots of datasets are https://www.kaggle.com/ and https://huggingface.co 

# Proejct (Idea 2)

Race chart is just one of the many way to visualize a dataset.  There are many more chart types and libraries you 
can use in conjunction with python and pandas to create impactful and meaningful visualizations.

Pick a dataset and create a visualization not covered in this class.  You can start by looking at the matplotlib
library https://matplotlib.org/stable/plot_types/index.html or https://seaborn.pydata.org/examples/index.html

Another idea is to create a sankey diagram via the plotly library: https://plotly.com/python/sankey-diagram/ 

# Hand-in

Make sure you push the completed project using the following commands:

```console
git add -A
git commit -a -m 'submit'
git push
```

For our final class, you will be presenting your visualization to the class.  Focus on your learning journey, 
highlighting the following:

  1. Why you picked the dataset
  2. How did you acquire the dataset
  3. What transformation do you need to perform to the raw data
  4. Which visualization library did you use?
  5. Why did your visualization is effective?  What story are you trying to tell?
  6. What techniques did you learn during this project that we did not cover in class?






