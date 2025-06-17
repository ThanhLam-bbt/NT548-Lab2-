# NT548.P21_Lab2_Bai2

# Triển khai tự động hóa kiểm thử & triển khai hạ tầng AWS với CodeBuild, CodePipeline

Repository này chứa các file YAML dùng để:

- Triển khai hạ tầng AWS bằng AWS CloudFormation.
- Tự động hóa kiểm thử (lint, validate) template với AWS CodeBuild tích hợp cfn-lint, taskcat.
- Tự động hóa build & deploy ứng dụng với AWS CodePipeline.

---

## Yêu cầu

- Tài khoản AWS với quyền đủ để tạo/điều chỉnh các dịch vụ (CloudFormation, S3, IAM, CodeBuild, CodePipeline, v.v.).
- AWS CLI đã cài đặt trên máy cá nhân.
- Đã cấu hình AWS CLI với quyền thích hợp (`aws configure`).
- Đã tạo các role cần thiết trong IAM.

---

## a. Triển khai AWS CodeBuild tích hợp cfn-lint và taskcat

### 1. Tải các file CloudFormation template lên S3

Tạo bucket mới (nếu chưa có):

```bash
aws s3 mb s3://nt548-group10-cloudformation-new
```

Upload các file module lên S3 bằng các dòng lệnh bên dưới:

```bash
aws s3 cp modules/ec2.yaml s3://nt548-group10-cloudformation-new
aws s3 cp modules/nat-gateway.yaml s3://nt548-group10-cloudformation-new
aws s3 cp modules/route-tables.yaml s3://nt548-group10-cloudformation-new
aws s3 cp modules/security-group.yaml s3://nt548-group10-cloudformation-new
aws s3 cp modules/vpc.yaml s3://nt548-group10-cloudformation-new
aws s3 cp modules/root.yaml s3://nt548-group10-cloudformation-new
```

### 2. Tạo AWS CodeBuild Project

Tạo project CodeBuild để tự động kiểm thử cfn-lint và taskcat thông qua file buildspec.yml:

```bash
aws codebuild create-project \
  --name nt548-group10-lab02-validation \
  --source type=GITHUB,location=https://github.com/<Tên user Github>/NT548.P21_Lab2_YeuCau2.git,buildspec=buildspec.yml \
  --source-version main \
  --environment type=LINUX_CONTAINER,computeType=BUILD_GENERAL1_SMALL,image=aws/codebuild/standard:7.0 \
  --service-role arn:aws:iam::<Mã tài khoản AWS của bạn>:role/CodeBuildCloudFormationRole \
  --artifacts type=NO_ARTIFACTS
```

**Lưu ý:**

- Hãy thay thế `<Tên user Github>`, `<Mã tài khoản AWS của bạn>` bằng thông tin tương ứng của bạn.
- Đảm bảo file `buildspec.yml` đã có trong repo và cấu hình cfn-lint, taskcat phù hợp.

---

## b. Triển khai AWS CodePipeline tự động hóa build & deploy từ mã nguồn

### 1. Triển khai IAM roles

```bash
aws cloudformation deploy \
  --template-file "modules/iam-roles.yaml" \
  --stack-name "NT548-Lab2-B-IAM-Roles" \
  --capabilities CAPABILITY_NAMED_IAM
```

### 2. Triển khai CodePipeline

```bash
aws cloudformation deploy \
  --template-file "modules/codepipeline.yaml" \
  --stack-name "NT548-Lab2-B-CodePipeline" \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameter-overrides \
    GitHubOwner=<Tên user Github> \
    GitHubRepo=<Tên Repo github chứa mã nguồn> \
    GitHubBranch=main \
    GitHubOAuthToken=<Personal Access Token Github> \
    ArtifactBucket=nt548-group10-cloudformation-new \
    PipelineRoleArn=arn:aws:iam::<Mã tài khoản AWS>:role/NT548-CodePipelineRole \
    CloudFormationRoleArn=arn:aws:iam::<Mã tài khoản AWS>:role/NT548-CloudFormationExecutionRole
```

**Lưu ý:**

- Hãy thay thế `<Tên user Github>`, `<Personal Access Token Github>`, `<Tên Repo github chứa mã nguồn> ` , `<Mã tài khoản AWS>` bằng thông tin thực tế của bạn.
- Đảm bảo các Role đã tồn tại trong IAM.

---

## Tài liệu tham khảo

- [AWS CodeBuild Documentation](https://docs.aws.amazon.com/codebuild/latest/userguide/welcome.html)
- [AWS CodePipeline Documentation](https://docs.aws.amazon.com/codepipeline/latest/userguide/welcome.html)
- [AWS CloudFormation Documentation](https://docs.aws.amazon.com/cloudformation/index.html)
- [cfn-lint](https://github.com/aws-cloudformation/cfn-lint)
- [taskcat](https://github.com/aws-quickstart/taskcat)
