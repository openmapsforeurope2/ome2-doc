![Logo](docs/images/logo_ome2.png)

Under construction

The OME2 project is co-funded by the European Union. It is being delivered by a consortium comprising: 
* EuroGeographics, the not-for-profit membership association for Europe’s National Mapping, Cadastral and Land Registration Authorities;
* National Geographic Institute, Belgium;
* National Institute of Geographic and Forest Information, France;
* Hellenic Cadastre;
* General Directorate for the Cadastre, Spain;
* Cadastre, Land Registry and Mapping Agency, The Netherlands. 
 
OME2 is developing a new production process and technical specification for free-to-use, edge-matched data under a single open licence. Authoritative 1:10 000 scale data for 10 countries will be delivered via the user interface built by the award-winning Open Maps For Europe Project. OME2 is also enhancing the five existing Open Maps For Europe datasets, including the pilot Open Cadastral Map. By the end of the year, it will offer coverage for 10 countries to provide a foundation for future pan-European high-value datasets. 

This github project is used to host the source code of the tools which constitute the new production process for a high-value large-scale pan-European database:
<img src="docs/images/Process_nice_EN.png" width="500" height="auto">

The first step of the production process consists in collecting open datasets covering 3 themes (Administrative units, Transport network and Hydrography) from NMCAs (National Mapping and Cartographic Agencies) members of EuroGeographics. In a second step, a number of tools are applied to the data to transform it to the OME2 large-scale data model and ensure consistent edge-matching along international boundaries.

After these production steps, the data is integrated to a central pan-European database, hosted on a Cloud server, which can then be released on the [Open Maps for Europe portal](https://www.mapsforeurope.org/datasets/hvlsp)

An update process is also being put into place in order to update a country's data when a new version of their national dataset becomes available.
