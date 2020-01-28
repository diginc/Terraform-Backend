# Terraform S3 and Dynamo Backend Bootstrap

Terraform on AWS runs well with an S3 & Dynamo backend for statefulness locking. This repo will show how to create said S3 and dymo repo with terraform and than switch to using it in the same terraform project. 

This process also migrate the old/initial backend (local) to the new backend (s3) which prevents loosing track of the created resources. That can prove useful should you ever have to manipulate these resources futher in the future, share this infrastructure with other Admins, or integrate CI/CD for terraform.

Out of scope but worth mentioning are prequiresites make sure you have installed `awscli` package and entered Administrator credentials for whatever account you're bootstrapping with `aws configured`.  Then the following commands should work to bootstrap your remote s3 terraform backend.

- `alias tf="docker run -it --rm -v "$HOME/.aws":/root/.aws -v $(pwd):/ws -w /ws hashicorp/terraform:light"`
  - I use this alias instead of the `terraform` command, replace `tf` with `terraform` if you prefer to use that
- `tf init`
- `tf apply --auto-approve`
  - this will create your initial buckets and dynamo with a local backend
  - Take note of the bucket name for usage in the next tf init command
- Next enabled the backend in this stack to test it out
- `mv terraform.tfdisable terraform.tf``
- `tf init --backend=true --backend-config="dynamodb_table=tf-remote-state-lock" --backend-config="bucket=tc-remotestate-xxxxx" --force-copy`
  - __Note:__ Bucket Name random digits (-xxxxx) will be changed to what your initinal apply created
  - `--force-copy` will migrate your local backend to your new s3 remote backend

The output of the second init should have mentioned using the s3 backend and you should be able to confirm a state/lock in the s3 bucket and dynamo if you look in AWS web console.

## Resources:

* https://www.terraform.io/docs/commands/init.html
* https://www.terraform.io/docs/backends/types/s3.html
* Original Article based off of - https://www.techcrumble.net/2021/01/how-to-configure-terraform-aws-backend-with-s3-and-dynamodb-table/
