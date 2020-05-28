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
And **attach** the policy to user.<br/>

And make sure your AWS config is at `~/.aws/config`
Example credentials config file:
```shell
[default]
aws_access_key_id=XXXXXXXX
aws_secret_access_key=XXXXXXX
```
Replace the id with the user credentials

### 3. Generation of certificates
```shell
sudo certbot certonly \
--dns-route53 \
--dns-route53-propagation-seconds 5 \
-d this.is.an.example.com
```
Then 4 files will be generated:<br/>
`privkey.pem`  : the private key for your certificate.<br/>
`fullchain.pem`: the certificate file used in most server software.<br/>
`chain.pem`    : used for OCSP stapling in Nginx >=1.3.7.<br/>
`cert.pem`     : will break many server configurations, and should not be used without reading further documentation (see link below). <br/>

The valid period of the certificates is **3** months. For renewal purpose, please use the same command as above.<br/>

### 4. Application Configuration
For application to use the certificates, a secret containing tls.crt and tls.key shall be created.<br/>
Taking harbor as an example, while deploying using Bitnami helm chart, overwrite the values for `service.tls.secretName`.

## Prerequisites
1. A TKG Kubernetes cluster built on AWS EC
