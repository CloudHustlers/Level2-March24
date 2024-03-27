# GSP751
```cmd
export REGION=
```
```cmd
gcloud config set compute/region $REGION
git clone https://github.com/terraform-google-modules/terraform-google-network
cd terraform-google-network
git checkout tags/v6.0.1 -b v6.0.1
cd examples/simple_project
gcloud config list --format 'value(core.project)'
rm -f variables.tf main.tf
cat > variables.tf <<EOF
variable "project_id" {
  description = "The project ID to host the network in"
  default     = "$DEVSHELL_PROJECT_ID"
}

variable "network_name" {
  description = "The name of the VPC network being created"
  default     = "example-vpc"
}
EOF
cat > main.tf << EOF
module "test-vpc-module" {
  project_id   = var.project_id
  network_name = var.network_name
  source       = "terraform-google-modules/network/google"
  version      = "~> 6.0"
  mtu          = 1460

  subnets = [
    {
      subnet_name   = "subnet-01"
      subnet_ip     = "10.10.10.0/24"
      subnet_region = "$REGION"
    },
    {
      subnet_name           = "subnet-02"
      subnet_ip             = "10.10.20.0/24"
      subnet_region         = "$REGION"
      subnet_private_access = "true"
      subnet_flow_logs      = "true"
    },
    {
      subnet_name               = "subnet-03"
      subnet_ip                 = "10.10.30.0/24"
      subnet_region             = "$REGION"
      subnet_flow_logs          = "true"
      subnet_flow_logs_interval = "INTERVAL_10_MIN"
      subnet_flow_logs_sampling = 0.7
      subnet_flow_logs_metadata = "INCLUDE_ALL_METADATA"
      subnet_flow_logs_filter   = "false"
    }
  ]
}
EOF
terraform init
terraform apply --auto-approve
terraform destroy --auto-approve
cd ~
rm -rd terraform-google-network -f
cd ~
touch main.tf
mkdir -p modules/gcs-static-website-bucket
cd modules/gcs-static-website-bucket
touch website.tf variables.tf outputs.tf
cat > website.tf << EOF
resource "google_storage_bucket" "bucket" {
  name               = var.name
  project            = var.project_id
  location           = var.location
  storage_class      = var.storage_class
  labels             = var.labels
  force_destroy      = var.force_destroy
  uniform_bucket_level_access = true

  versioning {
    enabled = var.versioning
  }

  dynamic "retention_policy" {
    for_each = var.retention_policy == null ? [] : [var.retention_policy]
    content {
      is_locked        = var.retention_policy.is_locked
      retention_period = var.retention_policy.retention_period
    }
  }

  dynamic "encryption" {
    for_each = var.encryption == null ? [] : [var.encryption]
    content {
      default_kms_key_name = var.encryption.default_kms_key_name
    }
  }

  dynamic "lifecycle_rule" {
    for_each = var.lifecycle_rules
    content {
      action {
        type          = lifecycle_rule.value.action.type
        storage_class = lookup(lifecycle_rule.value.action, "storage_class", null)
      }
      condition {
        age                   = lookup(lifecycle_rule.value.condition, "age", null)
        created_before        = lookup(lifecycle_rule.value.condition, "created_before", null)
        with_state            = lookup(lifecycle_rule.value.condition, "with_state", null)
        matches_storage_class = lookup(lifecycle_rule.value.condition, "matches_storage_class", null)
        num_newer_versions    = lookup(lifecycle_rule.value.condition, "num_newer_versions", null)
      }
    }
  }
}
EOF
cat > variables.tf << EOF
variable "name" {
  description = "The name of the bucket."
  type        = string
}

variable "project_id" {
  description = "The ID of the project to create the bucket in."
  type        = string
}

variable "location" {
  description = "The location of the bucket."
  type        = string
}

variable "storage_class" {
  description = "The Storage Class of the new bucket."
  type        = string
  default     = null
}

variable "labels" {
  description = "A set of key/value label pairs to assign to the bucket."
  type        = map(string)
  default     = null
}


variable "bucket_policy_only" {
  description = "Enables Bucket Policy Only access to a bucket."
  type        = bool
  default     = true
}

variable "versioning" {
  description = "While set to true, versioning is fully enabled for this bucket."
  type        = bool
  default     = true
}

variable "force_destroy" {
  description = "When deleting a bucket, this boolean option will delete all contained objects. If false, Terraform will fail to delete buckets which contain objects."
  type        = bool
  default     = true
}

variable "iam_members" {
  description = "The list of IAM members to grant permissions on the bucket."
  type = list(object({
    role   = string
    member = string
  }))
  default = []
}

variable "retention_policy" {
  description = "Configuration of the bucket's data retention policy for how long objects in the bucket should be retained."
  type = object({
    is_locked        = bool
    retention_period = number
  })
  default = null
}

variable "encryption" {
  description = "A Cloud KMS key that will be used to encrypt objects inserted into this bucket"
  type = object({
    default_kms_key_name = string
  })
  default = null
}

variable "lifecycle_rules" {
  description = "The bucket's Lifecycle Rules configuration."
  type = list(object({
    action = any
    condition = any
  }))
  default = []
}
EOF
cat > outputs.tf <<EOF
output "bucket" {
  description = "The created storage bucket"
  value       = google_storage_bucket.bucket
}
EOF
cd ~
cat > main.tf << EOF
module "gcs-static-website-bucket" {
  source = "./modules/gcs-static-website-bucket"

  name       = var.name
  project_id = var.project_id
  location   = "$REGION"

  lifecycle_rules = [{
    action = {
      type = "Delete"
    }
    condition = {
      age        = 365
      with_state = "ANY"
    }
  }]
}
EOF
touch outputs.tf variables.tf
cat > outputs.tf <<EOF
output "bucket-name" {
  description = "Bucket names."
  value       = "module.gcs-static-website-bucket.bucket"
}
EOF
cat > variables.tf <<EOF
variable "project_id" {
  description = "The ID of the project in which to provision resources."
  type        = string
  default     = "$DEVSHELL_PROJECT_ID"

}

variable "name" {
  description = "Name of the buckets to create."
  type        = string
  default     = "$DEVSHELL_PROJECT_ID"
}
EOF
terraform init
terraform apply --auto-approve
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/master/modules/aws-s3-static-website-bucket/www/index.html > index.html
curl https://raw.githubusercontent.com/hashicorp/learn-terraform-modules/blob/master/modules/aws-s3-static-website-bucket/www/error.html > error.html
gsutil cp *.html gs://$DEVSHELL_PROJECT_ID
```
