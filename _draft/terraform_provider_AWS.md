Terraform provider AWS
##### [AWS provider guide page](https://registry.terraform.io/providers/hashicorp/aws/latest/docs)

- ### redshift

  cluster 생성  
  cluster?

  #### aws_redshift_cluster
  ```json
  // defalut
  resource "aws_redshift_cluster" "brep-redshift" {
    cluster_identifier = "tf-redshift-cluster" 
    database_name      = "mydb"  
    master_username    = "foo"
    master_password    = "Mustbe8characters" 
    node_type          = "dc1.large"
    cluster_type       = "single-node"
    number_of_nodes           = 1
    cluster_subnet_group_name = "subnet_group_id"

  }
  ```
  - Argument
    - cluster_identifier(Required):  cluster 식별자(name)
    - database_name(Optional): database name. 설정하지 않을 경우 dev    
    - master_username(Required unless a snapshot_identifier is provided): DB instance master name
    - master_password(Required unless a snapshot_identifier is provided): DB instance master PW. 8자이상, 대문자, 소문자, 숫자
        <details><summary>master password는 AWS KMS를 이용하여 암호화</summary>

        ```json
        data "aws_kms_secrets" "example" {
          secret {
            name    = "master_password"
            payload = "AQ*******Sc="
          }
        }
      
        resource "aws_redshift_cluster" "example" {
          # ... other configuration ...
          master_password = data.aws_kms_secrets.example.plaintext["master_password"]
        }
        ```

        [aws kms encryption 방법](https://ryanpark.dev/419fe316-5049-4c03-bc50-87582855faa1)  
        [aws kms 개념 설명](https://bluese05.tistory.com/71)
        </details>

    - node_type(Required): 프로비저닝 node type(DC2(large/8xlarge), RA3(4xlarge/16xlarge))
    - cluster_type(Optional): single-node or multi-node.
    - number_of_nodes(Optional): multi-node일 경우 node의 갯수 선택 1~32
    - cluster_subnet_group_name(Optional): 생성한 vpc를 사용하기 위해서는 클러스터 subnet group이 구성 되어 있어야 한다.

  - depend on
    * cluster subnet group
    * iternet gateway

  #### aws_redshift_subnet_group
  ```json
  // subnet group
  resource "aws_vpc" "foo" {
  ...
  }

  resource "aws_subnet" "foo" {
  ...
  }

  resource "aws_subnet" "bar" {
  ...
  }

  resource "aws_redshift_subnet_group" "brep-redshift-subnet-group" {
    name       = "foo"
    subnet_ids = [aws_subnet.foo.id, aws_subnet.bar.id]
  }
  ```

  - depend on  
    * vpc
    * subnet



- ### s3
#### aws_s3_bucket
```json
resource "aws_s3_bucket" "b" {
  bucket = "my-tf-test-bucket"
  acl    = "private"

  tags = {
    Name        = "My bucket"
    Environment = "Dev"
  }
}
```
  - Argument
    - bucket(Optional): 지정하지 않을경우 유니크한 이름을 자동 생성
    - acl(Optional): Canned ACL 값들을 설정
      <details><summary>valid value</summary>
        소유자는 FULL_CONTROL.  

        |canned ACL|적용대상|권한|
        |----------|-------|----|
        |private|버킷과 객체|소유자외 다른 누구도 엑세스 권한이 없음|
        |public-read|버킷과 객체|Alluser 그룹은 READ 엑세스 권한|   
        |public-read-write|버킷과 객체|Alluser 그룹은 READ, WRITE 엑세스 권한|
        |authenticated-read|버킷과 객체|AuthenticatedUsers 그룹은 READ 엑세스 권한을 가짐
        |aws-exec-read|버킷과 객체|EC2는 S3에서 READ 엑세스 권한을 가짐. (AMI GET용도)|
        |log-delivery-write|버킷|LogDelivery 그룹은 버킷에 대해 WRITE, READ_ACP 권한을 가짐|
      </details>
      
    - grant(Optional): Canned ACL이외의 ACL권한을 설정 할 수 있다.
    - lifecycle_rule(Optional): 객체의 lifecycle을 정할 수 있다. *expiration*으로 만료일을 지정하거나 *transition*으로 s3 Glacier등의 다른 storage class로 이전할 수 있다. versioning을 사용한다면 *noncurrent_version_transition*, *noncurrent_version_expiration*옵션을 사용하여 구버전의 이전 및 삭제를 지연할 수 있다. 

#### aws_s3_bucket_public_access_block
특정 bucket에 대한 public access를 차단 설정
```json
resource "aws_s3_bucket_public_access_block" "example" {
  bucket = aws_s3_bucket.b.id

  block_public_acls   = true
  block_public_policy = true
  ignore_public_acls = true
  restrict_public_buckets = true
}
```


