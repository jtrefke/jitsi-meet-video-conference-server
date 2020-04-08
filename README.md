# AWS CloudFormation Template for Jitsi Meet Video Conference Server

This template installs the open-source Jitsi Meet Video Conference Server on 
an Amazon Web Services EC2 instance (fully automated).
Jitsi is a free, simple multi participant video conference 
solution, similar to Google Meet, Google Hangouts, Skype Meet Now, or Zoom.
While there is a reference installation at https://meet.jit.si you may want to
operate your own server for family, friends, neighbors, a company, for
privacy or fun reasons.

This CloudFormation template uses the [Jitsi Meet Video Conference Installer](https://github.com/jtrefke/jitsi-meet-video-conference-installer)
and is mainly meant for disposable, temporary personal use on the Internet or 
for use in private networks as security, scalability, or optimized asset 
delivery has not been a main consideration in the implementation so far.
This is not to say, that you cannot host a large number of participants 
(potentially 100+) depending on the instance size you choose 
(see https://jitsi.org/jitsi-videobridge-performance-evaluation/).

**Click the button below to setup your own Jitsi meet video conferencing server in your AWS region**
<center>

[![Launch CloudFormation Stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png
)](https://console.aws.amazon.com/cloudformation/home#/stacks/new?stackName=jitsi-meet-video-conference&templateURL=https://cf-templates-1pndz72m6jnrt-eu-west-2.s3.eu-west-2.amazonaws.com/jitsi-server.cfn.yaml)

</center>

## Used AWS resources

The server setup focuses on simplicity, ease-of-use and very little AWS 
commitment (both, resource-wise and financially); that means it does not use 
a lot of resources nor does it require the use of other potentially cost-incurring 
AWS services such as Route 53 or ELBs beyond the EC2 instance.

This template will create an EC2 instance based on a Ubuntu 18.04 image. 
Some instance sizes are configurable in the template. 
For on-demand pricing information in your region see: https://aws.amazon.com/ec2/pricing/on-demand/
For suggested spot market prices in your region (if used) see: https://aws.amazon.com/ec2/spot/pricing/

Furthermore the template provides options to schedule the start and shutdown of 
the server to save costs and reduce exposure for security reasons.

The template should be executable in the following AWS regions:

| Americas     | Europe/ME      | Asia Pacific   |
|--------------|----------------|-----------------
| us-east-1    | eu-west-1      | ap-northeast-1 |
| us-east-2    | eu-west-2      | ap-northeast-2 |
| us-west-1    | eu-west-3      | ap-southeast-1 |
| us-west-2    | eu-north-1     | ap-southeast-2 |
| ca-central-1 | eu-central-1   | ap-southeast-3 |
| sa-east-1    | me-south-1     | ap-south-1     |
|              |                | ap-east-1      |

## Tips and tricks

**DNS/SSL**

In order to access your Jitsi meet installation and to be able to provide others
a link to a meeting room, it is advisable to make the instance reachable through
a memorable/shareable name that can be resolved through DNS.
More importantly, a valid DNS record associated with the AWS EC2 server is 
required in order to obtain a trusted SSL certificate through letsencrypt,
which is certainly recommended.

If you have not registered a domain or if you cannot easily update the DNS records
for that domain, a free 
[Dynamic DNS provider](https://en.wikipedia.org/wiki/Dynamic_DNS) may be helpful.
The template allows for specifying a URL that will be requested pre and post-
installation as well as on reboot.
Most dynamic DNS providers allow for updating a DNS record by simply invoking a
URL with a provided password (for example 
https://www.dynu.com/DynamicDNS/IPUpdateClient/Custom).
One thing to keep in mind is the duration within which the DNS records will
be refreshed or for how long they are cached by other DNS servers or the 
operating system (Time To Live (TTL)). 
Depending on the provider or plan this may be configurable or not and take up
to a couple of hours.
To account for this TTL there is an additional wait period that can be specified
in the template for the installation to wait so that the DNS record will 
definitely be updated when needed (for example for the SSL certificate 
verification).

In case you have not registered a domain and don't want to for your server, 
free dynamic DNS providers usually allow you to choose subdomain from a list of 
domains the provider owns. This will enable you to get an SSL certificate issued
and provide you at least with a constant domain name to share.
One provider that supports specifying the TTL for free is for instance 
https://www.dynu.com.

When you already have your own domain and access to the nameserver but are 
unable to update it in a programmatic way, a dynamic DNS provider 
may prove to be helpful as well.
With a `CNAME` record in your registered domain's nameserver with an equally 
short TTL that points to the dynamic DNS name, it is similarly possible to have a constant domain name that refreshes dynamically under your own domain.

**Use of the CloudFormation stack**

The CloudFormation stack is meant to quickly produce re-producible results.
If you don't need use the server, you can just delete the stack which will
delete the server and its associated resources from your account. 
Once you need it again, you can just create a new one again within minutes.

Alternatively, the server can be shut down and started again when needed.
However, this may incur costs for the storage used while turned off and 
currently there is no security update mechanism in place which might lead to an
out-dated system over time (whereas on creation all updates will be installed).

**Creating certificate parameters**

The template includes parameters for existing SSL certificate and key.
In order to provide the value it needs to be gzipped and base64 encoded without
new line characters.
This project includes a helper script `encode-certificate` in the `helpers` 
directory which will transform an existing file into the required format.

**Using the AWS EC2 spot market**

This template contains an option to enter a maximum price to bid for AWS spot 
market servers, i.e. excess server capacities that are up for bidding.
This means, that, only if there are instances on the spot market in your region
and the current bidding price is less than the maximum price provided, a 
server will be started. If the bidding price is too low, though, no server will 
be started until the price lowers or if the price increased, the server will 
be terminated (and re-started once the provided price wins a bid).
Obviously this means, that this concept is only suitable if your service does 
not require 100% reliability/uptime. 
However, theoretically, providing a maximum price that is higher or equal to 
the on-demand price, should always win an instance (if there are any available).

If reliability is not too much of a concern, spot pricing can result in about 
up to 90% in cost savings. For temporary use this option is should be sufficient
and cost-effective.

The prices for each region for each instance size can be found here:
- Spot prices: https://aws.amazon.com/ec2/spot/pricing/
- On-demand prices: https://aws.amazon.com/ec2/pricing/on-demand/

More information about EC2 spot instances can be found here: 
https://aws.amazon.com/ec2/spot/

## Debugging

This template does not define an AWS key pair to access the instance through 
SSH. 
If something needs debugging on the server side, a key pair needs to be added
to the template.
The logs from the install that runs during the cfn init process is stored in
`/var/log/cloud-init-output.log`.
