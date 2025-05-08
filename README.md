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
export const url = service.loadBalancer.hostname;
