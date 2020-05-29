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

## Using TKG Extensions Bundle (envoy + contour + cert-manager)
### 1. Prerequisites
- A TKG Kubernetes cluster built on AWS EC
- A valid DNS domain

### 2. Deploy TKG Extensions Bundle (Ingress Control)
Download the extensions bundle from https://www.vmware.com/go/get-tkg and unpack the tar file.

Set the context of kubectl to the Tanzu Kubernetes cluster on which to deploy Contour.
```
kubectl config use-context my-cluster-admin@my-cluster
```
Install cert-manager on the cluster.
```
kubectl apply -f tkg-extensions-v1.1.0/cert-manager/
```
Deploy Contour and Envoy on the cluster.
```
kubectl apply -f tkg-extensions-v1.1.0/ingress/contour/aws/
```
Deploy some test pods and services on the cluster.
```
kubectl apply -f ingress/contour/examples/common/
```
Deploy the Kubernetes ingress resource on the cluster.
```
kubectl apply -f ingress/contour/examples/https-ingress/
```
Get the host name of the Envoy service load balancer.
```
kubectl get service envoy -n tanzu-system-ingress -o jsonpath='{.status.loadBalancer.ingress[0].hostname}'
```
Get the IP address of the Envoy service load balancer.

Replace <ENVOY_SERVICE_LB_HOSTNAME> in the following command with the output of the kubectl get service envoy command that you ran in the preceding step.
```
nslookup <ENVOY_SERVICE_LB_HOSTNAME>
```
Add an /etc/hosts entry to map the IP address of the Envoy service load balancer to foo.bar.com.

Replace <ENVOY_SERVICE_LB_IP> in the following command with the output of the nslookup command that you ran in the preceding step.
```
echo '<ENVOY_SERVICE_LB_IP> foo.bar.com' | sudo tee -a /etc/hosts > /dev/null
```
Verify that the following URLs work by going to the following addresses.
- https://foo.bar.com/foo
- https://foo.bar.com/bar <br/>
You should see output similar to the following:
```
Hello, world!
Version: 1.1.0
Hostname: helloweb-7cd97b9cb8-vmnbj
```

### 3. Configure DNS for Envoy
Retrieve the external address of the load balancer assigned to Contour’s Envoys by your cloud provider:
```
kubectl get svc -n tanzu-system-ingress                                                                                                      
NAME      TYPE           CLUSTER-IP       EXTERNAL-IP                                                                   PORT(S)                      AGE
contour   ClusterIP      10.109.148.227   <none>                                                                        8001/TCP                     9d
envoy     LoadBalancer   10.104.132.243   a1b17f5b46a70473da8f81fc14330142-863409382.ap-southeast-1.elb.amazonaws.com   80:30812/TCP,443:30171/TCP   9d
```
The value of EXTERNAL-IP varies by cloud provider. In our case, AWS gives a long DNS name.

To make it easier to work with the external load balancer, we will add a DNS record to a domain we control that points to this load balancer’s IP address.
On AWS, you specify a CNAME, not an A record, and it would look something like this:
```
host envoy.aws.ronk8s.cf                                                                                                                  
envoy.aws.ronk8s.cf is an alias for a1b17f5b46a70473da8f81fc14330142-863409382.ap-southeast-1.elb.amazonaws.com.
a1b17f5b46a70473da8f81fc14330142-863409382.ap-southeast-1.elb.amazonaws.com has address 13.229.221.89
a1b17f5b46a70473da8f81fc14330142-863409382.ap-southeast-1.elb.amazonaws.com has address 3.0.88.159
```

### 4.
