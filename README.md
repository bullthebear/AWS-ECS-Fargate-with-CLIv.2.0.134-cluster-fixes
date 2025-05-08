import * as aws from "@pulumi/aws";
import * as awsx from "@pulumi/awsx";

// 1. Create a new VPC (with public+private subnets)
const vpc = new awsx.ec2.Vpc("cobber-vpc", {});

// 2. Create an ECS cluster in that VPC
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// 3. Create a Fargate service with a public load balancer
const nginx = new awsx.ecs.FargateService("cobber-nginx", {
    cluster,
    taskDefinitionArgs: {
        container: {
            image: "nginx",
            cpu: 256,
            memory: 512,
            portMappings: [{ containerPort: 80 }],
        },
    },
    desiredCount: 1,
});

// 4. Export the public URL
export const url = nginx.loadBalancer.hostname;


curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -
sudo yum install -y nodejs
*

# 1. Remove old Node.js
sudo yum remove -y nodejs

# 2. Set up NodeSource for Node.js 18
curl -fsSL https://rpm.nodesource.com/setup_18.x | sudo bash -

# 3. Install Node.js 18
sudo yum install -y nodejs

# 4. Confirm version
node -v

import * as awsx from "@pulumi/awsx";

// Create a VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc");

// Create an ECS cluster
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// Create a Fargate service running nginx
const service = new awsx.ecs.FargateService("nginx-service", {
    cluster,
    taskDefinitionArgs: {
        container: {
            name: "nginx",
            image: "nginx",
            cpu: 256,
            memory: 512,
            portMappings: [{ containerPort: 80 }],
        },
    },
    desiredCount: 1,
});

// Export the public load balancer's URL
export const url = service.listener.url;

