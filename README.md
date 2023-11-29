# devops.tooling.website.solution
Build a comprehensive devops tooling website solution.

You will implement a tooling website solutio which makes access to DevOps tools within the corporate infrastructure easily accesisble. 

This project implements the following components:

  - Infrastructure: AWS
  - Webserver Linux: Red Hat ENterprise Linux 8
  - Database Server: Ubuntu 20.04 + MySQL
  - Storage Server: Red Hat Enterprise Linux 8 + NFS Server
  - Programming Language
  - Code Repository: GitHub

A 3-tier architecture.
Webservers treat the the storage and databse units as local systems to retrieve data from:
![](https://github.com/naqeebghazi/devops.tooling.website.solution/blob/main/images/1.three-tierArchitecture.png?raw=true)

Storage types:
NFS, Block and Object storage.

How do you know which storage system to use depends on:
  - Data type
  - Format type
  - Data access
  - Who/what will access the data
  - From where will the data be accessed
  - How frequently will the data be accessed

## Implementing a business website using NFS for the backend file

