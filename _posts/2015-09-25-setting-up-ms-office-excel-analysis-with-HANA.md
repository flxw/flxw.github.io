---
layout: post
title: Connecting MS Office Excel Analysis with HANA
titlecolor: white-text
covercolor: blue darken-1
---

Here at my SAP internship, I want to connect Analysis for Excel with
SAP HANA. Unfortunately, this fairly simple procedure is the ultimate example for
bad documentation, and I want to note down my approach here for getting it to work.

The following is for Analysis for Excel versions **2.0 and greater** ([see why](http://scn.sap.com/docs/DOC-63785))
  - Make sure Analysis for Excel is up to date. An obvious step but you don't want any version conflicts after having it set up.
  - Install the *HANA Client for Excel* from [here](https://support.sap.com/software/installations.html) (Click *Search for Software*)
  - Grant your database user the required privileges for accessing HANA via Excel through HTTP ([see why](http://scn.sap.com/community/businessobjects-analysis-ms-office/blog))
    - Import the delivery unit *AHCO_INA_SERVICE.tgz* from the server
    - Add the *INA_USER* role to your user under the *granted roles* section
    - Add the object privileges *_SYS_BI*, *_SYS_BIC* and *_SYS_RT* to your user and enable the *SELECT* checkbox for those
  - Open Analysis for Excel ([see images](http://scn.sap.com/docs/DOC-63784))
    - Click the *Analysis* tab
    - Hit the *Insert Data Source* and *Select Data Source...*
    - Right-click inside the selection menu, and *Create new SAP HANA connection...*
    - Enter your credentials and the selection menu for views should pop up
