# Geocoder
Django REST API for PostGIS TIGER/Line geocoder

Setup steps:

1. Follow steps on https://experimentalcraft.wordpress.com/2017/11/01/how-to-make-a-postgis-tiger-geocoder-in-less-than-5-days/, roughly:
  - On your server, install Postgres
  - On your server, install Postgis
  - On your server, create a database to handle geocoding data (using pgadmin or however you like to deal with Postgres)
  - Within that database, enable postgis extensions:
    - CREATE EXTENSION postgis;
    - CREATE EXTENSION fuzzystrmatch;
    - CREATE EXTENSION postgis_tiger_geocoder;
    - CREATE EXTENSION address_standardizer;
  - Use functions built into postgis to generate a scrpt that will download TIGER gis data and install it into the geocoding database (note that the below code is for Windows command line):
    - psql -U postgres -c "SELECT Loader_Generate_Nation_Script('windows')" -d geocoder -tA > C:/gisdata/nation_script_load.bat
    - From the command line, run nation_script_load.bat
    - In the geocoding database, verify you have the data: 
      - SELECT count(*) FROM tiger_data.county_all;
  - Use simmilar functions to install state data:
    - psql -U postgres -c "SELECT Loader_Generate_Script(ARRAY['CA', 'NV'], 'windows')" -d geocoder -tA > C:/gisdata/ca_nv_script_load.bat
    - From the command line, run ca_nv_script_load.bat
  - Update tables -- not sure what this step is for, to be honest -- maybe building indices?
    - SELECT install_missing_indexes();
    - vacuum analyze verbose tiger.addr;
    - vacuum analyze verbose tiger.edges;
    - vacuum analyze verbose tiger.faces;
    - vacuum analyze verbose tiger.featnames;
    - vacuum analyze verbose tiger.place;
    - vacuum analyze verbose tiger.cousub;
    - vacuum analyze verbose tiger.county;
    - vacuum analyze verbose tiger.state;
    - vacuum analyze verbose tiger.zip_lookup_base;
    - vacuum analyze verbose tiger.zip_state;
    - vacuum analyze verbose tiger.zip_state_loc;
  - At this point, you should be able to geocode within the database:

SELECT g.rating, ST_AsText(ST_SnapToGrid(g.geomout,0.00001)) As wktlonlat,
(addy).address As stno, (addy).streetname As street,
(addy).streettypeabbrev As styp, (addy).location As city, (addy).stateabbrev As st,(addy).zip
FROM geocode('424 3rd St, Davis, CA 95616',1) As g;


2. Next, install django:  
  - https://docs.djangoproject.com/en/3.0/topics/install/
  
3. Finally, add the files in this repo as a django project and run the server


(I'm sure I've forgotten steps -- let me know what doesn't work!)
