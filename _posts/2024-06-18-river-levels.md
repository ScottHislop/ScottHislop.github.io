# River level forecasting

In this article I explore an approach to producing forecasts of river level heights. I will discuss the process of finding and storing the data (for free) and discuss different ways to produce the forecasts. Lastly, we implement this in production so it can be accessed by potential users.

## Motivation
While studying in Cambridge, I was involved in college rowing as a rower and a cox. Outings were often cancelled due to poor weather conditions and this usually happened with relatively short notice. Rowing on the Cam (the main river in Cambridge) is governed by the [CUCBC](https://www.cucbc.org/) (Cambridge University Combined Boat Clubs). Outings are typically cancelled due to something called the [flag](https://www.cucbc.org/handbook/flag) which is set to different colours depending on how safe it is to go out on that day. Typically, conditions that lead to an outing being cancelled are:
<ul>
<li> High wind speeds (35mph or so)
<li> Fog / Mist leading to poor visibility
<li> High water levels or fast flowing water
<li> Extreme cold
</ul>
