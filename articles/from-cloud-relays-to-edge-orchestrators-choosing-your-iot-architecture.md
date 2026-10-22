# From Cloud Relays to Edge Orchestrators: Choosing Your IoT Architecture

- Date: March 4th, 2026
- Category: Technical
- Authors: Leonel Barragán
- URL: https://www.marvik.ai/blog/from-cloud-relays-to-edge-orchestrators-choosing-your-iot-architecture

---

## Introduction: The IoT Cloud Architecture Dilemma

Deploying IoT devices at scale raises a set of questions that are easy to answer during prototyping but become critical in production:

- How do devices connect to the cloud?

- How do we push updates?

- How do we manage a fleet?

The landscape offers multiple answers to each question, and the choices are not always independent. A direct MQTT client is simple but leaves update management entirely to you. An edge orchestrator like Azure IoT Edge or AWS Greengrass handles connectivity and module deployment but adds resource overhead. Fleet management can be delegated to dedicated platforms, handled through the orchestrator, or implemented as a custom fleet management solution.

This post documents different approaches to these problems, starting with a cloud-agnostic MQTT/AMQP client, then exploring edge orchestrators, and finally covering fleet management options and practical considerations that emerged from real deployments.

## Cloud-Agnostic Architecture with MQTT/AMQP

The first approach prioritized flexibility over features. The goal was to build a cloud agnostic client that could connect to either Azure IoT Hub or AWS IoT Core, switching between them based on configuration alone.

In practice, this client runs between local sensors or applications and the selected cloud service. Its job is simple: collect data, format it according to what the cloud expects, and transmit it securely.

#### Architectural Highlights

- Dynamic Cloud Selection : By utilizing a CLOUD_PROVIDER environment variable, the device can switch its upstream destination between Azure and AWS in real-time without requiring a firmware flash.

- X.509 Identity at the Edge : We prioritized security by implementing X.509 Self-Signed authentication. This replaces static connection strings with unique cryptographic identities.

- Minimal Footprint : Performance benchmarks validated the efficiency of this approach, with system metrics recording a CPU usage of only ~0.3% and a memory footprint of ~13.4% in test environments.

In practice, this approach works well because:

- Simple debugging : when something breaks, the cause is usually obvious.

- True multi-cloud flexibility : switch providers with zero code changes.

But it also has limitations:

- No native offline capabilities : messages are lost if there is no connection.

- Updating logic requires redeploying the entire container in a monolithic setup. With a multi-container design, partial updates are possible (see note below).

- No local processing before sending to the cloud.

- Store-and-forward must be built manually if needed.

A note on update granularity : This limitation depends on how the architecture is designed. In a monolithic setup, any code change forces a full rebuild and container restart. However, splitting these responsibilities into separate services in a docker-compose file enables partial updates: changing service X only triggers a rebuild and restart of X, while Y and Z remain untouched. Edge orchestrators like Greengrass take this further. Greengrass manages components as an independent process in its runtime, and when a deployment is triggered from the cloud, only the targeted component is updated without restarting the Greengrass container itself.

In practice, this means faster update cycles with less disruption: a component update takes seconds without interrupting other running components or the device's cloud connectivity, while a container update requires pulling a new image, stopping, and restarting the service (which temporarily disrupts it and consumes more bandwidth). For frequent, small changes to application logic, updates at process level reduce both deployment time and risk.

#### Azure IoT Edge and Comparison with AWS IoT Greengrass

When the limitations of the cloud-agnostic approach become blockers, edge computing orchestrators enter the picture. Both Azure IoT Edge and AWS Greengrass V2 offer runtimes that run on the device and enable capabilities that are difficult to build from scratch.

#### Azure IoT Edge Architecture

Azure IoT Edge extends Azure IoT Hub capabilities to the edge through three main components:

- IoT Edge Security Daemon (aziot-edged): Manages module lifecycle, secure communication and device identity.

- Edge Agent ($edgeAgent): Receives deployment instructions from the cloud and manages which modules should be running on the device.

