+++
title = "Geocoding addresses in Talend Open Studio for Data Integration"
date = "2015-04-18T00:00:00+00:00"
tags = ["Talend", "Java"]
+++

Last week I had to geocode the addresses in a dataset using Talend. Geocoding is adding coordinates to addresses. This way you can easily visualize your data on a map.

I experienced problems when I tried to download some plugins from Talend Exchange but I found [this blog post](http://datacatalyst.blogspot.be/2011/01/custom-google-geocoder-using-talend.html) about geocoding without the use of a plugin. The blogpost wasn't very clear so I decided to write a step-by-step explanation which you can find below. In this guide, we will be using the [Google Geocoding API](https://developers.google.com/maps/documentation/geocoding/).

  1.	Map your location dataset (with a *tMap*) to 2 columns: an unique ID of your location and a concatenated string containing the location address. I used one of Talend's built-in *StringHandling*-functions.

          StringHandling.CHANGE(LOCATIONS.STREET_ADDRESS + LOCATIONS.POSTAL_CODE + LOCATIONS.CITY," ","+")

      My first column was called *LOCATION_ID* and my second column was called *LOCATION_CONCAT_ADDRESS*. Make sure to remember the name of your output. I named mine *LocConcatOut*.

  1.	Next, connect your output from the *tMap* to a *tFlowToIterate* component to start processing all records one by one.

  1.	The output from the *tFlowToIterate* has to be connected to a new *tFileInputJson* component. You can change the number of parallel iterations to speed up the geocoding. In the *tFileInputJson* component, check the *Use Url* option and enter the following URL:

            https://maps.googleapis.com/maps/api/geocode/json?address= + ((String)globalMap.get("LocConcatOut.LOCATION_CONCAT_ADDRESS"))

      Add `+ "&key=YourAPIkey"` if you would like to use your own API key for the [Google Geocoding API](https://developers.google.com/maps/documentation/geocoding/).

      Edit the schema for this component and make sure it contains 2 columns: *LATITUDE* and *LONGITUDE*. Their type should be float, nullable.

      In the mapping, *LATITUDE* should be set to `"$.results[0].geometry.location.lat"` and *LONGITUDE* should be set to `"$.results[0].geometry.location.lng"`.

  1.	Now we're going to map the coordinates back to the UIDs. Add another *tMap* component at the end. Create a new output, map the 2 coordinates from the *tFileInputJson* to identical columns in the new *tMap* and create a new column with a primary key. Enter the following for the value of this key (my keys are integers):

            ((Integer)globalMap.get("LocConcatOut.LOCATION_ID")).intValue()

  1.	Finally connect the output from the *tMap* component to a *tBufferOutput*. That ends the loop started with the *tFlowToIterate* component.

  1.	The previously mentioned components should all be put in the same subjob. Use a trigger like *OnSubjobOk* to start your next job after this one is completed. You can find all the location identifiers with their coordinates in the *tBufferInput* component. This is easily mapped (joined) to the original locations dataset using a *tMap*. Make sure to set the correct data types in the mapping (1 integer and 2 floats) for the *tBufferInput* component.
