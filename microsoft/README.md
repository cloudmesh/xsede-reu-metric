#Retrieval/Analysis for Microsoft Academic Knowledge API

##Introduction

The intent of this script is to retrieve the entire Microsoft Knowledge API database, then place it into MongoDB for analysis against XSEDE publications.

##Dependencies

This project requires both MongoDB and ElasticSearch to be installed and running (together, so MongoDB needs a replica set plugged into Elastic).

##Usage

This project is a work in progress.

###Monthly Data Retrieval

Currently, there is a wrapper command to retrieve Microsoft information by a range of years:

    ak_api.py range <start> <end>
    
So to retrieve 2005-2017:

    ak_api.py range 2005 2017
    
###Extended Metadata Retrieval

Extended metadata retrieval is trickier. Microsoft's servers are picky about the amount of records retrieved, so many calls must be made. A monthly run of this would require many thousands of calls.

    ak_api.py extended <year>
    
###Get top-level Fields for Each Publication

To retrieve the top-level field for each publications use:

    ak_api.py fields fields [--count=<count> --start=<start>]
    
Count is the number per loop to run and start allows you to begin at a specified index. This command requires both FieldsOfStudy.txt and FieldOfStudyHierarchy.txt to be present.

###Fuzzy Matching for Publications

To run a fuzzy matching comparison using ElasticSearch a filename with a list of publication titles:

    ak_api.py match <filename>

This creates the file <filename>_ms.txt (the file extension is removed for naming purposes)

For example:

    ak_api.py match xsede.txt 
    
Will create the file xsede_ms.txt with the rows formatted as "XD_ID | MS_ID".

###XSEDE collection creation

The XSEDE collection can be created after pubs_report_ms.txt and pubs_xup_ms.txt have been created using <i>match</i>.

    ak_api.py xsede
    
This will create the xsede collection and gather citation and peer citation information for each xsede publication

###Bridges comparison

This is a wrapper to quickly compare bridges pubs (from bridges.txt) with citation information:

    ak_api.py bridges

##Database

The database currently has 5 collections:

- publications
- extended
- fields
- journals
- xsede

###publications

This collection contains all information for each publication, retrieved directly from Microsoft.

###extended

This collection stores all of the extended metadata retrieved from Microsoft for each publication (referenced by Id)

###fields

This collection stores the top-level fields of each publication in the <i>publications</i> collection. The information is gathered from FieldsOfStudy.txt and FieldsOfStudyHierarchy.txt.

###journals

This collection is an aggregation of the <i>publications</i> and <i>extended</i> collections. It stores citation count for each journal (by journal Id) for each volume and issue.

###xsede
    
This collection is created using the text files for pubs_report and pubs_xup (XID | MSID). The collection then uses the information from the <i>journals</i> collection to get the peer citation count for each xsede publication.