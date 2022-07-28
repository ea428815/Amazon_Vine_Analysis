# Amazon_Vine_Analysis
This is a project analyzing Amazon reviews written by members of the paid Amazon Vine program.

## Perform ETL on Amazon Products Review

In this project, I picked a dataset on U.S. Beauty products from Amazon Review dataset https://s3.amazonaws.com/amazon-reviews-pds/tsv/index.txt

The U.S. reviews dataset on Beauty product link for this repository is as follows:

> https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Beauty_v1_00.tsv.gz


I used PySpark to read in a CSV file, to perform ETL process to extract the datasets, to get DataFrame information, to connect pgAdmin to a AWS RDS, to create RDS and AWS, to shut down AWS instancs and transform and filter data. Later, I used PySpark to determine if there is any bias toward the reviews from Vine members in the datasets.

### Deliverable1

The purpose of this project is to create 4 tables which are customers, products, Review_id, vine; to upload them in a AWS database by pgAdmin; and to analyze vine table.

## Results

### Perform ETL on Amazon Software Product Reviews

The database was named "database16" were created in AWS and the scheme of 4 tables were created in pgAdmin. See picture below:

<img width="1296" alt="Screen Shot 2022-07-28 at 8 53 50 AM" src="https://user-images.githubusercontent.com/62758795/181520327-0bc7bcf5-76f6-4574-80a4-b0fb193229df.png">

First, I extracted the review datasets, then created a new Dataframe using the code:

> from pyspark import SparkFiles
>
> url = Òhttps://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Beauty_v1_00.tsv.gzÓ
> 
> spark.sparkContext.addFile(url)
> 
> df = spark.read.option("encoding", "UTF-8").csv(SparkFiles.get("amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Beauty_v1_00.tsv.gz "), >sep="\t", header=True, inferSchema=True)\

Data frame customers_df was created by the code

> customers_df = df.groupby("customer_id").count().withColumnRenamed("count", "customer_count")

The first 20 rows of the dataframe is below.

<img width="265" alt="Screen Shot 2022-07-28 at 12 50 18 AM" src="https://user-images.githubusercontent.com/62758795/181503545-7f236fe7-e4be-473f-968d-d02ddc9ab8a8.png">

DataFrame products_df was created using the code:

> products_df = df.select(["product_id","product_title"]).drop_duplicates()

The first 20 rows of the DataFrame is below.

<img width="285" alt="Screen Shot 2022-07-28 at 12 50 39 AM" src="https://user-images.githubusercontent.com/62758795/181506680-3023317f-0e7d-45e0-ac0a-3554f4c45e09.png">

DataFrame review_id_df was created using the code:

> review_id_df = df.select(["review_id","customer_id","product_id","product_parent", to_date("review_date", 'yyyy-> MM-dd').alias("review_date")])

The first 20 rows of the dataframe is below.

<img width="570" alt="Screen Shot 2022-07-28 at 12 51 00 AM" src="https://user-images.githubusercontent.com/62758795/181506329-44b43aae-13bd-4ead-be76-642cda643d8d.png">

DataFrame vine_df was created using the code:

> vine_df = df.select(["review_id","star_rating","helpful_votes","total_votes","vine","verified_purchase"])

The first 20 rows of the dataframe is given below.

<img width="657" alt="Screen Shot 2022-07-28 at 12 51 30 AM" src="https://user-images.githubusercontent.com/62758795/181507302-7088be73-80a1-4ed2-a7ca-6c484fc88e19.png">

Note that, I connected the AWS RDS instance (database16) to load the tables using the code:

