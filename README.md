# data-512-homework_2
This notebook constructs and analyzes a dataset from the page info and ORES APIs from English Wikimedia.

The goals of this project are:
1. Create a repository, code, and analysis which focuses on implementing the best practices described in
   ["Assessing Reproducibility"](http://www.practicereproducibleresearch.org/core-chapters/2-assessment.html)
   and ["The Basic Reproducible Workflow Template"](http://www.practicereproducibleresearch.org/core-chapters/3-basic.html).
2. Construct 6 tables which provide insight into bias in data:
   1. Top 10 countries by coverage.
      - The 10 countries with highest total articles per capita (in descending order).
   2. Bottom 10 countries by coverage.
      - The 10 countries with lowest total articles per capita (in descending order).
   3. Top 10 countries by high quality
      - The 10 countries with highest high quality articles per capita (in descending order).
   4. Bottom 10 countries by high quality.
      - The 10 countries with the lowest high quality articles per capita (in ascending order).
   5. Geographic regions by total coverage.
      - A rank ordered list of geographic regions (in descending order) by total articles per capita.
   6. Geographic regions by high quality coverage.
      - Rank ordered list of geographic regions (in descending order) by high quality articles per capita.

## Dependencies
For replication and reproduction of this data set construction and analysis, the Python version and packages are
listed here:

- Python 3.10.11
- Jupyter 1.1.1
- Polars 1.9.0
- Pandas 2.2.3
- Pyarrow 17.0.0
- Numpy 2.1.1
- Pyarrow 17.0.0


## Folder Structure:
The repository is divided into two folders:
```bash
data-512-homework_2
├── data
│   ├── articlequality.csv
│   ├── politician_page_data.csv
│   ├── politicians_by_country_AUG.2024.csv
│   ├── population_by_country_AUG.2024.csv
│   ├── wp_countries-no_match.txt
│   └── wp_politicians_by_country.csv
├── LICENSE
├── README.md
└── construct_and_analyze_bias.ipynb
```

## Data Source Information and Disclosures

Wikimedia's data is provided under the [CC0 1.0 License](https://creativecommons.org/publicdomain/zero/1.0/).
Abiding by Wikimedia Foundations' [terms of use](https://foundation.wikimedia.org/wiki/Policy:Terms_of_Use),
this project does its best to provide a dataset and accompany basic analysis that is accurate to the data and
not intentionally misleading or false. The authors of the project receive no contribution from any other party that
is a conflict of interest.

Wikimedia Foundations' API can be found here: https://doc.wikimedia.org/generated-data-platform/aqs/analytics-api/.

ORES API documentation can be found here: https://www.mediawiki.org/wiki/API:Info

The ORES github can be found here: https://github.com/wikimedia/ores/tree/master

Abiding by the ORES [code of conduct](https://www.mediawiki.org/wiki/Code_of_Conduct), this project does its
best to follow ORES' principles and avoid the unacceptable behaviors listed in the code of conduct.


## Data Sources, Intermediary Files, and Data Outputs
This project uses the two files, provided by Dr. David W. McDonald from
University of Washington's Department of Human Centered Design & Engineering, `politicians_by_country_AUG.2024.csv` and
`population_by_country_AUG.2024.csv` as the initial source data to determine which politicians to pull page data on and
the populations of countries / regions.

The project specifically uses the `name` column to pull page info, from which the revision id is used to pull ORES info.
In the instance of an invalid page, the page is skipped and no page information nor ORES data is collected.

The notebook generates two intermediary files which are used to generate the final datasets.
`politician_page_data.csv` is a collection of all the page data for all the politicians listed in the source data.
`articlequality.csv` is a collection of the ORES data for each page in politician page data.

These intermediary files are used to assemble the two final data sets `wp_countries-no_match.txt` and
`wp_politicians_by_country.csv`. The first being a list of countries / regions that had no matching data between
the two source files. The latter being a csv which holds data on article title, quality and its associated region, country,
and population.

## Data Schema
This section is dedicated to mapping out the schema of the intermediary and final dataset used in the
analysis.

All intermediary data and datasets in .csv come in the form of "dataframe-like" format.
`politician_page_data.csv` has the columns:
- pageid
- ns
- title
- contentmodel
- pagelanguage
- pagelanguagehtmlcode
- pagelanguagedir
- touched
- lastrevid
- length
- watchers
- talkid
- fullurl
- editurl
- canonicalurl

`articlequalty.csv` has the columns:
- revid
- prediction
- probability_B
- probability_C
- probability_FA
- probability_GA
- probability_Start
- probability_Stub

`wp_politicians_by_country.csv` has the columns:
- country
- region
- population
- article_title
- revision_id
- article_quality


## Data Issues
In the original data set, there were some countries that were listed
with a population of 0. These were filtered out of our analysis.

The two data sources `politicians_by_country_AUG.2024.csv` and `population_by_country_AUG.2024.csv`
are merged using the columns `Geography` and `country`. However, these columns do not map one-to-one.
For example, there are politicians labeled as "Korean" which does not map to a single country. For cases,
where the countries do match but differ due to differences in things like dashes or parenthesis, we
manually fix these cases.

These countries that do not match one-to-one can be found in `wp_countries-no_match.txt`.
To counteract this issue we use the fact that the `population_by_country_AUG.2024.csv` is in
a hierarchical order by region. For our data set found in `wp_politicians_by_country.csv`, we implement
a hierarchical merge which merges the country to its lowest hierarchical region.


# Research Implications
In our analysis we created six tables which displayed the top and bottom 10 countries as well as regions
by coverage and quality. High Quality articles were defined to have a classification of "FA" or "GA" in the ORES
data. In our first table, we observe that the top 10 countries with highest `articles_per_capita` are countries
with relatively small populations, with the greatest population being Bhutan with a population of 800,000. Following this,
we observe in our second table that the bottom 10 countries with the lowest `articles_per_capita` are large
countries with substantial populations - in fact the two countries with the lowest `articles_per_capita` are
India and China which are the two countries with the largest populations in the world.

Moving onto the ranking by quality, observe that a mix of large and small nations, but we can see that many of the
smaller countries have only 1 or 2 high quality articles but still have a high `high_quality_articles_per_capita`.
In the bottom 10 countries by quality, we see more smaller countries with many African and Central / South American
countries.

We observe a similar pattern when we group by regions rather than individual countries. This is not
particularly surprising as when we assembled the dataset, we joined the population dataset on the closest region - 
usually meant a direct matching with country name.

At first this pattern of smaller nations taking the top 10 in coverage and larger nations in the bottom 10 of coverage
was surprising. However, if we remind ourselves that our metrics are per-capita, 
we can see that seeing this pattern in our results shouldn't be surprising at all. 
Our primary metrics were per-capita - this would mean that countries with larger populations were going
to be "penalized" whilst countries with a smaller population would get a "boost". 
This is because the total number of articles does not vary as substantially as population.

For example, China, which has a population of around 1.4 billion had only 16 total articles in our set. This number of
articles is comparable to the Marshall Islands at 13 articles with a population of 100,000.

The top 10 and bottom 10 by rankings gets a little bit more interesting. While a per-capita metric is still used,
we observe that the top 10 and bottom 10 are a mix of larger and smaller nations rather than the smaller-only, larger-only
pattern observed in the coverage tables. Despite this, these tables still do show some potential issues with the using
only a per-capita metric as two of the countries listed in the top 10 only have one high quality article.

An issue to note was missingness in our source data. The source data - `politicians_by_country_AUG.2024.csv` is
missing many countries that one may argue are large, significant nations with many politicians that could
change the rankings we observed in our tables. Some examples of missing countries were:
- Canada
- Australia
- Mexico
- Philippines
- Netherlands

Given these results and issues, I theorize that there's some bias in the original data sources.
A bias, that was touched on a bit earlier, is the idea of population size and representation in the data.
As we pointed out earlier, China only has 16 politicians represented out of 1.4 billion people while the 
Marshall Islands had 13 politicians with 100,000 people.

Lastly, a potential bias is that we're working with data from the English Wikipedia. There may be politician pages
that are not present on the English Wikipedia and may only be in another language. This would make our data
bias towards what English speakers would deem "important enough" to create a Wikipedia page about.


1. What biases did you expect to find in the data (before you started working with it), and why?

Prior to working with the data, I anticipated that more developed and well-off
countries would have more articles and higher quality articles compared to less developed countries. 
I thought this because I believed that internet access, and thus access to resources like Wikipedia,
would be less common in countries which are less developed and therefore see less total articles and less quality
articles. Given that we were also working with English Wikipedia, I anticipated there to be fewer articles on
politicians from non-western nations that were not politically relevant in the western world.

2. What (potential) sources of bias did you discover in the course of your data processing and analysis?

In the course of the processing and analysis, I found that there could be some bias when it came to representing
countries with larger populations. I also found that there could be bias in article quality based on the country
as wealther nations may have greater article counts / higher quality articles. Lastly, the data on English Wikipedia
only gives us what English speakers have created and edited which may leave out other important figures who have
pages in non-english pages only.

3. What might your results suggest about (English) Wikipedia as a data source?

These results perhaps suggest that English Wikipedia may be a somewhat biased source. A big concern is
that because the source is English Wikipedia that it will favor countries that use or have large English-speaking
populations which may skew the data towards underrepresenting countries which do not as language barriers may
hinder contribution from these nations.