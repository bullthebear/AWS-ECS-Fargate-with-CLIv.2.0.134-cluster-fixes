import * as awsx from "@pulumi/awsx";

// 1. Create a VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc", {});

// 2. Create an ECS cluster
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// 3. Create a Fargate service running nginx
const service = new awsx.ecs.FargateService("nginx-service", {
    cluster,
    taskDefinitionArgs: {
        containers: {
            nginx: {
                image: "nginx",
                cpu: 256,
                memory: 512,
                portMappings: [{ containerPort: 80 }],
            },
        },
    },
    desiredCount: 1,
});

// 4. Export the public URL safely
export const url = service.listeners.apply(listeners =>
    listeners && listeners.length > 0 ? listeners[0].endpoint.hostname : "pending..."
);
