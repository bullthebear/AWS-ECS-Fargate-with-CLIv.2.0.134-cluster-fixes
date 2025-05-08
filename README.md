import * as awsx from "@pulumi/awsx";
import * as ecs from "@pulumi/awsx/ecs";
import * as lb from "@pulumi/awsx/lb";

// 1. Create a VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc");

// 2. Create an ECS Cluster
const cluster = new ecs.Cluster("cobber-cluster", { vpc });

// 3. Create an ALB Listener
const listener = new lb.ApplicationListener("web-listener", {
    vpc,
    port: 80,
});

// 4. Create a Fargate Service
const service = new ecs.FargateService("nginx-service", {
    cluster,
    taskDefinitionArgs: {
        container: {
            name: "nginx",
            image: "nginx",
            cpu: 256,
            memory: 512,
            portMappings: [listener],
        },
    },
    desiredCount: 1,
});

// 5. Export public URL
export const url = listener.endpoint.hostname;
