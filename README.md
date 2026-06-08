# Lab 2 - CloudFormation + CodePipeline

## Yêu cầu
- AWS CLI đã cấu hình
- S3 bucket: `lab-2-cfn-templates`
- EC2 Key Pair: `keypair-nt548`

## Cấu trúc
```
templates/
├── main.yaml       # Root stack
├── network.yaml    # VPC, Subnet, NAT Gateway
├── security.yaml   # Security Groups
└── compute.yaml    # EC2 instances
```

## Chạy thủ công

```bash
# 1. Tạo S3 bucket
aws s3api create-bucket \
  --bucket lab-2-cfn-templates \
  --region ap-southeast-1 \
  --create-bucket-configuration LocationConstraint=ap-southeast-1

# 2. Upload templates lên S3
aws s3 sync templates/ s3://lab-2-cfn-templates/cfn/ --region ap-southeast-1

# 2. Deploy stack
aws cloudformation deploy \
  --template-file templates/main.yaml \
  --stack-name lab-2 \
  --region ap-southeast-1 \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    ProjectName=lab-2 \
    TemplateBaseUrl=https://s3.amazonaws.com/lab-2-cfn-templates/cfn \
    KeyName=keypair-nt548
```

## Chạy qua Pipeline

```bash
# Deploy pipeline (chỉ chạy 1 lần)
aws cloudformation deploy \
  --template-file pipeline-infra.yaml \
  --stack-name lab2-pipeline-infra \
  --capabilities CAPABILITY_NAMED_IAM \
  --region ap-southeast-1

# Push code lên GitHub để trigger pipeline tự động
git push origin main
```

Pipeline gồm 3 stage: Source (GitHub) → LintAndTest (cfn-lint + taskcat) → Deploy

## Xóa hạ tầng

```bash
aws cloudformation delete-stack --stack-name lab-2 --region ap-southeast-1
```