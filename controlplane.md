# Control Plane NGINX Operator Refinement Documentation

## Summary
The current control plane setup involves two configuration files—one for the **default domain** and another for the **custom domain**. These files are managed by Kubernetes jobs for each tenant, which have limitations in fault tolerance and reliability. The objective is to adopt an **NGINX Operator-based pattern**, where configuration management is handled through **Custom Resource Definitions (CRDs)**. This change will enhance the scalability, reliability, and maintenance of NGINX configurations.

---

## Motivation
The existing approach of using Kubernetes jobs to manage tenant-specific configurations is prone to failures, which may lead to service disruptions. Adopting an operator-based model would streamline the configuration process and ensure more **robust and fault-tolerant management** of NGINX instances across tenants.

---

## Goals
- Replace the current Kubernetes jobs for configuration management with an **NGINX Operator**.
- Define two **Route Custom Resources (CRs)**:
  - One for the **default domain** configuration.
  - Another for **tenant-specific custom domain** configurations.
- Align the current requirements with the Route CRD specifications to ensure both configurations are fully supported.
- Provide **tenant-specific configurations** through the custom domain Route CR.

---

## Requirements vs. Route CRD Support

| **Requirement**                | **Current Implementation**   | **Supported by Route CRD?** | **Comments**                                            |
|---------------------------------|------------------------------|-----------------------------|---------------------------------------------------------|
| Default domain configuration    | Managed by K8s job            | Yes                         | The default domain will have a dedicated Route CR.       |
| Custom domain configuration     | Tenant-specific and managed by K8s job | Yes               | Each tenant will have a custom Route CR for its domain.  |

---

## Design Considerations
The control plane will require two Route CRs:
- One for the **default domain**.
- One for each **tenant's custom domain**.

---

### Default Domain Configuration

- **Purpose**:  
  This configuration file is responsible for handling traffic to the default domain of the control plane. It manages SSL and routing for requests using a **wildcard subdomain pattern** (`*.qdn`), allowing dynamic handling of requests across different regions or services.

- **Logging and SSL Setup**:  
  It includes **dynamic logging** that differentiates based on whether a custom log file name is provided or defaults to the tenant name. SSL certificates are dynamically managed using the provided **TLS secret**, ensuring secure communication.

- **Proxy Settings**:  
  It defines the `proxy_pass` directive to forward requests to the control plane's **upstream endpoint**. It also includes configurable timeouts (`proxy_connect_timeout`, `proxy_send_timeout`, and `proxy_read_timeout`) to ensure robust handling of network conditions.

---

### Custom Domain Configuration

- **Purpose**:  
  This configuration file is responsible for managing **tenant-specific custom domains**. Each tenant's custom domain will have its own server block, enabling dedicated routing and security settings.

- **Logging and SSL Setup**:  
  Similar to the default domain, logging is **tenant-specific**, and SSL certificates are managed via a **custom TLS secret** for each tenant. This setup ensures that each custom domain operates under a secure environment.

- **Proxy Settings and Redirection**:  
  Requests are proxied to the control plane’s **upstream endpoint**. Additionally, HTTP-to-HTTPS redirection is enforced to ensure all traffic is securely transmitted over **HTTPS**.

