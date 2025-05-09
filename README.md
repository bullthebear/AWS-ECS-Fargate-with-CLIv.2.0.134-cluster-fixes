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

// 4. Cobber Fargate Service (replaces nginx)
const cobberService = new awsx.ecs.FargateService("cobber-service", {
    cluster,
    taskDefinitionArgs: {
        containers: {
            cobber: {
                image: "registry.git.cenic.org/inf-p1/cobber:latest",
                cpu: 256,
                memory: 512,
                portMappings: [listener],
                repositoryCredentials: {
                    // ⛳️ Replace the below with the actual secret ARN from Secrets Manager
                    credentialsParameter: pulumi.interpolate`arn:aws:secretsmanager:us-west-2:<your-account-id>:secret:gitlab-cobber-registry-XXXX`,
                },
            },
        },
    },
    desiredCount: 1,
});

// 5. Export the load balancer URL
export const cobberUrl = listener.endpoint.hostname;
