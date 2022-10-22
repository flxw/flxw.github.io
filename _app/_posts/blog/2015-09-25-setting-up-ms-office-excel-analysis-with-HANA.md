---
layout: post
title: Excel Analysis with HANA data
category: blog
---

Here at my SAP internship, I want to connect Analysis for Excel with
SAP HANA. Unfortunately, this fairly simple procedure is the ultimate example for
bad documentation, and I want to note down my approach here for getting it to work.

The following is for Analysis for Excel versions **2.0 and greater** ([why](http://scn.sap.com/docs/DOC-63785))

<ol>
<li>Make sure Analysis for Excel is up to date. An obvious step but you don't want any version conflicts after having it set up.</li>
<li>Install the <em>HANA Client for Excel</em> from <a href="https://support.sap.com/software/installations.html">here</a> (Click <em>Search for Software</em>)</li>
<li>Grant your database user the required privileges for accessing HANA via Excel through HTTP (<a href="http://scn.sap.com/community/businessobjects-analysis-ms-office/blog">why</a>)

<ol>
<li>Import the delivery unit <em>AHCO_INA_SERVICE.tgz</em> from the server</li>
<li>Add the <em>INA_USER</em> role to your user under the <em>granted roles</em> section</li>
<li>Add the object privileges <em>_SYS_BI</em>, <em>_SYS_BIC</em> and <em>_SYS_RT</em> to your user and enable the <em>SELECT</em> checkbox for those</li>
</ol></li>
<li>Open Analysis for Excel (<a href="http://scn.sap.com/docs/DOC-63784">see images</a>)

<ol>
<li>Click the <em>Analysis</em> tab</li>
<li>Hit the <em>Insert Data Source</em> and <em>Select Data Source...</em></li>
<li>Right-click inside the selection menu, and <em>Create new SAP HANA connection...</em></li>
<li>Enter your credentials and the selection menu for views should pop up</li>
</ol></li>
</ol>
