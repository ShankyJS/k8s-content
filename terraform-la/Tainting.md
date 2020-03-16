# Tainting and Untainting Resources

## Terraform commands

- taint: Manually mark a resource for recreation untaint: Manually unmark a resource as tainted

### Tainting a resource

````tf
terraform taint [NAME]
````

### Untainting a resource

````tf
terraform untaint [NAME]
````

### Set up the environment

```cd terraform/basics```
Redeploy the Ghost image:

```terraform apply```

### Taint the Ghost blog resource

````tf
terraform taint docker_container.container_id
````

See what will be changed:

````tf
terraform plan
````

Remove the taint on the Ghost blog resource:

````tf
terraform untaint docker_container.container_id
````

Verity that the Ghost blog resource is untainted:

````tf
terraform plan
````

Updating Resources
Let's edit main.tf and change the image to ghost:alpine.

Open main.tf:

main.tf contents:

````tf
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:alpine"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "ghost_blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}
````

Validate changes made to main.tf.

See what changes will be applied:

````tf
terraform plan
````

Apply image changes:

````terraform apply````

List the Docker containers:

````docker container ls````

See what image Ghost is using:

````tf
docker image ls | grep [IMAGE]
````
