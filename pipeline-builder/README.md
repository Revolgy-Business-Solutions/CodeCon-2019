#Docker image used as a pipeline builder

Since we use some additional tools apart from docker engine in our pipeline, we need to prepare our own building image.

We will be pushing our pipeline images via docker to Google Container Repository. For that to work we install to our building image google-cloud-sdk. Furthermore this allows us install kubectl cli as component of google SDK, we used that for creating namespaces for new feature branches.

The second tool we need to have in our building image is Helm. We are deploying our application with prepared helm chart.
