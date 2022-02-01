---
title: "Sdeng: the full requirements"
categories:
- "Sdeng"
---

Welcome back! In the [introduction]({% post_url 2022-01-01-sdeng-introduction %}) we described the goal of this project. It's now time to formalize this goal into software requirements.

The first functionalities to design are the persistence operations. In a second moment we will move to more advanced features such as plotting.

As far as persistence is concerned, the user can choose among two main data formats: relational databases or language-specific dataframes. More specifically, pandas dataframes for python, and, in a second moment, R dataframes. I did not include csv files since they can be easily obtained from dataframes. 

In a query, the user can choose among many filters and request types.

Now, a naming clarification: there are download formats (or data formats), which are the ways we want to store data, namely relational databases or data frames. Then there are requests formats, that indicate which kind of data to download, for instance players statistics, play-by-play logs, or games box-scores. Finally, there are requests filters, which allow specifying additional details to the request, such as the season or the team.

As you can imagine, there are dozens of different request types, and embedding all of them in the code would be impossible. Moreover, we must consider how different data types have different goals: relational databases allow complex queries but a bounded structure, while dataframes offer immediateness.

Thus said, different data formats are most suited for different request types. Since all statistics are computed starting from play-by-play logs, in the database case only play-by-play will be allowed as request type. This may seem a bit excessive, but is based on the idea that a database will be reused for different queries, and storing data in the same format will eventually be more practical.

Clearly, some data cannot be retrieved in play-by-play logs, for instance awards, contracts, staff members, while other information, such as dictionaries or season, should be set up before starting, since in most cases are independent of downloaded data. We will get back to this.

Data frames, on the other hand, given their flexibility, will allow all request formats. Of course, since not all leagues have advanced APIs, in some cases fetching complex information will be time-consuming, since data must be downloaded integrally and later transformed.


| requirement id | requirement                                                                                           | type           |
|----------------|-------------------------------------------------------------------------------------------------------|----------------|
| FR1            | The library allows downloading data from different leagues                                            | Functional     |
| NFR1           | Adding new leagues should be as easy as possible                                                      | Non functional |
| FR2            | Data can be download in many formats, for instance relational databases, or language-specific objects | Functional     |
| NFR2           | Adding new download formats should be as easy as possible                                             | Non functional |
| FR3            | User may request different types of data, such as players stats, team stats etc.                      | Functional     |
| NFR3           | Adding new request types should be as easy as possible                                                | Non functional |
| FR4            | Downloading data to a relational database does not allow custom request types                         | Functional     |
| FR5            | Downloading data to a csv file or to an object allow custom request types                             | Functional     |
| FR6            | The library allows loading data once saved locally                                                    | Functional     |
| FR7            | Data is loaded to a language-specific object, namely a pandas dataframe or an R dataframe             | Functional     |
| FR7            | Data can be loaded only if it was previously downloaded                                               | Functional     |
| FR8            | Request types should allow as many filters as possible                                                | Functional     |
| NFR4           | Adding new filters should be as easy as possible                                                      | Non functional |
| FR9            | User can load data from a relational database using custom queries                                    | Functional     |
| NFR5           | Library interface should be as similar as possible to pandas' one                                     | Non functional |
| NFR6           | Download should be as fast as possible                                                                | Non functional |

Since csv files can be easily obtained from data frames, I did not include csv in the allowed data formats.

Summing up, there are three main use cases:
1. download to sql
2. download to dataframe
3. load from sql

To which we can add some useful util functions:
4. download to sql non pbp data, such as contracts, staff members, awards
5. download all profile pics of players, useful for plotting and visualization

## Scenarios

We will now analyze the main scenarios for these use cases.

| Title                       | Download to SQL                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Description                 | Data are downloaded from leagues' websites to a SQL database                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| Actors                      | User, Websites                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| Relations                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| Preconditions               | A database exists                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| Postconditions              | The database is filled with required data                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| Main scenario               | 1. The user asks for a set of games for a set of leagues<br>2. The system checks that the leagues are valid<br>3. The system downloads the play-by-play logs<br>4. The system cleans the play-by-play logs according to the database format<br>5. The system checks that all referenced tables in the log exists<br>6. In case a referenced table does not exist, the system fetches the required data and repeats the points 4 and 5 recursively, then inserts the missing data<br>7. The system inserts the play-by-play cleaned logs in the database<br>8. The system outputs the result of the insertion |
| Alternative scenarios       | Invalid league<br>1. The system outputs the allowed leagues and proceeds with the other leagues in the set<br>Empty data (logs not tracked)<br>1. The system notifies the user about missing play-by-play logs, and ask if he still wants to proceed saving all other data                                                                                                                                                                                                                                                                                                                                   |
| Non functional requirements | NFR6: Downloads should be as fast as possible                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Questions                   |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |

<br>

| Title                       | Download to data frame                                                                                                                                                                                                                                                                                             |
|-----------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Description                 | Data are downloaded from leagues' websites to a pandas (or R) data frame                                                                                                                                                                                                                                           |
| Actors                      | User, Websites                                                                                                                                                                                                                                                                                                     |
| Preconditions               |                                                                                                                                                                                                                                                                                                                    |
| Postconditions              | A data frame is returned to the user                                                                                                                                                                                                                                                                               |
| Main scenario               | 1. The user asks for a specific table with various filters<br>2. The system checks that the request is allowed<br>3. The system fetches the necessary data to complete the request<br>4. The system transforms the data in the appropriate way<br>5. The system returns the data in the required format            |
| Alternative scenarios       | Invalid request<br>1. The system outputs the allowed requests and stops<br>Invalid filter (team, season, player, etc)<br>1. The system notifies the user that the filter is invalid and stops<br>Empty data<br>1. The system notifies the user that his request produced no results and return an empty data frame |
| Non functional requirements | NFR3: Adding new request types should be as easy as possible<br>NFR4: Adding new filters should be as easy as possible<br>NFR6: Downloads should be as fast as possible                                                                                                                                            |

<br>

| Title                       | Load from SQL                                                                                                                                                                                                                                                                                                                 |
|-----------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Description                 | Data are downloaded from the SQL database to a pandas (or R) data frame                                                                                                                                                                                                                                                       |
| Actors                      | User, Database                                                                                                                                                                                                                                                                                                                |
| Preconditions               | A database exists                                                                                                                                                                                                                                                                                                             |
| Postconditions              | A data frame is returned to the user                                                                                                                                                                                                                                                                                          |
| Main scenario               | 1. The user asks for a specific table with various filters<br>2. The system checks that the request is allowed<br>3. The system fetches the necessary data to complete the request<br>4. The system executes the query associated with the request using the filters<br>5. The system returns the data in the required format |
| Alternative scenarios       | Invalid request<br>1. The system outputs the allowed requests and stops<br>Empty data<br>1. The system notifies the user that his request produced no results and return an empty data frame<br>Custom query<br>1. The user defines a custom query and executes it                                                            |
| Non functional requirements | NFR3: Adding new request types should be as easy as possible<br>NFR4: Adding new filters should be as easy as possible                                                                                                                                                                                                        |


These use cases are the functions we are going to develop in a first moment. Later we will move to more complex functionalities, such as analysis and visualization. These functions will be our `io` package.

This is all for today! In the next chapters we will analyze the domain from which we will develop the relational database. Once we do that, we will be able to fully describe and design these functions, defining the parameters and the full behaviour.
