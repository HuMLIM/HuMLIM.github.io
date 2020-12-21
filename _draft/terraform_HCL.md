##### Terraform은 HCL언어로 정의하고 클라우드 리소스를 생성하는 도구이다.
##### terraform 0.11버전과 그 이후 버전간에는 문법적, 구조적 차이가 있으므로 0.12버전 이상을 사용 한다.
##### [Terraform 공식 ref](https://www.terraform.io/docs/configuration/index.html)
+ ## **기본 구조**
    ```hcl
    <BLOCK TYPE> "<BLOCK LABEL>" "<BLOCK LABEL>" {
      # Block body
      <IDENTIFIER> = <EXPRESSION> # Argument
    }
    ```

+ ## BLOCK TYPE
    - ### terraform
        ```hcl
        terraform {
          required_providers {
            aws = {
              source  = "hashicorp/aws"
              version = "~> 3.0"
            }
          }
        }
        ```
        terraform의 버전과 provider의 버전이 따로 관리 된다.

    - ### provider
        ```hcl
        provider "aws" {
          access_key = "AWS ACCESS KEY"
          secret_key = "AWS SECRET KEY"
          region = "AWS REGION"
        }
        ```
        access_key와 secret_key는 파일에 hardcording되면 안되므로 다른 방법을 통해 적용

    - ### resource
        ##### 기본 문법
        ```hcl
        resource "resource_type" "resource_name" {
          <ATTR_NAME>="<ATTR_VALUE>"
        }
        ```
        ##### Meta-Argument

        resource_type에 상관없이 모든 곳에서 사용할 수 있는 argument로 *depends_on*, *count*, *for_each*, *provider*, *life_cycle*이 있다.
        - for_each : map이나 set자료형을 순회하면서 키 또는 값을 가져온다. 키 또는 값을 가져올 때는 each.key 또는 each.value를 사용 한다.  
        ✔ map이나 set의 값이 apply 이후에 할당되는 값이면 안된다.
      
        - provider : resource의 provider를 변경 할 수 있다.

    - ### variable
        ```hcl
        variable "aws_region" {
          default = "ap-northeast-2"    // default가 없을 경우 사용자에게 입력 받는다.
          type = string
          description = "this is aws region variable"
        }

        provider "aws" {
          region = var.aws_region
        }
        ```
        변수로 설정하여 다른 block에서 사용
         - default - A default value which then makes the variable optional.
         - type - This argument specifies what value types are accepted for the variable.
         - description - This specifies the input variable's documentation.
         - validation - A block to define validation rules, usually in addition to type constraints. (CLI v0.13.0)
         - sensitive - Limits Terraform UI output when the variable is used in configuration. (CLI v0.14.0)  
        
        변수를 사용할 때는 var.\<BLOCK LABEL NAME> 형식으로 사용한다.

    - ### module
        ```hcl
        module "network" {
          source = "./net"
          base_cidr_block = "10.0.0.0/16"
        }
        ```
        모듈화를 통한 이미 만들어진 인프라 재사용
        source는 필수 속성이며, path정보를 가지고 있다. local path뿐만 아니라 다양한 정보가 올 수 있다.
        
    - ### data
        ```hcl
        data "aws_ami" "example" {
          owners = ["account ID"]
          filter {
              name   = "state"
              values = ["available"]
          }
          most_recent = true
        }
        ```
        aws 리소스의 정보를 가져올 때 사용 한다.  
        
        > 예시의 aws_ami는 owners는 필수 정보이다.
        > filter 속성을 통해 결과 범위를 줄일 수 있으며,
        > 다수의 결과가 return될 경우 most_recent 속성을 통해 최근 사용한 AMI로 선택할 수 있다.

    - ### output
        ```hcl
        ```
+ ## 
