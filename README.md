# GCP-BigQueryML-ForecastingModel

• Use BigQuery to find public datasets

• Query and explore the public taxi cab dataset

• Create a training and evaluation dataset to be used for batch prediction

• Create a forecasting (linear regression) model in BigQuery ML

• Evaluate the performance of your machine learning model.

## Explore NYC taxi cab data
How many trips did Yellow Cab taxis take each month in 2015?

#standardSQL
SELECT
  TIMESTAMP_TRUNC(pickup_datetime,
    MONTH) month,
  COUNT(*) trips
FROM
  `bigquery-public-data.new_york.tlc_yellow_trips_2015`
GROUP BY
  1
ORDER BY
  1
Then click Run.

You should receive the following result:

![Image of CloudBuild](https://github.com/IamVigneshC/GCP-BigQueryML-ForecastingModel/blob/main/Result00.png)

As we see, every month in 2015 had over 10 million NYC taxi trips—no small amount!

Replace the previous query with AvgSpeed.sql

You should receive the following result:

![Image of CloudBuild](https://github.com/IamVigneshC/GCP-BigQueryML-ForecastingModel/blob/main/Result0.png)

During the day, the average speed is around 11-12 MPH; but at 5:00 AM the average speed almost doubles to 21 MPH. Intuitively this makes sense since there is likely less traffic on the road at 5:00 AM.

Create a machine learning model in BigQuery to predict the price of a cab ride in New York City given the historical dataset of trips and trip data. Predicting the fare before the ride could be very useful for trip planning for both the rider and the taxi agency.

## Select features and create your training dataset

The New York City Yellow Cab dataset is a public dataset provided by the city and has been loaded into BigQuery for your exploration. Browse the complete list of fields and preview the dataset to find useful features that will help a machine learning model understand the relationship between data about historical cab rides and the price of the fare.

Your team decides to test whether these below fields are good inputs to your fare forecasting model:

• Tolls amount

• Fare amount

• Hour of the day

• Pick up address

• Drop off address

• Number of passengers


Note a few things about the query:

The main part of the query is at the bottom (SELECT * from taxitrips).

taxitrips does the bulk of the extraction for the NYC dataset, with the SELECT containing your training features and label.

The WHERE removes data that you don't want to train on.

The WHERE also includes a sampling clause to pick up only 1/1000th of the data.

Define a variable called TRAIN so that you can quickly build an independent EVAL set.

total_fare is the label (what you will be predicting). You created this field out of tolls_amount and fare_amount, so you could ignore customer tips as part of the model as they are discretionary.

![Image of CloudBuild](https://github.com/IamVigneshC/GCP-BigQueryML-ForecastingModel/blob/main/Result1.png)


## Create a BigQuery dataset to store models

create a new BigQuery dataset which will store your ML models.

In the left-hand Resources panel, select your Qwiklabs Project ID.

Then on the right-hand side of the page, click CREATE DATASET.

In the Create Dataset dialog, enter in the following:

For Dataset ID, type taxi.
Leave the other values at their defaults.

Then click Create dataset.


## Select a BigQuery ML model type and specify options

Now that you have your initial features selected, you are now ready to create your first ML model in BigQuery.


Refer Train_Model_1.sql

Next, click Run to train your model.

Wait for the model to train (5 - 10 minutes).


After your model is trained, you will see the message "This was a CREATE operation. Results will not be shown" which indicates that your model has been successfully trained.

Look inside your taxi dataset and confirm taxifare_model now appears.


## Evaluate classification model performance

Select your performance criteria
For linear regression models you want to use a loss metric like Root Mean Square Error (RMSE). You want to keep training and improving the model until it has the lowest RMSE.

In BigQuery ML, mean_squared_error is a queryable field when evaluating your trained ML model. Add a SQRT() to get RMSE.

Now that training is complete, you can evaluate how well the model performs with this query using ML.EVALUATE.


Refer Evaluate.sql


You are now evaluating the model against a different set of taxi cab trips with your params.EVAL filter.

After the model runs, review your model results (your model RMSE value will vary slightly).

Row	 rmse

1	   9.477056435999074

After evaluating your model you get an RMSE of $9.47. Knowing whether or not this loss metric is acceptable to productionalize your model is entirely dependent on your benchmark criteria, which is set before model training begins. Benchmarking is establishing a minimum level of model performance and accuracy that is acceptable.


## Predict the taxi fare amount

Next you will write a query to use your new model to make predictions.


Refer Predict.sql

![Image of CloudBuild](https://github.com/IamVigneshC/GCP-BigQueryML-ForecastingModel/blob/main/Result2.png)

Now you will see the model's predictions for taxi fares alongside the actual fares and other features for those rides. 


## Improve the model with feature engineering

Building machine learning models is an iterative process. Once we have evaluated the performance of our initial model, we often go back and prune our features and rows to see if we can get an even better model.

Filter the training dataset
Let's view the common statistics for taxi cab fares.

Let's limit the data to only fares between 6 and 200 dollars.

You still have a large training dataset of over 800 million rides for our new model to learn from. Let's re-train the model with these new constraints and see how well it performs.

### Retrain the model

Let's call our new model taxi.taxifare_model_2 and retrain our linear regression model to predict total fare. You'll note that you've also added a few calculated features for the Euclidean distance (straight line) between the pick up and drop off.

SELECT
  COUNT(fare_amount) AS num_fares,
  MIN(fare_amount) AS low_fare,
  MAX(fare_amount) AS high_fare,
  AVG(fare_amount) AS avg_fare,
  STDDEV(fare_amount) AS stddev
FROM
`nyc-tlc.yellow.trips`
WHERE trip_distance > 0 AND fare_amount BETWEEN 6 and 200

![Image of CloudBuild](https://github.com/IamVigneshC/GCP-BigQueryML-ForecastingModel/blob/main/Result3.png)

Refer Retrain_Model.sql

### Evaluate the new model
Now that our linear regression model has been optimized, let's evaluate the dataset with it and see how it performs.

Refer Evaluate_retrained_model.sql

![Image of CloudBuild](https://github.com/IamVigneshC/GCP-BigQueryML-ForecastingModel/blob/main/Result4.png)

As you see, you've now gotten the RMSE down to: +-$$5.12 which is significantly better than +-$$9.47 for your first model.

Since RMSE defines the standard deviation of prediction errors, we see that the retrained linear regression made our model a lot more accurate.
