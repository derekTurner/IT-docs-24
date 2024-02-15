# Nitric

Nitric is an open source framework for the deployment of applications to the cloud.

Rather than developing an app with a fixed resource such as a MongoDB database, the user specifies the general resourses, such as a noSQL database.  Nitric provides these resources in a local development environment.  When cloud deployment is needed any of the supported cloud services AWS, Azure and Google Cloud SErvices can be selected.  The resources of the cloud service will be matched to the requirements, so for example the application might use DynamoDB or CosmosDB according to the choice of provider.

There are a number of examples using nitric with nodejs on the [nitric website](https://nitric.io/docs).  Will nitric work well in a docker development environment?