- Edge Hub ($edgeHub): Acts as a local proxy for IoT Hub, handling message routing between modules and to the cloud, including “store-and-forward” (offline buffering) when connectivity is lost.

Running IoT Edge inside a container requires enabling systemd and careful management of the config.toml generation from environment variables. This adds complexity but enables the full orchestration capabilities.

One of the most significant advantages of IoT Edge is deploying modules without touching the base device image. The workflow:

- Build the module as a Docker container.

- Push to a registry (Azure Container Registry, Docker Hub)

- Configure the deployment manifest in Azure Portal (or via Azure CLI).

- The Edge Agent pulls and runs the module automatically.

#### AWS Greengrass: Architecture and Key Concepts

AWS IoT Greengrass V2 offers similar capabilities with a different architecture. Where Azure IoT Edge relies on a microservices architecture with multiple daemons managed by systemd, Greengrass is built around a single runtime (the Greengrass Nucleus ) that acts as the core engine on the device.

The Greengrass Nucleus is a JVM process that manages the entire component lifecycle: installation, startup, monitoring, and updates. It communicates with AWS IoT Core to receive deployment instructions and report device status. IoT Core handles device communication with the Cloud and Greengrass extends that to the edge by running software locally on the device. Every Greengrass device is also an IoT Core thing .

Components are the deployment unit in Greengrass. They can be:

- Native processes (script, binaries) running directly on the device.

- Docker containers managed by the Docker application manager component.

- Lambda functions deployed to the edge.

Each component has a recipe (YAML or JSON) that defines its lifecycle, dependencies, and configuration. When a deployment is created from the cloud, the Nucleus downloads only the updated components and manages their lifecycle without restarting itself.

Greengrass tends to be more “container-friendly” because it does not depend on systemd as the init process. The Nucleus is a standalone Java process that can run directly without requiring a full init system, which simplifies containerization.

## Fleet Management: Options and Considerations

Regardless of which connectivity approach is chosen, managing the operating system and device updates at scale remains a challenge. Several options exist, each with different trade-offs.

#### The Fleet Management Problem

Managing hundreds of edge devices distributed across different locations present specific challenges:

- Deploying updates safely without physical access.

- Handling rollbacks when an update fails.

- Configuring devices individually without maintaining separate images.

- Monitoring fleet health in real time.

#### Available Approaches

- Dedicated Fleet Management Platforms (e.g. Balena): Provide atomic OTA updates at both OS and application level, delta updates to minimize bandwidth, remote configuration, and rollback capabilities. They abstract much of the complexity but introduce platform dependency.

- Orchestrator-Native Management (e.g., AWS Greengrass or Azure IoT Edge): Utilizes the deployment capabilities of the cloud orchestrator to manage the device's software. While this approach reduces the number of external tools by staying in a single ecosystem, it is typically limited to the application layer (containers or components). This leaves Host OS updates, kernel health, and low-level system recovery as separate challenges that still need to be addressed.

- Custom Solutions: Build update mechanisms tailored to specific requirements. Offers maximum flexibility but requires significant development and testing effort. Makes sense for highly specialized deployments or when existing tools do not fit the constraints.

The choice depends on fleet size, update frequency, connectivity patterns, and how much operational complexity the team can absorb.

## Use Cases: Choosing the Right Approach

The architecture choice depends fundamentally on the use case. Here are some scenarios and the recommended approach for each.

Scenario: Distributed Sensors reporting temperature, humidity and air quality every five minutes. Stable WiFi connectivity. No local processing required.

Recommended Approach : Cloud Agnostic client (direct MQTT/AMQP)

Why? : Simplicity is sufficient. No need for advanced offline capabilities with stable connectivity. Low resource consumption allows using cheaper hardware. The interval means losing some messages during reconnections is not critical.

Scenario : Devices are installed on construction sites or manufacturing plants and rely on cellular connectivity. Telemetry data is used for predictive maintenance, and losing data during outages is not acceptable.

Recommended Approach : Azure IoT Edge or AWS Greengrass (ideally combined with a dedicated fleet management platform)

