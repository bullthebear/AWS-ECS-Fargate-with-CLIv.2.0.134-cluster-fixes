import * as awsx from "@pulumi/awsx";

// 1. Create a VPC
const vpc = new awsx.ec2.Vpc("cobber-vpc");

// 2. Create an ECS Cluster
const cluster = new awsx.ecs.Cluster("cobber-cluster", { vpc });

// 3. Create a Load Balancer Listener
const alb = new awsx.elasticloadbalancingv2.ApplicationLoadBalancer("cobber-alb", { vpc });
const listener = alb.createListener("web-listener", { port: 80 });

// 4. Create a Fargate Service
const service = new awsx.ecs.FargateService("nginx-service", {
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

// 5. Export the public URL
export const url = listener.endpoint.hostname;
