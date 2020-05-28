# TKG-cert-management

## Manual Approach for Enabling TLS (not necessarily for TKG only)
### 1. Installation of certbot client and dns plugin
You may choose your preferred plugin based on cloud provider. Since my DNS is hosted on AWS, so following is taking AWS as an example.
```shell
brew install certbot
pip3 install certbot-dns-route53
```
### 2. Create respective AWS user and policy
Example AWS policy file: 
```json
{
    "Version": "2012-10-17",
    "Id": "certbot-dns-route53 sample policy",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ListHostedZones",
                "route53:GetChange"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect" : "Allow",
            "Action" : [
                "route53:ChangeResourceRecordSets"
            ],
            "Resource" : [
                "arn:aws:route53:::hostedzone/YOURHOSTEDZONEID"
            ]
        }
    ]
}
```
Only replace the **YOURHOSTEDZONEID** with your DNS Hosted Zone ID.
And attach the policy to user.
And make sure your AWS config is at `~/.aws/config`

## Prerequisites
1. A TKG Kubernetes cluster built on AWS EC2