Why? : Native store-and-forward ensures that telemetry collected during outages is preserved and uploaded once connectivity is restored. The ability to deploy logic updates remotely allows maintenance models and data processing rules to evolve without replacing device images or requiring on-site intervention.

## Considerations and Lessons Learned

Deploying IoT systems at scale surfaces challenges that are not always obvious during prototyping. The following considerations emerged from real implementations and should inform architecture decisions early in the process.

#### Resource Planning

- Memory Budgets are real constraints: Edge orchestrators consume ~500MB at idle. On devices with low RAM, this leaves limited capacity for application workloads. ‍

- Storage for offline buffering: Store-and-forward requires disk space. Calculate expected message volume during maximum expected outage duration and add margin.

#### Update and Rollout Strategies

- Canary deployments are essential at scale: Never push updates to an entire fleet simultaneously. Start with a small percentage (e.g., 10%), monitor for issues, then gradually expand.

- Rollback must be automatic and tested: Define clear health checks that trigger automatic rollback. Test the rollback mechanism as rigorously as the update mechanism.

- Delta updates matter on constrained networks: Cellular data plans are often metered or have bandwidth limitations. Transferring full images for minor changes is wasteful and slow. ‍

- Versioning and Backward Compatibility: Payload formats and command schemas should be versioned explicitly. Device software evolves slower than cloud services, and backward compatibility becomes critical once fleets are deployed.

#### Security and Credential Management

- Certificate storage requires careful design: X.509 certificates for cloud authentication must be stored securely on the device. Options include encrypted filesystems, or environment variables injected at runtime. Each has different security and operational trade-offs. ‍

- Principle of least privilege: Each device should only have permissions for what it needs. Avoid shared credentials across devices (compromise of one device should not compromise the fleet).

#### Connectivity and Resilience

- Assume Connectivity will fail: Design for intermittent connectivity from the start, even if current deployment has reliable networks. ‍

- Proxy and firewall considerations: Enterprise environments often require traffic to pass through proxies. Ensure the connectivity stack supports proxy configuration and that required endpoints are whitelisted.

#### Operational Visibility

- Logging and metrics collection : Decide early how logs and metrics will be collected from devices. Local storage, batched upload, real-time streaming (each has bandwidth and storage implications). ‍

- Remote debugging capabilities: When something fails at a remote site, how will it be diagnosed? Plan for remote shell access, log retrieval, and diagnostic commands.

## Conclusion

The three questions posed at the beginning of this post (how to connect, how to update, and how to manage) do not have a single answer. But they do have a clear decision framework.

For Cloud Connectivity , the choice ranges from direct MQTT/AMQP clients to full edge orchestrators like Azure IoT Edge or AWS Greengrass. The cloud-agnostic approach offers simplicity, flexibility and low resource consumption (ideal for prototypes, MVPs, and scenarios with stable connectivity). Edge orchestrators add complexity and resource overhead but provide tested capabilities: offline operation, remote module deployment, and local processing. The added cost is justified when these requirements are critical.

For Fleet management , options include dedicated platforms, orchestrator tools or custom solutions. Each comes with different trade-offs in operational complexity, vendor dependency, and development effort. The choice should be evaluated independently from the connectivity approach.

Beyond architecture, practical considerations around resource planning, rollout strategies, security, and operational visibility often determine success or failure at scale. These concerns should inform decisions early, not after deployment.

There is no universal solution. The right choice depends on connectivity conditions, resource constraints, update requirements, security posture, team experience, and operational capabilities. Understanding the available options (and the practical considerations that come with scale) is the first step toward making an informed decision.

## References

- Azure IoT Edge Documentation

- Azure IoT Hub Documentation

- IoT Edge Runtime and Architecture

- IoT Edge Deployment Manifests

- AWS IoT Greengrass V2 Developer Guide

- How AWS IoT Greengrass Works

- Greengrass Nucleus Component

- Greengrass Component Management

- AWS IoT Core Documentation

- BalenaOS Documentation

- Balena Multi-Container Apps
