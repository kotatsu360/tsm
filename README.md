# tsm
Tsm is a tool to connect to multiple EC2 instances simultaneously via [AWS Systems Manager Session Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager.html).
(Tmux Session Manager)

![demo](https://user-images.githubusercontent.com/10104981/190131715-2d308797-78b7-4123-9b39-88b7aed5861d.gif)

## Key Features
* **Simultaneous Connections**: Connect to EC2 instances using their Name tags and instance IDs. Since Session Manager is used for the connection, it is not necessary to open the security group of the EC2 instances.
* **Simultaneous Operation**: The same key input is performed for all instances connected with the tmux function.
* **List Retrieval**: This is useful for searching for a connection destination.

## Getting Started

Here are the steps you need to take to get tsm working.

### Prerequisites

The following software and AWS IAM permissions are required to use tsm.

* software: aws cli 2.x / tmux
* IAM permissions: ec2  / ssm

> And on the EC2 instance side, the following two conditions must also be met.
>
> 1. an instance profile including AmazonSSMManagedInstanceCore must be attached
> 2. ssm-agent must be installed.
>
> These are requirements for using Session Manager, not tsm, so they are not explained here.
> For more information, please click [Step 1: Complete Session Manager prerequisites - AWS Systems Manager](https://docs.aws.amazon.com/systems-manager/latest/userguide/session-manager-prerequisites.html).

#### Installing prerequisites software

```bash
brew install awscli tmux
```

#### IAM Permissions
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": [
                "ec2:DescribeInstances",
                "ssm:StartSession",
                "kms:GenerateDataKey"
            ],
            "Resource": [
                "*"
            ],
            "Effect": "Allow"
        },
        {
            "Action": [
                "ssm:TerminateSession",
                "ssm:ResumeSession"
            ],
            "Resource": [
                "arn:aws:ssm:*:*:session/${aws:username}-*"
            ],
            "Effect": "Allow"
        }
    ]
}
```

I put the AWS CloudFormation template for creating above IAM policy and a trial user in the [iam](./iam/) directory. If you are a CloudFormation user, I hope this will be helpful.

### Installing tsm

```bash
git clone https://github.com/kotatsu360/tsm.git
sudo cp tsm/tsm /path/to/your/bin/ # ex. /usr/local/bin/
```

## Usage
```bash
$ tsm --help
tsm is a tool to connect to multiple EC2 instances simultaneously via SSM.

Usage:
    tsm [options] [target ...]

Options:
    -r --region STRING   The region where the EC2 instance(s) is located. If omitted, use config/env settings in the aws cli.
    -p --profile STRING  Name of the aws cli profile to use for the connection. If omitted, use default profile or env settings in the aws cli.
    -o --options STRING  If you want to pass options other than region and profile to "aws ssm start-session", use this. For example, -o "--reason hoge"
    -l --list            Print a list of Name tags and instance ids for EC2 instances in the location specified by region and profile.
    -h --help            Print this
```

Print a list of Name tags for EC2 instances in the location specified by region and profile.
```bash
$ tsm -l
i-076a3c31dbf0af073	MyServer
i-0b55c94fbcedc0863	MyServer
i-0325abf52cb86ad50	MyServer2
```


Connect to EC2 instances
```bash
# If there are multiple instances matching the Name, connect to all of them
$ tsm MyServer

# It can also be specified with an instance ID.
$ tsm MyServer i-0325abf52cb86ad50

# Of course, multiple Names can be specified.
$ tsm MyServer MyServer2

```

After disconnecting from the EC2 instances, the tmux session can be terminated with Ctrl-d.

## Note
I have not tested it on Linux. I think it will work on a relatively new bash/zsh running environment, but if it doesn't, please let me know. I would like to solve this problem as much as possible.

## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details

## Acknowledgments
The following blog post was used as a reference for implementation.

* [tmuxで複数サーバの同時オペレーション – NaviPlus Engineers' Blog](http://tech.naviplus.co.jp/2014/01/09/tmuxで複数サーバの同時オペレーション/)
* [【最終完全版】 bash/zsh 用オプション解析テンプレート (getopts→shift)](https://zenn.dev/takakiriy/articles/e65780261dd5e3)


