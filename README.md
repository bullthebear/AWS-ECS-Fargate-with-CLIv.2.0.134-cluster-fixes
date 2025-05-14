import * as awsx from "@pulumi/awsx";
import * as pulumi from "@pulumi/pulumi";

// 1. VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc", {});

// 2. ECS Cluster
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// 3. Application Load Balancer Listener with health check
const listener = new awsx.lb.ApplicationListener("cobber-listener", {
    vpc,
    port: 80,
    targetGroup: {
        port: 80,
        healthCheck: {
            path: "/index.html", // You can change this if needed
            interval: 30,
            timeout: 5,
            healthyThreshold: 2,
            unhealthyThreshold: 2,
        },
    },
});

// 4. Cobber Fargate Service
const cobberService = new awsx.ecs.FargateService("cobber-service", {
    cluster,
    assignPublicIp: true, 
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

// 5. Export the ALB hostname
export const cobberUrl = listener.endpoint.hostname;
