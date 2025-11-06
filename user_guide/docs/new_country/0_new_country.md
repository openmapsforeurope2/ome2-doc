# Add new country

When data is provided for a new country, here are the main steps to follow:
* Upload the data in a PostGIS database on the OME2 server.
* Convert the data to integrate it in the central database.
* Process international boundaries.
* If neighbouring countries are already included, edge-match the data for each theme successively, starting with Administrative units (AU).
