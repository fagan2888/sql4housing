## Example Use Case: Chicago

Suppose we would like to build a database for a housing app in the city of Chicago. There are a number of datasets available via open data portals such as [Chicago's open data portal](https://data.cityofchicago.org/), [Cook County's open data portal](https://datacatalog.cookcountyil.gov/), and [HUD's open data portal](https://hudgis-hud.opendata.arcgis.com/).

The Chicago open data portal provides us with geospatial files of [building footprints](https://data.cityofchicago.org/Buildings/Building-Footprints-current-/hz9b-7nh8) and [neighborhood boundaries](https://data.cityofchicago.org/Facilities-Geographic-Boundaries/Boundaries-Neighborhoods/bbvz-uum9), along with Socrata APIs for building [permits](https://data.cityofchicago.org/Buildings/Building-Permits/ydr8-5enu/data) and [violations](https://data.cityofchicago.org/Buildings/Building-Violations/22u3-xenr/data), a [building code scofflaw list](https://data.cityofchicago.org/Buildings/Building-Code-Scofflaw-List-Map/hgat-td99), and a [problem landlord list](https://data.cityofchicago.org/Buildings/Problem-Landlord-List-Map/dip3-ud6z). We may also be interested in gathering geospatial files on [parcel maps](https://datacatalog.cookcountyil.gov/GIS-Maps/Historical-ccgisdata-Parcels-2016/a33b-b59u), [street midlines](https://datacatalog.cookcountyil.gov/GIS-Maps/Historical-ccgisdata-Street-Midlines-2015/73aw-3v3w), and [address points](https://datacatalog.cookcountyil.gov/GIS-Maps/Historical-ccgisdata-Address-Points-for-Area-13-20/6y64-fiuv) provided by Cook County. If our app aims to provide information on federally funded housing, we may wish to incorporate [Low-Income Housing Tax Credit (LIHTC) properties](http://hudgis-hud.opendata.arcgis.com/datasets/low-income-housing-tax-credit-properties), [HUD assisted](http://hudgis-hud.opendata.arcgis.com/datasets/multifamily-properties-assisted) and [HUD insured](http://hudgis-hud.opendata.arcgis.com/datasets/hud-insured-multifamily-properties/data) multifamily properties, and [public housing developments](http://hudgis-hud.opendata.arcgis.com/datasets/public-housing-developments). Suppose we are also interested in HUD's [physical inspection score data](https://www.hud.gov/program_offices/housing/mfh/rems/remsinspecscores/remsphysinspscores) as available via an Excel file download. We have then filtered the dataset for inspections in Chicago, IL and saved the file locally.

We may also be interested in key demographic characteristics of each census tract within the city of Chicago. Currently, we can load any table from the [5-year American Community Survey](https://www.census.gov/programs-surveys/acs/technical-documentation/table-and-geography-changes/2017/5-year.html) or [Decennial 2010 Census](https://www.census.gov/programs-surveys/decennial-census/decade/2010/about-2010.html). This option is built on top of the [cenpy package](https://github.com/cenpy-devs/cenpy).

#### NOTE:
Key values HUD_TABLES must be GeoService URLs. This URL can be obtained in the 'APIs' dropdown located on the upper right hand corner of each dataset's page. To pre-filter the dataset, click on the 'Data' tab as shown below.

![Alt text](images/data_tab.png "Public Housing Buildings")

After applying filters to the dataset, the 'APIs' dropdown will include a 'Filtered Dataset' option. In the example below, I filtered the "Public Housing Developments" dataset to only include developments within Chicago, IL. I then copied the GeoService URL under 'Filtered Dataset' into bulk_load.py.

![Alt text](images/hud_example.png "Public Housing Buildings")

#### NOTE:
[American FactFinder](https://factfinder.census.gov/faces/nav/jsf/pages/index.xhtml) is the Census Bureau's site for navigating datasets and available tables. [Census Reporter](https://censusreporter.org/) is an excellent alternative. Currently, the provided example of [bulk_load.yaml](https://github.com/sunlightpolicy/Housing_Data/blob/master/sql4housing/bulk_load.yaml) includes the variables B25105, B25064, and B25104 from the 5-year ACS 2017 and 2016. These variables represent Median Monthly Housing Costs, Median Gross Rent, and Monthly Housing Costs, respectively.

### Bulk Loading

We can load all of the above datasets at once by running this command in the same folder where bulk_load.yaml is stored.
    
    sql4housing bulk_load

In order to do so, we must fill out the file [bulk_load.yaml](https://github.com/sunlightpolicy/Housing_Data/blob/master/sql4housing/bulk_load.yaml) according to the documented format. All datasets listed above are filled out within the current example of bulk_load.yaml. Each relation to be loaded into your database allows for an optional table name. Without a table name, the relation will default to a sanitized version of the path or hyperlink.


### Example Query

After executing the command,

    sql4housing bulk_load

using the current example, we should be able to run

    psql chi_property_data
    \d

and see that the database is populated with a list of relations.

With a PostGIS extension, we will be able to answer questions such as "What is the number of public housing developments in Chicago by neighborhood?" with a geospatial query like the following:

    SELECT pri_neigh, COUNT(*)
    FROM public_housing_developments
    JOIN neighborhood_boundaries
    ON ST_INTERSECTS(public_housing_developments.geometry, neighborhood_boundaries.geometry)
    GROUP BY pri_neigh
    ORDER BY COUNT(*) DESC;

In doing so, we learn that Bronzeville has the highest number at 17, followed by River North, United Center, and Kenwood. This query is possible because we have loaded a dataset of [public housing developments](https://hudgis-hud.opendata.arcgis.com/datasets/public-housing-developments) from HUD and [neighborhood boundaries](https://data.cityofchicago.org/Facilities-Geographic-Boundaries/Boundaries-Neighborhoods/bbvz-uum9) from Chicago's open data portal.

![Alt text](images/query_output.png "Public Housing Buildings")

#### NOTE:

While SQLAlchemy can support a number of databases, housing data often entails geospatial data types. Sql4housing urrently only supports geospatial data types on Postgres with a [PostGIS extension](https://postgis.net/install/). Without PostGIS, datasets will still upload but geospatial columns will be skipped.

### Individual Inserts

We can also insert datasets into a database individually. Suppose we want to include [affordable rental housing developments supported by the City of Chicago](https://data.cityofchicago.org/Community-Economic-Development/Affordable-Rental-Housing-Developments/s6ha-ppgi) in 'postgres:///chi_property_data'. Since this dataset is supported by the Socrata Open Data API (SODA), we should be able to note the dataset ID (in this case, 's6ha-ppgi') and enter the command

    sql4housing socrata data.cityofchicago.org s6ha-ppgi --d="postgres:///chi_property_data"

Detailed instructions on individual inserts are available within [cli.py](https://github.com/sunlightpolicy/sql4housing/blob/master/sql4housing/cli.py).

