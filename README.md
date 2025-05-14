import * as awsx from "@pulumi/awsx";
import * as pulumi from "@pulumi/pulumi";

// 1. VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc", {});

// 2. ECS Cluster
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// 3. Application Load Balancer Listener
const listener = new awsx.lb.ApplicationListener("cobber-listener", {
    vpc,
    port: 80,
});

// 4. Cobber Fargate Service using DockerHub image
const cobberService = new awsx.ecs.FargateService("cobber-service", {
    cluster,
    taskDefinitionArgs: {
        containers: {
            cobber: {
                image: "dundycenic/cobber-nginx:latest", 
                cpu: 256,
                memory: 512,
                portMappings: [listener],
            },
        },
    },
    desiredCount: 1,
});

// 5. Export the ALB URL
export const cobberUrl = listener.endpoint.hostname;
