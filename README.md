# REA Challenge - Simple Sinatra app

## How to use this stack
  1. Obtain an AWS account
  1. Choose a region, and ensure the default VPC configuration exists (including a configured subnet, internet gateway, DHCP, route table). AWS usually provides this automatically out of the box.
  1. Create an EC2 key pair (suggested name: `sinatra`)
  1. Create a CloudFormation stack using `sinatra.yaml`. If alternate name as used when creating key pair above, then make sure to update the `KeyNameParam` parameter with that name.
  1. Check stack outputs for the `SinatraURL`

## Design Decisions
  1. OS: Amazon Linux - Very lean OS to run your application on, no unnecessary services running out of the box.
  1. Web Server: Rack - Due to lacking in depth knowledge of Ruby application hosting, decided to use the built in rack provided server
  1. App run as unprivileged user: Rack running as a standard user 'sinatra', prevents application vulnerabilities from exposing privileged access to your host
  1. No host level firewall configured, for simplicity only security group has been used to prevent unintended access to the host.

## Limitations / Areas for improvement
  1. Uses default VPC / subnets etc in your AWS account. Ideally, define all the required VPC, subnets, DHCP, NACLs, route tables,  Internet Gateways in CloudFormation.
  1. Questionable scalability of the "rack" server. I'm not deeply familiar with Ruby stacks, but this built in web server is likely only scalable to DEV workloads. Ideally, replace with Apache/Passenger? or some other scalable server.
  1. Logs are not collected and stored persistently. Ideally, configure logs to be pushed periodically, and upon shutdown to an S3 bucket for analysis/reference later.
  1. In order to push out an updated version of the app, need to alter the cloudformation template to force a rebuild of the EC2 instance. One way to address this, is bake an AMI using packer, triggered by CI/CD system.
  1. If the host dies, the website goes down. A good way to fix this, without using multiple hosts, is to setup an launch configuration and autoscaling group with desired host count of 1, with an ELB placed in front of the group. If the only host in the group is detected as being unreachable, a new host will be spun up just in time to replace it.
  1. No pretty DNS address: Obviously an IP address is a hard URL to remember, for your user's sake, setting up a DNS CNAME pointing to an ELB, or DNS A record pointing to an EIP attached to the host would be desirable.
  1. No common vulnerability scanning: using AWS's WAF functionality, or tools like mod_secure, you could prevent simple web app vulnerabilities from being exploited, such as SQL injection.
  1. All requests server by the app server: Depending on number of users, and nature of the content being served, a CDN such as CloudFront may be considered to take some load off of serving static content for your site.
  1. Host level firewall not configured: it is good practice to configure a host level firewall in case of misconfiguration of the security group.
  1. Host OS patching: No automated host OS patching has been configured. Triggering a rebuild of the EC2 instance will however result in the latest OS patches and App code being deployed.
  1. Wasted VM resources: This app is tiny, and is running in the smallest EC2 instance, but that's still overkill. A good way to combat this waste would be to package the app into a docker container and run it on a container cluster where only the minimum resources required would be consumed, and shared with other workloads.
