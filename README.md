import * as awsx from "@pulumi/awsx";

// 1. VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc", {});

// 2. ECS Cluster
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// 3. Application Load Balancer Listener
const listener = new awsx.lb.ApplicationListener("nginx-listener", {
    vpc,
    port: 80,
});

// 4. Fargate Service (binds to listener)
const service = new awsx.ecs.FargateService("nginx-service", {
    cluster,
    taskDefinitionArgs: {
        containers: {
            nginx: {
                image: "nginx",
                cpu: 256,
                memory: 512,
                portMappings: [listener],
            },
        },
    },
    desiredCount: 1,
});

// ✅ 5. Export public ALB hostname
export const url = listener.endpoint.hostname;
