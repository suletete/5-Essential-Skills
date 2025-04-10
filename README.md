
### **Shell Script: `aws_deploy.sh`**

```bash
#!/bin/bash

# -- 1. Load AWS Credentials from Environment Variables
AWS_ACCESS_KEY=$AWS_ACCESS_KEY
AWS_SECRET_KEY=$AWS_SECRET_KEY
AWS_REGION=$AWS_REGION

# -- 2. Validate AWS Credentials
if [ -z "$AWS_ACCESS_KEY" ] || [ -z "$AWS_SECRET_KEY" ]; then
  echo "AWS credentials are not set in environment variables."
  exit 1
fi

# -- 3. Declare Functions for EC2 and S3 Operations

# Function to create EC2 instance
create_ec2_instance() {
  local instance_type=$1
  echo "Creating EC2 instance of type $instance_type..."

  instance_id=$(aws ec2 run-instances \
    --image-id "ami-0c55b159cbfafe1f0" \
    --instance-type "$instance_type" \
    --count 1 \
    --region "$AWS_REGION" \
    --key-name "MyKeyPair" \
    --security-group-ids "sg-0123456789abcdef0" \
    --subnet-id "subnet-0bb1c79de3EXAMPLE" \
    --output text --query 'Instances[0].InstanceId')

  if [ $? -eq 0 ]; then
    echo "EC2 instance $instance_id created successfully!"
    EC2_INSTANCE_IDS+=("$instance_id") # Add to array
  else
    echo "Error creating EC2 instance."
    exit 1
  fi
}

# Function to create S3 bucket
create_s3_bucket() {
  local bucket_name=$1
  echo "Creating S3 bucket named $bucket_name..."

  aws s3 mb "s3://$bucket_name" --region "$AWS_REGION"
  if [ $? -eq 0 ]; then
    echo "S3 bucket $bucket_name created successfully!"
    S3_BUCKETS+=("$bucket_name") # Add to array
  else
    echo "Error creating S3 bucket."
    exit 1
  fi
}

# -- 4. Command Line Argument Parsing

# Default values
EC2_INSTANCE_TYPE="t2.micro"
S3_BUCKET_NAME="my-s3-bucket"

# Parse command-line arguments
while getopts "t:b:" opt; do
  case $opt in
    t) EC2_INSTANCE_TYPE=$OPTARG ;;
    b) S3_BUCKET_NAME=$OPTARG ;;
    *) echo "Usage: $0 [-t EC2_INSTANCE_TYPE] [-b S3_BUCKET_NAME]"; exit 1 ;;
  esac
done

# -- 5. Main Script Execution

# Array to store EC2 instance IDs
EC2_INSTANCE_IDS=()

# Array to store S3 bucket names
S3_BUCKETS=()

# Create EC2 instance and S3 bucket
create_ec2_instance "$EC2_INSTANCE_TYPE"
create_s3_bucket "$S3_BUCKET_NAME"

# -- 6. Display Results

echo "Created EC2 Instances: ${EC2_INSTANCE_IDS[@]}"
echo "Created S3 Buckets: ${S3_BUCKETS[@]}"

# -- 7. Error Handling for any unhandled exceptions
trap 'echo "An error occurred during the script execution."; exit 1' ERR

```

### **Explanation of the Script:**

1. **Environment Variables:**
   - The script expects AWS credentials and region to be set in environment variables. These should be configured before running the script (e.g., in `.bashrc` or `.bash_profile`).
   
   ```bash
   export AWS_ACCESS_KEY="your_aws_access_key"
   export AWS_SECRET_KEY="your_aws_secret_key"
   export AWS_REGION="us-east-1"
   ```

2. **Functions:**
   - `create_ec2_instance()`: This function automates the creation of an EC2 instance with a specified instance type.
   - `create_s3_bucket()`: This function creates an S3 bucket with the given name.

3. **Arrays:**
   - `EC2_INSTANCE_IDS` stores the instance IDs of the EC2 instances created.
   - `S3_BUCKETS` stores the names of the S3 buckets created.

4. **Command Line Arguments:**
   - `-t` allows the user to specify the EC2 instance type (e.g., `t2.micro`).
   - `-b` allows the user to specify the S3 bucket name.
   
   Example usage:
   ```bash
   ./aws_deploy.sh -t t2.medium -b my-bucket-name
   ```

5. **Error Handling:**
   - The script uses the `trap` command to catch any unhandled errors and output a message before exiting.

---

### **Steps to Run the Script:**

1. **Set Environment Variables:**
   Make sure your AWS credentials and region are set as environment variables, as shown earlier.

2. **Make the Script Executable:**
   ```bash
   chmod +x aws_deploy.sh
   ```

3. **Run the Script:**
   You can now run the script with command-line arguments for customizing the EC2 instance type or S3 bucket name. For example:
   ```bash
   ./aws_deploy.sh -t t2.large -b my-custom-s3-bucket
   ```

   This will create a `t2.large` EC2 instance and a bucket named `my-custom-s3-bucket`.

---

### **Final Notes:**

- Ensure you have AWS CLI configured and authenticated with the necessary IAM roles to create EC2 instances and S3 buckets.
- Replace placeholder values in the script (like `ami-0c55b159cbfafe1f0` for the AMI, and `sg-0123456789abcdef0` for the security group) with values appropriate for your setup.

