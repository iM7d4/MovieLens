# MovieLens

## Project Description
---
* This project is a case study for a start-up, involved with recommending movies to users as a service, as well as investigating certain factors contributing to the success of movies.
* The aim of this project, is to perform Extract, Transform, Load, on movies data, to answer questions the business may have about its users, such as:
    * What is the highest rated movie of all time?
    * Which genre of movies are the most popular with users?
    * Trends in box office earnings - Does releasing a movie at a certain quarter/month of a year, lead to higher box office earnings?
    * Which genres are the highest earning of all-time, normalized against a consumer price index?

* The movies data and metadata comes from Movielens, extracted from a [Kaggle dataset](https://www.kaggle.com/rounakbanik/the-movies-dataset). The data contains 26 million user ratings of over 270,000 users on a collection of over 45,000 movies.
* In addition, Consumer Price Index of Admission to Movies, Theaters, and Concerts in U.S. City Average is extracted from [Here](https://fred.stlouisfed.org/series/CUSR0000SS62031). This will help us normalize box office earnings against inflation over the years.

## Architecture
---
The technical architecture for this project is as show below:  
  
![Architecture]()

1. Data Extraction is done using Kaggle API and using GET request to St Louis Fred's CPI dataset.  
Set up an EC2 instance with python and pip installed. Then, run `pip install kaggle`. To download the movielens dataset, run 
```bash
kaggle datasets download -d "rounakbanik/the-movies-dataset"
```  
For St Louis Fred's Consumer Price Index dataset, run 
```bash
wget https://fred.stlouisfed.org/graph/fredgraph.csv?bgcolor=%23e1e9f0&chart_type=line&drp=0&fo=open%20sans&graph_bgcolor=%23ffffff&height=450&mode=fred&recession_bars=on&txtcolor=%23444444&ts=12&tts=12&width=1168&nt=0&thu=0&trc=0&show_legend=yes&show_axis_titles=yes&show_tooltip=yes&id=CUSR0000SS62031&scale=left&cosd=1999-01-01&coed=2020-04-01&line_color=%234572a7&link_values=false&line_style=solid&mark_type=none&mw=3&lw=2&ost=-99999&oet=99999&mma=0&fml=a&fq=Monthly&fam=avg&fgst=lin&fgsnd=2009-06-01&line_index=1&transformation=lin&vintage_date=2020-05-31&revision_date=2020-05-31&nd=1999-01-01
```

2. Next, copy the files downloaded from the EC2 instance to S3. Make sure that it has the aws-cli installed. Run `aws configure` and then `aws s3 cp {FILE} s3://{S3_BUCKET}/{S3_FOLDER}/` to transfer the files to S3. Note that if the situation changes such that this becomes a daily job, we can write a shell script containing these commands, and add the command to run this shell script in our Airflow data pipeline

3. Run the ETL pipeline, scheduled using Airflow. Data Processing is done using Spark, and data is eventually ingested into Redshift.

## Choice of Technologies
---
* For data processing (data transformation step), Spark is chosen because of its parellel processing capabilities. Should the amount of data proliferate to 100x, more worker nodes can be added to the spark cluster to scale out.

* For orchestrating the steps in the pipeline, Airflow is chosen as it allows building of data pipelines that are straghtforward and modular. Airflow allows tasks to be defined in a Directed Acyclic Graph (DAG) with dependencies of tasks between one another. This allows running of tasks to be optimized. It also enables the pipeline to be run on a schedule (for eg, daily) should the need arise. Finally, it has an intuitive UI that allows users to check the steps in the data pipeline should any part of the pipeline fail.

* Redshift is chosen as the cloud Data Warehouse as it is highly scalable. Should our data grow in size, we can provision more nodes or scale up, to handle the larger volume of data.

* Docker is used to encapsulate package dependencies the code may have, to allow the code to run on any machine
