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
