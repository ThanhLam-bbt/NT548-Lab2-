# NT548.P21_Lab2_Bai2: Tự động hóa CI/CD với AWS CodeBuild & CodePipeline

---

Dự án này trình bày cách xây dựng quy trình **tích hợp và triển khai liên tục (CI/CD)** tự động trên AWS, sử dụng:

- **AWS CloudFormation** để quản lý hạ tầng dưới dạng mã (IaC).
- **AWS CodeBuild** để tự động kiểm thử các template CloudFormation với `cfn-lint` và `taskcat`.
- **AWS CodePipeline** để triển khai ứng dụng tự động từ mã nguồn.

---

## Chuẩn bị

Để bắt đầu, bạn cần:

- **Tài khoản AWS** với đầy đủ quyền để tạo và quản lý các dịch vụ như CloudFormation, S3, IAM, CodeBuild, CodePipeline.
- **AWS CLI** đã được cài đặt và cấu hình (`aws configure`) với quyền truy cập phù hợp.
- Các **IAM role** cần thiết đã được tạo sẵn.

---

## Triển khai

### a. Thiết lập CodeBuild để kiểm thử CloudFormation

Kiểm thử tự động các template CloudFormation bằng cách tích hợp `cfn-lint` và `taskcat` vào CodeBuild.

1.  **Tải lên template CloudFormation lên S3:**

    ```bash
    aws s3 mb s3://nt548-group09-cloudformation-new # Tạo bucket nếu chưa có
    aws s3 cp modules/ec2.yaml s3://nt548-group09-cloudformation-new
    aws s3 cp modules/nat-gateway.yaml s3://nt548-group09-cloudformation-new
    aws s3 cp modules/route-tables.yaml s3://nt548-group09-cloudformation-new
    aws s3 cp modules/security-group.yaml s3://nt548-group09-cloudformation-new
    aws s3 cp modules/vpc.yaml s3://nt548-group09-cloudformation-new
    aws s3 cp modules/root.yaml s3://nt548-group09-cloudformation-new
    ```

2.  **Tạo dự án CodeBuild:**

    ```bash
    aws codebuild create-project \
      --name nt548-group09-lab02-validation \
      --source type=GITHUB,location=[https://github.com/](https://github.com/)<Tên user Github>/NT548.P21_Lab2_2.git,buildspec=buildspec.yml \
      --source-version main \
      --environment type=LINUX_CONTAINER,computeType=BUILD_GENERAL1_SMALL,image=aws/codebuild/standard:7.0 \
      --service-role arn:aws:iam::<Mã tài khoản AWS của bạn>:role/CodeBuildCloudFormationRole \
      --artifacts type=NO_ARTIFACTS
    ```

    _Lưu ý: Thay thế `<Tên user Github>` và `<Mã tài khoản AWS của bạn>` bằng thông tin của bạn. Đảm bảo file `buildspec.yml` đã được cấu hình chính xác._

---

### b. Xây dựng CodePipeline tự động hóa triển khai

Thiết lập một quy trình CI/CD hoàn chỉnh với CodePipeline để tự động build và deploy ứng dụng từ mã nguồn.

1.  **Triển khai IAM Roles:**

    ```bash
    aws cloudformation deploy \
      --template-file "modules/iam-roles.yaml" \
      --stack-name "NT548-Lab2-B-IAM-Roles" \
      --capabilities CAPABILITY_NAMED_IAM
    ```

2.  **Triển khai CodePipeline:**

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
        ArtifactBucket=nt548--cloudformation-new \
        PipelineRoleArn=arn:aws:iam::<Mã tài khoản AWS>:role/NT548-CodePipelineRole \
        CloudFormationRoleArn=arn:aws:iam::<Mã tài khoản AWS>:role/NT548-CloudFormationExecutionRole
    ```

    _Lưu ý: Thay thế các placeholder bằng thông tin thực tế của bạn. Đảm bảo các IAM Role đã tồn tại._

---
