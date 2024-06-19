# River level forecasting

In this article I explore an approach to producing free forecasts of river level heights for the river Cam (the main river in Cambridge). I will discuss the process of finding and storing the data and discuss different ways to produce the forecasts. Lastly, we implement this in production so it can be accessed by potential users. All of this was done using free software / sites. 

This is intended to be a helpful guide to implementing a similar forecasting measure on other rivers.

## Motivation
While studying in Cambridge, I was involved in college rowing as a rower and a cox. Outings were often cancelled due to poor weather conditions and this usually happened with relatively short notice. Rowing on the Cam (Specifically on the section between Jesus Lock and Baitsbite Lock) is governed by the [CUCBC](https://www.cucbc.org/) (Cambridge University Combined Boat Clubs). Outings are typically cancelled due to something called the [flag](https://www.cucbc.org/handbook/flag) which is set to different colours depending on how safe it is to go out on that day. Typically, conditions that lead to an outing being cancelled are:
1. High wind speeds (35mph or so)
1. Fog / Mist leading to poor visibility
1. Extreme cold
1. High water levels or fast flowing water
The first three of these can usually be determined in advance by weather forecasts but the last one, the river height and current speed, is not easily obtainable. It's also one of the more common reasons for cancellations. 

## River 

## Data sources
To be able to do any forecasts, we of course need data. To ingest data from different sources, one viable method is to use google [sheets](https://www.google.co.uk/sheets/about/). This online spreadsheet system has several useful functions we will use to automate the ingestion of data. I set up a system of google sheets that collected river information hourly and then after letting this run for a few months I performed the analyses. This was primarily because for some of the sources there was no way to download historical data, only current data. 

### Data availability

Another important consideration is when data is available. Checking the UK government hydrology data explorer found [here](https://environment.data.gov.uk/hydrology/explore) we can see three recording stations upstream of the centre of Cambridge that could be useful:
1. Burnt Mill - River Rhee
1. Stapleford - River Granta
1. Dernford - River Cam
However, the only way to obtain measurement values from these stations has a time lag of approximately 2 days. One can determine the width of the river at the monitoring station using the NRFA (National River Flow Archive) site, for instance for the river [Granta](https://nrfa.ceh.ac.uk/data/station/info/33053) we can access rough size estimates and pictures. Using these as well as typical river heights from the government hydrology site we can determine rough estimates of the cross sectional area of the rivers. We can also get river flow volume data from the nrfa and hence determine the river speed (roughly). Doing so and using google maps to estimate the distances, one can find that the water would flow between the station and the portion of the Cam we're concerned about in between 6 and 24 hours, so the two day time lag makes this information useless for forecasting as it is available too late.

### River levels
For the portion of the Cam where rowing takes place, there are three nearby stations for which data could be relevant for forecasting and d
One good site to consider for obtaining river height data across England, Scotland and Wales is [riverlevels.uk](https://riverlevels.uk/) which displays data for all river level monitoring stations. For the portion of the Cam that rowing outings usually take place on, the relevant monitoring station is the Jesus Lock Sluice Auto Cambridge Monitoring Station, which can be found [here](https://riverlevels.uk/jesus-lock-sluice-auto-cambridge-cambridgeshire). River levels UK only displays the current value on the site, however past values can be downloaded in the form of a csv. To import this in sheets we use the sheets function:

```=IMPORTXML("https://riverlevels.uk/jesus-lock-sluice-auto-cambridge-cambridgeshire","/html/body/div[4]/div[1]/div/div[1]/h2/span")```

The first entry is the link to the website, and the second entry is the XML path to the part of the webpage that contains the river level value. One can determine this path more generally by using inspect element, then finding the html code corresponding to that part of the website. Next, one can right click on this portion and click "Copy XML path" to get the "html/body/div..." code for that component of the webpage. 

There are other parts of the river we also want to collect data for. These are the height just above Jesus Lock, and just below Baitsbite Lock. The sluices above stream are adjusted to keep the height approximately the same, but the small variations may still have explanatory power. We get these data sources directly from the government Hydrology API. In the sheets implementation we import this data using code like:

```=IMPORTDATA("http://environment.data.gov.uk/flood-monitoring/id/measures/E60501-level-stage-i-15_min-mASD/readings?latest")```

This gets the most recent value as a JSON file which then spans multiple cells in Google Sheets. We're concerned with the value of that monitoring station, which can be found several lines into the JSON file we import. If one wants to look at more data, then they can write into a cell in sheets:

```=IMPORTDATA("https://environment.data.gov.uk/flood-monitoring/id/measures/E60502-level-downstage-i-15_min-mAOD/readings?_sorted&_limit=2200")```

And then we can change the *_limit* parameter to vary how many records are shown. One can then reorder and transform this data to get the most recent n samples from that monitoring station. For my sheets implementation I only used the most recent values so this was not needed, but for another project it may be helpful to go back into the past using a large *_limit* value.

I then put all of these into a single row in sheets, with columns:
- Date
- Time
- Flag (the colour of the flag at that time)
- Height (Just below Jesus Lock)
- Above JL Height (Height just above Jesus Lock)
- Baitsbite Height (Height just below Baitsbite Lock)

This row updates live, and changes every hour. We want to store this data, which can be done by using Apps Script to automatically record the data hourly. I set up a trigger that runs every hour, and on the trigger it runs the following function (written in TypeScript): 

```TypeScript
function record_data(){
  var Daily_data = SpreadsheetApp.getActive().getSheetByName("Hourly data")
  var data_range = Daily_data.getRange(2,1,15,10).getDisplayValues()
  var Records = SpreadsheetApp.getActive().getSheetByName("Records")
  for (let i=0; i < data_range.length; i++){
    Records.appendRow(data_range[i])
  }
} 
```
This copies the line the hourly updated data is on in the Hourly data sheet to the bottom of the Records sheet. For other projects where you have different amounts of data, the same approach will work but you may need to get a different range in the *Daily_data.getRange(2,1,15,10)* part of the code.

### Historical Weather data
There are several options for weather sources. I opted to use the Cambridge computer lab's raw daily weather data, which can be found [here](https://www.cl.cam.ac.uk/weather/index-daily-text.html). This source was chosen because of it's low latency. One can get the data for Monday on Tuesday just after midnight. One can also obtain the current weather conditions on a different page found [here](https://www.cl.cam.ac.uk/weather/). Other sources generally had some small time lag that made them less useful for short term forecasting. We will now discuss the data provided by this source. Data is stored as text files for each day, and has the following columns:
- Time
- Temp - Temperature (in degrees celsius)
- Humid - Humidity (in % from 0 to 100)
- DewPt - Dew Point (in degrees celsius)
- Press - Pressure (in milibars)
- WindSp - Wind speed (in knots)
- WindDr - Wind direction (takes values like "N" for north)
- Sun - Cumulative number of hours of sum (values in \[0,24\])
- Rain - Cumulative amount of rainfall (in milimeters) (One should note that in some files it is cumulative, and in others it is not)
- Start - Start time of the recording for that day (time, e.g. 00:00)
- MxWSpd - Maximum wind speed (in knots)
Of these, domain experience can tell us that the only column that may be relevant for forecasting the river level is the cumulative amount of rainfall. If one wished instead to create predictions of the river flag colour then the Humidity, Wind speed and Max wind speed may also be relevant.

### Importing the Weather data
One approach is to import the current data using:

```=IMPORTDATA("https://www.cl.cam.ac.uk/weather/txt/weather.txt")```

And add it as an extra column on our sheet to be recorded daily. One can also download the historical data since 1995 as a csv from the site and then transform this data into our desired output in sheets. The Computer Lab data is half hourly, so we will sum the rainfall to get the total amount each hour.  

This has half hourly weather details, but we have hourly data for the river levels. 

## Exploratory Analysis
