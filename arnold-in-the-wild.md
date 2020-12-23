Who’s Arnold?
-------------

Smart Energi is a cooperation between a number of District Heating
companies in Sweden. Together they have developed the application `K2`
which is used for outlier detection in District Heating systems. For
each metering point in a DH-system `K2` will detect outliers and provide
an Energy Analyst with a alarms if outliers are detected. `K2` uses an
algorithm developed by `Chalmers University` called `Kasper` but it is
somewhat enhanced by `Smart Energi`.

The algorithm `Arnold` is a development of the `Kasper`-algorithm. One
difference between the two algorithms is that `Arnold` uses a non linear
regression technique called `Generalized Additive Models`(GAM) while
`Kasper` uses a piecewise regression. However, the main distinction
between the two algorithms is that `Kasper` needs reference data (that
should be manually selected by the Energy Analyst) while `Arnold` does
not, although `Arnold` can be configured to use reference data if
needed.

If you want to know more about `Arnold`, you can read an introduction to
it here: And Ida Lundholm Benzi have written an excellent explanation of
`Kasper` here:

In this Notebook we’ll explore how `Arnold` works on a larger dataset.

Data from Öresundskraft
-----------------------

In the BRAVA Datalake we have data that originally comes from
Öresundskraft, but it is provided to BRAVA by Halmstad University who
have used the data in the development of their outlier detection
algorithm that you can read more about in their notebook [Conformal
anomaly sequence detection via the reference-group based approach: a
study on district heating
substations](https://kyso.io/smartenergi/conformal-anomaly-sequence-detection-via-the-reference-group-based-approach-a-study-on-district-heating-substations?team=smartenergi).

The data consist of hourly district heating consumption with around 12
000 metering points.

For this Notebook we’ll limit ourselves to using Arnold on daily
`energy` data and not `flow`. This is the same data that is used in K2.
Furthermore, daily data drastically reduces the computation needed to
perform outlier detection

    con <- dbConnect(odbc::odbc(), Driver = "Snowflake", Server = "lq42418.eu-west-1.snowflakecomputing.com", 
        uid = "filipw", Database = "SMARTENERGI_DATALAKE_SANDBOX", pwd = keyring::key_get("smartenergi"))

    tbl(con, in_schema("PUBLIC", "ORKAFT_METER_READINGS_TBL")) %>% 
      filter(PROPERTY == "energy") %>% 
      head()

    ## # Source:   lazy query [?? x 5]
    ## # Database: Snowflake 4.42.2[filipw@Snowflake/SMARTENERGI_DATALAKE_SANDBOX]
    ##   METERING_POINT_ID               date       PROPERTY UNIT_OF_MEASURE mean_value
    ##   <chr>                           <date>     <chr>    <chr>                <dbl>
    ## 1 676da7d4-70ac-11ea-ae00-062fc1… 2016-06-21 energy   MWh                   1.5 
    ## 2 676da7d4-70ac-11ea-ae00-062fc1… 2016-06-25 energy   MWh                   1   
    ## 3 aa6d500a-70ae-11ea-ae00-062fc1… 2016-07-23 energy   MWh                   1.43
    ## 4 aa6d500a-70ae-11ea-ae00-062fc1… 2016-09-28 energy   MWh                   1.14
    ## 5 aa6d500a-70ae-11ea-ae00-062fc1… 2016-11-06 energy   MWh                   2.71
    ## 6 9ecf5b20-70af-11ea-ae00-062fc1… 2016-01-31 energy   MWh                  14.9