> mode = "append"
> 
> jdbc_url="jdbc:postgresql://database16.cbchqixwjhxz.us-east-2.rds.amazonaws.com:5432/database16"
> 
> config = {"user":"postgres", 
> 
<img width="885" alt="Screen Shot 2022-07-28 at 12 55 07 AM" src="https://user-images.githubusercontent.com/62758795/181505231-ac89f981-dc70-4f4b-aee6-d52ffd874eb0.png">

When Loading review_id_df to table in RDS, use the code:

> review_id_df.write.jdbc(url=jdbc_url, table='review_id_table', mode=mode, properties=config)

This is what the table looks like on pgAdmin:

<img width="1296" alt="Screen Shot 2022-07-28 at 9 01 05 AM" src="https://user-images.githubusercontent.com/62758795/181521008-db43ed33-a717-4680-aed1-ec7c1aa63aff.png">

Loading products_df to table in RDS, the code is

> products_df.write.jdbc(url=jdbc_url, table='products_table', mode=mode, properties=config)

This is what the table looks like on pgAdmin:



Loading customers_df to table in RDS, the code is

> customers_df.write.jdbc(url=jdbc_url, table='customers_table', mode=mode, properties=config)

This is what the table looks like on pgAdmin:

<img width="1296" alt="Screen Shot 2022-07-28 at 9 57 46 AM" src="https://user-images.githubusercontent.com/62758795/181525832-33725ff4-2c7e-4dea-a375-4d0701053c5c.png">

Loading vine_df to table in RDS, the code is

> vine_df.write.jdbc(url=jdbc_url, table='vine_table', mode=mode, properties=config)

This is what the table looks like on pgAdmin:



You can find the entire code in the Main branch: ![Amazon_Software_Reviews_ETL](Amazon_Reviews_ETL.ipynb)

###Data Extraction

The dataset was extracted by the code

>from pyspark import SparkFiles
>
>url = "https://s3.amazonaws.com/amazon-reviews-pds/tsv/amazon_reviews_us_Software_v1_00.tsv.gz"
>
>spark.sparkContext.addFile(url)
>
>df = spark.read.option("encoding", "UTF-8").csv(SparkFiles.get("amazon_reviews_us_Software_v1_00.tsv.gz"), >sep="\t", header=True, inferSchema=True)

The wine table was created by the code 

> vine_df = df.select(["review_id","star_rating","helpful_votes","total_votes","vine","verified_purchase"])

The dataframe was filtered according to "total_votes" greater than or equal to 20 by the code

> tv_more_20_df=vine_df.filter(vine_df['total_votes']>=20)

The data frame was filtered according to the percentage of "helpful_vote" greater than or equal to 50 as

> vine_new_1_df=tv_more_20_df.filter(tv_more_20_df["helpful_votes"]/tv_more_20_df['total_votes']>=0.5)

The data frame vine_paid was created by

> vine_paid_df=vine_new_1_df.filter(vine_new_1_df['vine']=="Y")

The data frame vine_unpaid was created by

> vine_unpaid_df=vine_new_1_df.filter(vine_new_1_df['vine']=="N")

Finally, the following codes were written:

> **The number of paid reviews:**
> 
> paid_count=vine_paid_df.count()

> **The number of unpaid reviews:**
> 
> unpaid_count=vine_unpaid_df.count()
>
> **The number of paid 5-star reviews:**
> 
> five_star_paid_count=vine_paid_df.filter(vine_paid_df["star_rating"]==5).count()
>
> **The number of unpaid 5-star reviews:**
> 
> five_star_unpaid_count=vine_unpaid_df.filter(vine_unpaid_df["star_rating"]==5).count()
>
> **The percentage of paid 5-star reviews:**
> 
> five_star_paid_percent=round(100*five_star_paid_count/paid_count,0)
>
> **The percentage of unpaid 5-star reviews:**
> 
> five_star_unpaid_percent=round(100*five_star_unpaid_count/unpaid_count,0)
>
> **The data frame containing all of the above counts**
>
> counts_df=spark.createDataFrame([('paid',paid_count,five_star_paid_count,five_star_paid_percent),
> 
>                                 ('unpaid',unpaid_count,five_star_unpaid_count,five_star_unpaid_percent)],
>                                 
>                                ['vine','total_count','five_star_count','five_star(%)'])

As a result, there are 248 paid reviews from which 102 of them are 5-star. 5-star reviews are 41 % of the total paid reviews. Similarly, there are 17514 unpaid reviews from which 5154 of them are 5-star. 5-star reviews are 29 % of the total unpaid reviews.  

These results can be summarized by the following table:



## Summary

Although the number of paid reviews were smaller than the number of unpaid reviews, observe that the percentage of 5-star paid review was larger than the percentage of 5-star unpaid reviews.





