---
title: Time series forecasting with Weka
date: 2013-11-24 23:51
comments: true
categories: weka, programming
---

Here's an example of programmatically getting predicted values' date and time using Weka and pentaho's Java APIs. Recently, I've had to use Weka via their Java libraries to predict future electricity prices. What bothered me was that getting the forecasted price was easy, but getting the date and time of that forecasted price deemed more tricky than expected.

This example builds on pentaho's own example on predicting future wine sales numbers for "Fortified" and "Dry-white" based on data from previous sales.

You will need the pentaho Time Series Analysis plugin, which can be downloaded via the package manager included in Weka >= 3.7.3. It includes the Java libraries and documentation needed to follow their example and the following example.

This simple example also loads in the amazing Joda-Time library for Java that replaces Java's classes to manage time (link [here](http://www.joda.org/joda-time/)). The API is amazing and has a collection of really helpful helper classes that make managing time much much easier. This library, while not required, is used in the following to make sense of the value you get back from the forecaster: number of milliseconds since the Unix epoch.

I'm sure you can figure out what to do if you wish not to use Joda-Time.

The basic layout is exactly the same as pentaho's API example [here](http://wiki.pentaho.com/display/DATAMINING/Time+Series+Analysis+and+Forecasting+with+Weka#TimeSeriesAnalysisandForecastingwithWeka-4UsingtheAPI), bar for a couple lines. Here we're interested in getting the datetime object representing the forecasted data. That is done by adding the following two methods to their example:

{% highlight java %}
  private DateTime getCurrentDateTime(TSLagMaker lm) throws Exception {
    return new DateTime((long)lm.getCurrentTimeStampValue());
  }

  private DateTime advanceTime(TSLagMaker lm, DateTime dt) {
    return new DateTime((long)lm.advanceSuppliedTimeValue(dt.getMillis()));
  }
{% endhighlight %}

The first method merely reads whatever the latest time the forecaster knows about and makes turns it into a Joda-Time DateTime object.

The second method advances the forecaster with one unit, a single day in this case.

And here is the full copy-pastable example. All that is introduced, are the two methods above and a call to each on lines 61 and 77, respectfully:

```java
package dk.martinbmadsen.wekaTimeSeriesExample;

import java.io.*;

import java.util.List;

import org.joda.time.DateTime;
import weka.core.Instances;
import weka.classifiers.functions.GaussianProcesses;
import weka.classifiers.evaluation.NumericPrediction;
import weka.classifiers.timeseries.WekaForecaster;
import weka.classifiers.timeseries.core.TSLagMaker;

public class TimeSeriesExample {

  public static void main(String[] args) {
    try {
      // path to the Australian wine data included with the time series forecasting
      // package
      String pathToWineData = weka.core.WekaPackageManager.PACKAGES_DIR.toString()
          + File.separator + "timeseriesForecasting" + File.separator + "sample-data"
          + File.separator + "wine.arff";

      // load the wine data
      Instances wine = new Instances(new BufferedReader(new FileReader(pathToWineData)));

      // new forecaster
      WekaForecaster forecaster = new WekaForecaster();

      // set the targets we want to forecast. This method calls
      // setFieldsToLag() on the lag maker object for us
      forecaster.setFieldsToForecast("Fortified,Dry-white");

      // default underlying classifier is SMOreg (SVM) - we'll use
      // gaussian processes for regression instead
      forecaster.setBaseForecaster(new GaussianProcesses());

      forecaster.getTSLagMaker().setTimeStampField("Date"); // date time stamp
      forecaster.getTSLagMaker().setMinLag(1);
      forecaster.getTSLagMaker().setMaxLag(12); // monthly data

      // add a month of the year indicator field
      forecaster.getTSLagMaker().setAddMonthOfYear(true);

      // add a quarter of the year indicator field
      forecaster.getTSLagMaker().setAddQuarterOfYear(true);

      // build the model
      forecaster.buildForecaster(wine, System.out);

      // prime the forecaster with enough recent historical data
      // to cover up to the maximum lag. In our case, we could just supply
      // the 12 most recent historical instances, as this covers our maximum
      // lag period
      forecaster.primeForecaster(wine);

      // forecast for 12 units (months) beyond the end of the
      // training data
      List<List<NumericPrediction>> forecast = forecaster.forecast(12, System.out);

      DateTime currentDt = getCurrentDateTime(forecaster.getTSLagMaker());

      // output the predictions. Outer list is over the steps; inner list is over
      // the targets
      for (int i = 0; i < 12; i++) {
        List<NumericPrediction> predsAtStep = forecast.get(i);

        System.out.print(currentDt + ": ");

        for (int j = 0; j < 2; j++) {
          NumericPrediction predForTarget = predsAtStep.get(j);
          System.out.print("" + predForTarget.predicted() + " ");
        }
        System.out.println();

     	  // Advance the current date to the next prediction date
        currentDt = advanceTime(forecaster.getTSLagMaker(), currentDt);
      }

      // we can continue to use the trained forecaster for further forecasting
      // by priming with the most recent historical data (as it becomes available).
      // At some stage it becomes prudent to re-build the model using current
      // historical data.

    } catch (Exception ex) {
      ex.printStackTrace();
    }
  }

  private static DateTime getCurrentDateTime(TSLagMaker lm) throws Exception {
    return new DateTime((long)lm.getCurrentTimeStampValue());
  }

  private static DateTime advanceTime(TSLagMaker lm, DateTime dt) {
    return new DateTime((long)lm.advanceSuppliedTimeValue(dt.getMillis()));
  }
}
```

This post is inspired by a thread on pentaho's formus [here](http://forums.pentaho.com/showthread.php?89640-Weka-Programmatically-Get-Dates-along-with-Predicted-values). If you have any questions or comments, feel free to email me (me [at] martinbmadsen.dk) or write a comment on this page, and I will attempt to help you as much as possible.


Hope this helps!
