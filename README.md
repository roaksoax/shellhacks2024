# Initial Setup (On Ubuntu)
```
sudo apt-get install docker.io git
sudo snap install grype —classic
sudo usermod -aG docker $USER
```

# Docker Image Training
The goal of this exercise is to build and compare a small python application using the Upstream Python image vs the Chainguard Python Image.

## Build an image with hundreds of vulnerabilities
```
# Clone the repository
git clone https://github.com/cjohannsen81/training

# Access the folder
cd training/not-inky

# Build your image
docker build . -t not-inky

# Run your image
docker run --rm not-inky

# Scan your image for CVES
grype docker.io/library/not-inky
```

## Build and image with the Chainguard Image

```
# Build a Chainguard based image:
cd ../inky

# Check the FROM line in the Dockerfile:
cat Dockerfile

# Build the image:
docker build . -t inky

# Run the Chainguard based image:
docker run --rm inky

# Scan the upstream image:
grype docker.io/library/inky 
```

# Building a Chainguard Package (with Melange)

## Setup your development environment (after the initial setup)
```
sudo apt-get install build-essential bubblewrap
sudo chmod u+s /usr/bin/bwrap
sudo snap install go —classic
go install chainguard.dev/melange@latest
export PATH=$PATH:/home/ubuntu/go/bin
```

## Clone the wolfi package repository from github
```
git clone https://github.com/wolfi-dev/os.git
cd os/
```

## Make your package (APK)
```
# This will build your package and make it available under packages/ folder
make package/bash
```

# Building a Chainguard Package (with APKO)

## Prepare your environment
```
# Install apko
go install chainguard.dev/apko@latest

# Install k3d
wget -q -O - https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash

# Install Terraform https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli
# and put it under /usr/local/bin
mv terraform /usr/local/bin

# Clone the image repository
git clone https://github.com/chainguard-images/images.git

# Do configuration
make init
terraform init
```

You will also need to create so it uses a local environment
# copy/paste this
```
cat <<EOF > main_override.tf
provider "imagetest" {
  harnesses = {
    k3s = {
      registries = {
        "\${element(split("/", var.target_repository), 0)}" = {
          mirror = { endpoints = ["http://host.docker.internal:5005"] }
        }
      }
    }
  }
}
EOF
```

# Use the local registry for building and testing images
````
# Setup variables to it doesn't try to publish the image
export TF_VAR_target_repository=k3d-k3d.localhost:5005
# Specify the architecture you are building against
export TF_VAR_archs='["x86_64"]' # x86_64 (amd64) and aarch64 (arm64)
# Run this
make k3d
````

# Build your image
```
make image/bash
```

# References
You can find more information in:
 * wolfi-dev/os: https://github.com/wolfi-dev/os
 * chainguard-images/images: https://github.com/chainguard-images/images
 * How to build images using APKO: https://edu.chainguard.dev/open-source/build-tools/apko/getting-started-with-apko/
 * How to build packages using Melange: https://edu.chainguard.dev/open-source/build-tools/melange/getting-started-with-melange/
 * Example AI Applications with Pytorch: https://edu.chainguard.dev/chainguard/chainguard-images/getting-started/pytorch/
 * What if I already have an app? How can I migrate my app to Chainguard Images: https://edu.chainguard.dev/chainguard/migration/porting-apps-to-chainguard/
