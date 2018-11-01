+++
# Date this page was created.
date = "2018-10-29"

# Project title.
title = "Global Terrorism Analysis"

# Project summary to display on homepage.
summary = "R Dashboard to analyze Global Terrorism Data for 1970-2017"

# Optional image to display on homepage (relative to `static/img/` folder).
#image_preview = "Most_variant_features-2.png"

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ["data-analysis", "R", "rstats", "shiny"]

# Optional external URL for project (replaces project detail page).
#external_link = ""

# Does the project detail page use math formatting?
math = true

# Optional featured image (relative to `static/img/` folder).
#[header]
#image = "Most_variant_features-2.png"
#caption = "Features with high variance showing distinction between cancer and healthy"

+++

## **Overview**

The Global Terrorism Database (GTD) is an open-source database of terrorist events around the world from 1970 through 2017. It is maintained and updated by the GTD staff based at START headquarters at the University of Maryland. It includes systematic data on domestic as well as international terrorist incidents that have occurred during this time period and includes more than 180,000 cases.

## **Data background**

GTD includes data for incidents based on publicly available, unclassified source materials. These include media articles and electronic news archives, existing data sets, secondary source materials such as books and journals, and legal documents. The initial data were collected by various agencies as can be seen below.

| Agency                                                | Start Date | End Date            |
|-------------------------------------------------------|------------|---------------------|
| Pinkerton Global Intelligence Service (PGIS)          | 1970/1/1   | 1997/12/31          |
| Center for Terrorism and Intelligence Studies (CETIS) | 1998/1/1   | 2008/3/31           |
| Study of Violent Groups (ISVG)                        | 2008/1/4   | 2011/10/31          |
| Study of Violent Groups (ISVG)                        | 1970/1/1   | 2017/12/31(Ongoing) |

The complete data set was integrated by the GTD staff to maintain consistency, across all phases of data collection, in terms of the definitions and the methodology. To know more about current data collection methodology, head to the [GTD codebook](https://www.start.umd.edu/gtd/downloads/Codebook.pdf).

As an **important note**, the data for 1993 is missing from the database. 

## **When is an incident defined as a terrorist attack?**

As per GTD 'The GTD defines a terrorist attack as the threatened or actual use of illegal force and violence by a non-state actor to attain a political, economic, religious, or social goal through fear, coercion, or intimidation. In practice this means in order to consider an incident for inclusion in the GTD, all three of the following attributes must be present:

1. The incident must be intentional – the result of a conscious calculation on the part of a perpetrator.

2. The incident must entail some level of violence or immediate threat of violence -including property violence, as well as violence against people.

3. The perpetrators of the incidents must be sub-national actors. The database does not include acts of state terrorism. 

More details can be seen under **GTD Definition of Terrorism and Inclusion Criteria** in the [GTD codebook](https://www.start.umd.edu/gtd/downloads/Codebook.pdf).

## **Dashboard**

<iframe width="1200" height="1000" scrolling="yes" frameborder="no" src="http://krohitm.shinyapps.io/gtd_app/"> </iframe>

## **Variables and filters used in the dashboard**

Here is an explanation of how to use the filters above:

**Page 1:**

1. **Logistic of Attack:** This is a common filter that gets applied to the whole page. It indicates whether the nationality of the attacker was different from the country where the attack took place.

The next three filters are only for maps on page 1. The three filters work collectively.

2. **Years of attacks:** You can select a range of years to visualize the metrics on the maps for those years.
3. **Weapon used for attack:** You can select a particular weapon to visualize the metrics for attacks where that weapon type was used.
4. **Main Target:** You can select a target type to visualize the metrics for attacks where the selected type was the major target.

The next 2 filters are for the trends in the 2nd row on page 1. The two filters work collectively.

5. **Country where attack occured:** You can select a country to see the trends over the years.
6. **Terrorist group responsible:** You can select a terrorist group to see the trends over the years.