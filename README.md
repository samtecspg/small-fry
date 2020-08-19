# Big Data for the Small Fry
Companion repository for the talk Big Data for the Small Fry

The Smal Fry talk is about how to walk your way into Big Data systems when your experience (and technology stack) is firmly rooted in the traditional big-SQL world (On site ms-sql etc...)

This repo is a combination of instructions on how to "taste" various enabling technologies and instructions/code-samples for integrating/hardening those technologies for wide scale use in your company so they can be use it to walk into the cloud.

The technologies covered will be about building up to a (small/affordable version of the) Big Data infrastructure laid out in the Netflix blog post https://medium.com/netflix-techblog/notebook-innovation-591ee3221233.  The self-service/self-documenting/incremental nature of that stack spoke to our needs, and we suspect they will speak to yours too.

## The (most fundamental) basics: Notebooks & Runtime
AKA Jupyter(Hub) and (Docker+papermill)Airflow

The elegance of the infrastructure purposed by Netflix is that it can be distilled down to only two moving parts:

1. Notebooks which contain code
1. A way to periodically run those notebooks

Even though it will be an incomplete view, lets start by looking at the two technologies that can enable that distilled view:

1. (Docker) [Jupyter](https://github.com/jupyter/jupyter) - Place to make/test/document ETL style code 
1. (Docker) [papermill](https://github.com/nteract/papermill) - machinery to reproducibility run the code from Jupyter

There are many projects scattered throughout github/etc that provide demo environments for these two apps (and good parts of the netflixs flow), the follow provides a demo that really just contains these two elements, so if you want to understand this infrastructure at its most basic, this would be a good place to start:  https://github.com/lukasvrabel/test_airflow_papermill

## (Still fundamentals) Going multi-user
While the above technology is mildly novel, it would be easier to work with shell scripts and cron (or whatever your preferred ETA engine has been). These need to bring more value (... they will, and more after that and more after that...)

The first step to more value is for them to be multi-user - they get there by being wrapped in larger scale projects, so:

1. (Docker Jupyter) is encapsulated by [JupyterHub](https://github.com/jupyterhub/jupyterhub) - Multi-user Jupyter
2. (Docker papermill) is encapsulated by [Airflow](https://github.com/apache/airflow) - Multi-user scheduling

Both JupyterHub and Airflow have a _lot_ more features to bring-to-the-table, but if you would like to to try the most basic on-prem instance then look a !!this!!mike!still!needs!to!pick!! repo

## (Starting to get real) First touch of the clouds
The previous infrastructure provides an all-in-one on-site ETL system, but thats still not enough value to learn/build/adopt these systems.  They still need to bring more - how can they start moving us into the clouds?

The first step to moving to the clouds is (perhaps surprising) about storage.

Storage is what the cloud can do cheaper and better then anyone can do on-prem.  Any of cloud providers that offer object-storage (the big 3 and most of the first tier smaller players) offer crazy durability and uptime at a jaw-droppingly low price (at least when compared to database storage costs) - using napkin math, we worked out the combination of all the storage we had in databases could be stored in AWS for $50 a month - at those kinds of prices, even if it was just to enable analytics we could easily afford to store 10x - and with those increases in storage all sorts of ML/AI use cases open up.

Get storage live on any of the cloud providers is fairly straight forward.  We have the most history with AWS, so those examples will be the most common in this series.

[[Setting up AWS account and s3 bucket goes here]]
[[Config code for Jupyter to use S3contents]]
[[Config code for JupyterHub to make AWS tempkey for Jupyters]]
[[Improve airflow dag to auto-store runlogs]]
## (Starting to get real) Corporate muli-user - SSO
Part of the reason that JupyterHub and Airflow are compelling platforms to work with is that they both have robust oauth support - with oauth being the (public networks) primary way of creating Single-Sign-On experiences for users.

In our case we internally use microsoft active directory, but also sync our AD to Azure - this is where the magic can happen.  Whichever user-auth you use internally, if you sync up to one of the cloud user directories, those directories will allow you to do oauth/openid SSO for those sync logins (this is how the microsoft "login from your company" office360 apps work, they do oauth/openid requests back to the Azure sync of the companies internal AD).

To enable this systems you need to:

1. Have your user-auth info in one of the cloud services
1. Go to the oauth/openid part of that cloud service to request application keys for JupyterHub and Airflow
1. Add the extra config lines in JupyterHub and Ariflow configs so they uses those keys and oauth/callback URLs

[[Examples of setting up oauth go here - likely link to jupyterhub oauth instructions]]

## (The real deal) Blending SSO and cloud permissions
Now that you have storage and auth linked up, your almost certainly going to want a way to enable/disable Jupyter access to cloud resource (that bill by use) to users.   To do that you need to build user-associated access keys in the JupyterHub environment.

Like most cloud things, this APIs are provider specific - and you may want to mix-and-match (For instance we want auth from azure but access to resources on AWS) - so while I'll provide examples of how we made it work, some work will be required to support your specific need.

Feel free to reach out (or open issues) if you are thinking about implimenting - and if you succeed, I would love a pull request to add your particular solution to the repo.

[[Teraform example to make IAM/group stuff for AWS auth]]
[[JupyterHub code for per-user keys]]
[[Airflow example for job keys]]

## (The Small Fry system) Refining, embracing, extending
If you gotten to this stage - you have a small scale version of everything Netflix claimed you could gain with their infrastructure.   Much of your work will now be to teach how people should adjust their thinking for cloud work.

If you haven't already moved to a data lake, this is like the moment.  Same goes for a data catalog.

This is also a great time to start building supporting libraries to easy connections to things

[[code example for library]]
[[code example for shortcuts]]

## (The universe and beyond) going Kubernetes 
This is the phase my company is currently working on.

The original choice of working with JupyterHub and Airflow was largely driven by their deep support for kuberenets - the basics are already known to work (and work well) - its just the testing and translation of the above into code that will work in the Helm/connector world.

More to come on this section as we progress.


