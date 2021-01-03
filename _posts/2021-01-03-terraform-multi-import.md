---
title: Terraform Multi Import
layout: post
image: assets/images/blog/Terraform.jpg
---

Hope all of you know the terraform is an IaC(Infrastructure as a code) provider. The beauty of terraform not only supports multiple cloud infrastructure, we can also use its application configuration and CICD tool perspective.

One of our requirements is to migrate all artifactory repository into terrafrom state file. We all know the terraform has an import option to import the existing configuration into state file. 

But the terraform import only imports a single resource at a time. 

>Warning: Terraform expects that each remote object it is managing will be bound to only one resource address, which is normally guaranteed by Terraform itself having created all objects. If you import existing objects into Terraform, be careful to import each remote object to only one Terraform resource address. If you import the same object multiple times, Terraform may exhibit unwanted behavior. For more information on this assumption

We have n number of repository in the artifact. So doing individual resource import bore and time-consuming.

Used simple bash script to import all repository into state file. Let see the method

Get the list of repository names using the API with curl

```awk
curl -u testing:Testing@12345 -X GET "https://terraformtest.jfrog.io/artifactory/api/repositories" | grep key | awk '{gsub(/"/, "", $3); print "set", $3}' | awk -F'[, ]' '{print $2}' >> list_of_repo.txt
```

Here the script reads the text file and import to state file line by line.

```awk
input="list_of_repo.txt"

while IFS= read -r line
do
  cat << EOF >> main.tf 
	resource "artifactory_local_repository" "$line" {

	}
EOF

terraform import artifactory_local_repository.$line $line
  done < "$input"
```
