# Federated Identity Management on scientific collaborations

[CRISP](http://www.crisp-fp7.eu/) (Cluster of Research Infrastructures for Synergies in Physics) is a partnership which builds collaborations and creates long-term synergies between research infrastructures on the ESFRI (European Strategy Forum on Research Infrastructure). It is built on several work programmes that are splitted in WP (Work Packages). I am involved in IT & DATA MANAGEMENT WORK PROGRAMME, specifically in WP16 "IT & DM: Common User Identity System".

The objetives of my new project are to develop and deploy a pan-European system for unique identification (Authentication and authorisation
infrastructure: AAI) of users at the infrastructures of the participating RIs EuroFEL (PSI), ESRF, ESS, FAIR
(GSI), ILL, and XFEL for the management of local and remote access to facilities, experiments, data, and IT
resources.

The plan of this project is also output documentation. [This is how I plan to proceed.](fedid/plan.html)

__Identity Management System__ is the set of tools needed to manage the identity of people. (_Wikipedia source_)An IdM (Identity Management) system comprises: 

1. Establishment of the identity.
2. Describes the identity.
3. Follows identity activity.
4. Destroys the identity.

When we talk about __Federated IdM__ the we are providing a mean for partner services to agree on and establish a common, shared name identifier to refer to the user in order to share information about the user across the organizational boundaries.

##Motivation

The main goal of look into Federated access for scientific communities is to create a unique identification (AAI) system. An authentication and authorisation mechanism common to all users of different RIs can greatly simplify the access to distributed resources.
Also considering that the user community in Science is growing, it would be benefitial to keep the number of accounts to a minimum, this is to keep only one account for each user in the Federation. With this approach, a user would have less barriers when accessing remote resources. Also, federated identity management is good for security because it reduces the amount of accounts needed for a user (identity providers have the attention of attackers). 
There are a set of common needs from those communities (gathered in the [1st workshop on Federated Identity Management in Scientific Collaborations](https://indico.cern.ch/conferenceTimeTable.py?confId=129364#20110610.detailed)):

1. Need for single-sign on access.
2. Ease of use for part-time users. 
3. Controlling access to the resources. 
4. Support for homeless researchers. 
5. Interoperability accross national boundaries. 
6. Trust is needed with accreditation. IGTF is already in use by many of the communities.
7. Traceability.
8. Enable federated access from grids of High Performance Computing, as well as High Troughput Computing and new Distributed computing resources, such as Clouds, Supercomputing networks and Desktops grids. 


## State-of-the-art technologies and tools

### Workshops and meetings attended

As a first step I will research the state-of-the-art technologies and tools currently used or being adopted by other scientific collaborations. I attended the [1st workshop on Federated Identity Management in Scientific Collaborations](https://indico.cern.ch/conferenceTimeTable.py?confId=129364#20110610.detailed) and the [2nd workshop on Federated Identity Management in Scientific Collaborations](http://indico.cern.ch/conferenceTimeTable.py?confId=157486#20111103), where the main user-requirements, existing technologies and vision on this subject were presented. 

This is more or less the set of scenarios described and the tools that each of them is using currently.

| Organisation | #users      | Main concern                                  | Tool used  |
|:------------:|:------------|:----------------------------------------------|:-----------|
| PSI          |  ~ 10000    |Very big datasets, that want to access remotely. Very short lived users. Homeless users.| Umbrella (shibboleth + SAML)|
| CLARIN     |hundreds |Single domain with identity managed by home institute. Many diverse and distributed data sets with complex relations in between them.|eduGAIN (Shibboleth + SAML)|
|DARIAH |hundreds| VRE-Integration of homeless users and Job-Submission (e.g. Globus, gLite) through Shibboleth, based on Robot Certificates and Short-Lived-Credentials. Researching eduGAIN and integration with user-centered mechanisms like openID| Shibboleth|
| WLCG    | ~5900|IGFT is the approach to federated IdM. This solution does not scale. In the Grid it is difficult to identify the original source of compromise. x509 needs to be supported (security token service)|x509|
|ESGF (earth system grid fed.) |~5000 |Already many technologies existing.Single sign on with OpenID and MyProxy.Attribute propagation with SAML and OpenID. Authorisation with SAML. | openID + SAML|
| OSG (open science grid) | |    They want to integrate together InCommon and IGTF| SAML + Shibboleth  + x509|

### Tools
   
There are a number of technologies implied in the current scenarios of federated identity:

 - [SAML](fedid/saml.html)
 - [Discovery problem](http://sites.google.com/site/publisherinterfacestudy/home/the-discovery-problem) is a well known problem in the federated identity topic.


## [AuthN and AuthZ technologies](authNauthZ.html)

This is a review for the state-of-the-art major authN and authZ technologies. 

# Use case for FAIR

The actual use case for FAIR would be a computing center, located at GSI, and at LOEWE. Eventually, satellite Universities or scientific organizations, geographycally close, may take part of the infrastructure.
A user may send a job from GSI facility and use data from LOEWE file systems. 
So mainly the federated access would be towards file systems and remote computing from long-term users.

Current technologies federating access to resources mostly rely on applications running on an HTTP agent. Such is the case of Shibboleth, running on 
top of [SAML](fedid/saml.html). These technologies have been used widely in todays outsourced resources, like Google mail accounts, and generally in 
Internet2 applications. These solutions allow an organization providing services over the web to consume the identities of its research partners, 
business partners, customers or other affiliates. The users would use the same credential as they are using for accessing their local resources. 

We are more interested in federating access to resources that do not rely in a HTTP agent. However, today there is no good solution to take this 
federated identity management to applications beyond the web such as e-mail, instant messaging, file sharing, or remote computing. Moonshot is the 
effort to federate access tu such resources. 

These are the main applications we are interested in federate: 

 - Remote computing.
 - File systems, such as ext3, NFS, Lustre.

Nowadays, Moonshot does not have a solution for Lustre. There have been some efforts to solve this problem inside the project [Tera 
Grid](https://www.xsede.org/tg-archives). In this context, the project [Data Capacitor](http://pti.iu.edu/dc) from the University of Indiana, have 
developed a large data storage system based on use of the [Lustre](http://www.lustre.org/docs/whitepaper.pdf) file system.

[Lustre WAN](http://www.teragridforum.org/mediawiki/index.php?title=Lustre-WAN) is a project inside Data Capacity that enables Lustre file systems, 
as it has been demonstrated to have exceptional capabilities, to be able to move data to inside Tera Grid from outside institutions. For this it was 
necessary to ensure that access to files and permissions could be managed properly. This solution is explained in [Enabling Lustre WAN for 
Production Use on the TeraGrid](a19-simms.pdf)


