# Analysis of Residential Data

## Jeremy Chow
## 9/19/2019
**1. What data would you exclude from analysis for being unreliable or potentially a block
instead of an actual booking?**

I’m going to first approach this from a different angle: what are the indicators of a real booking?
The strongest indicator of a legitimate booking we can see in this dataset is when the price for a
day drops over multiple scraped dates, and the availability goes from True to False sometime
during those drops. This highly suggests that the cost eventually met the demand, and as a
result someone booked the location. This solution could be foiled if a host had their
SmartPricing option turned on or were manually adjusting prices, then they decided to block off
a date arbitrarily. Perhaps a model could be generated based on square footage, number of
bathrooms and bedrooms, and location to predict a target price, then that target price could be
compared to the actual price. If the actual price is significantly higher for a long period, it could
suggest a block.


On the other hand, if a guest already booked a place, the price should not change following the
booking. However, this behavior occurs in 3.6% of the property-date combinations. These
properties were listed as unavailable the entire time frame but the price changed at some point,
a behavior that never appears to happen using the above logic for legitimate bookings. Of
course, it is possible that a place scheduled two long term guests back-to-back, which is why we
should be cautious before removing this data.
Another arguably strong indicator would be looking at the date when the True to False
Availability switch occurs. Hosts are penalized for cancelling guest bookings, and thus they
would be incentivized to schedule their blocks earlier rather than later. According to an AirBnB
article by PhocusWire in 2016:


_“Around 20% of properties will be booked two months in advance, with this figure
reaching 80% at ten days out.”_


Given this information, a logical host would likely put a block on certain dates possibly months in
advance, and therefore the closer the switch occurs to the booking date, the more likely it is a
valid booking rather than a block.
Another useful factor would be the name of the host. Assuming blocks occur due to personal
reasons, hosts who are companies that owns the properties they post (not just manage) would
be less likely to have blocks unless they were remodeling, but that would be very rare and may
require human curation of the dataset.


The superhost column is interesting because while becoming a superhost doesn’t seem to have
any direct bearing on blocking metrics being based on ratings, reply rate, low cancellations, and
number of bookings, it results in more promotion and exposure by airbnb, which means any
unavailable date on a superhost-owned property is more likely to be guests.


From a different angle, there is also some information I would exclude while looking for blocks
vs. guests. This probably goes without saying, but the URL’s are not very helpful on their own.
Also without manual curation, the host name column in the listing dataframe has low signal due
to it only using the first names, which highly overlap between different hosts. Ideally we would
want a host_id to replace this, but of course that is difficult with scraped data. However beyond
that, I could make an argument for how all the other columns may have an impact on blocks
whether it be larger homes and a cleaning fee suggest a day blocked off for cleaning, busier
locations mean hosts are less willing to block, even the name of the listing could possibly be
used for signal given a dataset of labeled blocks vs non-blocked listings and an NLP
classification model.


Ultimately, erring on the side of caution, there is no 100% definitive way to rule out unavailable
slot as a booking or a block. The threshold of filters and logic used to deem an unavailable slot
as non-revenue generating is dependent on how conservative of revenue estimates we are
willing to make. A solution beyond the dataset could be to do a random survey of hosts to find
the percentage of days they block on average in a month, then multiply our results by scaling
factor at the end of revenue calculations using all data available. We could do a more granular
scaling factor for groups such as superhosts vs. non-superhosts, corporations vs. individuals as
well.


References:
https://blog.pillow.com/crunching-the-numbers-defining-normal-for-airbnb-hosts-and-listings/
https://www.phocuswire.com/Number-crunching-reveals-best-time-to-book-on-Airbnb


For the problems ahead, I created a dataframe with one row for each property and booking date
combination. The price and availability were taken from the last scraped data for each
combination. Unique prices and availabilities were also recorded for brief analysis into the
effects of price changes and changes in availability over time for a particular date.


**2. What is a good approach to estimate occupancy and revenue per unit?**


Once we have aggregated the data (grouping by property and date such that there is one row
for each property and day combination) and make a daily boolean ‘was_occupied’ metric from
the above logic or other logic we implement, we can then create a new ‘revenue’ column
generated by multiplying the last known price for each property and date by the boolean
was_occupied column, which will either return the price or 0. From there, we can group by
property and sum this revenue column to find the revenue over the dataset.

Alternatively, as mentioned previously, we can calculate the revenue for all the data then
multiply the output by some “booked days divided by total days” scaling factor.

**3. Which month appears to be more profitable? April or May?**


For this and the next question, I did not filter the data and treated every unavailable slot as a
booking. The idea was that any bias introduced by blocking would be equivalent in both groups
being compared, and thus the effect would cancel out.


**4. How much more revenue do places with 3 bedrooms make vs. places with 2
bedrooms?**


Once again, blocking vs. guests problem was bypassed here under the assumption that
blocking doesn’t affect 3 bedroom vs. 2 bedroom places differently, thus doing a relative
comparison will presumably neutralize its effect.
With that logic, 3-bedroom places make about 8.42% more than 2-bedroom places.


**5. What are any other interesting insights you may have found?**
20% of the properties in the dataset are listed as unavailable for all dates, either due to long
term rentals or long term blocks.
Around 24% of booked dates in the dataset get cancelled or rescheduled.

Here are some quick visualizations:

![Top_Cities](data/top_5_cities.png)


Scottsdale seems to be the most popular city in this dataset with over half of the listings.


![Unavailablity_Hist](data/unavailability_hist.png)


There are a significant number of properties which were 100% available and 100% unavailable
during this time period. About 9.4% of the vacant properties were listed by superhosts, and
about 16.6% of the fully booked properties were listed by superhosts. Overall 24% of the listings

are by superhosts.


![Capacity_Hist](data/capacity_hist.png)


The mean capacity of units in this dataset is 6.05 with a median of 6 people. 25% of the units in

the dataset have a capacity of 4 people.

![Unavailable_Over_Time](data/unavailability_line.png)

A look at the trend of normalized booked/blocked over total number of units over time shows a
stable trend. The spikes seem to partially overlap with weekends. There seems to be an

anomaly around May 7th, possibly due to May 5 festivities.





