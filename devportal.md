# DevPortal NGINX Operator Refinement Documentation

## Summary
DevPortal’s configuration involves two configuration files—one for the default domain and another for custom domains. The system leverages an NGINX Operator-based approach where configuration is managed using Custom Resource Definitions (CRDs). Additionally, Cloudflare is integrated for SSL verification. When Cloudflare is enabled, SSL certificates are verified through it; when disabled, the system handles SSL verification internally.

## Motivation
The current system, which uses Kubernetes jobs for managing domain-specific configurations, lacks robustness and fault tolerance. Transitioning to an operator-based model will enhance scalability and reliability, while Cloudflare integration will provide improved SSL handling.

## Goals
- Replace Kubernetes jobs with an NGINX Operator for configuration management.
- Introduce two Route Custom Resources (CRs):
  1. One for the default domain configuration.
  2. Another for tenant-specific custom domain configurations.
- Align DevPortal’s requirements with Route CRD specifications, and incorporate Cloudflare integration.
- Ensure seamless Cloudflare SSL verification, with a fallback to the internal SSL verification method when Cloudflare is disabled.

## Requirements vs. Route CRD Support

| Requirement                     | Current Implementation                  | Supported by Route CRD? | Comments                                                                                      |
|----------------------------------|-----------------------------------------|-------------------------|-----------------------------------------------------------------------------------------------|
| Default domain configuration     | Managed by K8s job                      | Yes                     | The default domain will have a dedicated Route CR.                                             |
| Custom domain configuration      | Tenant-specific and managed by K8s job  | Yes                     | Each tenant will have a custom Route CR for its domain.                                        |
| Cloudflare SSL verification      | Not part of current setup               | Yes                     | SSL certificates are verified through Cloudflare when enabled, falling back to direct SSL.     |

## Design Considerations
DevPortal will require the following two Route CRs:
1. One for the default domain.
2. One for each tenant’s custom domain.

### Default Domain Configuration

- **Purpose:**  
  Manages traffic to DevPortal’s default domain, including SSL verification and routing via a wildcard subdomain pattern (e.g., `*.qdn`). It also handles dynamic requests across various services.
  
- **Logging and SSL Setup:**  
  Logging is dynamic, and SSL certificates are handled through TLS secrets. When Cloudflare is enabled, SSL verification is performed through Cloudflare; otherwise, direct SSL verification is used.

- **Proxy Settings:**  
  Requests are forwarded to DevPortal’s upstream endpoint. Configurable timeouts ensure network conditions are handled appropriately.

### Custom Domain Configuration

- **Purpose:**  
  Manages tenant-specific custom domains, with dedicated routing and security per tenant.

- **Logging and SSL Setup:**  
  SSL certificates are managed per tenant through custom TLS secrets. Cloudflare integration allows for SSL verification through their service, falling back to internal verification when Cloudflare is disabled.

- **Proxy Settings and Redirection:**  
  Requests are proxied to the upstream endpoint, and HTTP-to-HTTPS redirection is enforced for secure communication.

## Cloudflare Integration

- **Enabled:**  
  When enabled, Cloudflare will manage SSL certificate verification. The NGINX configuration will ensure that requests are routed through Cloudflare, leveraging their CDN and security features.
  
- **Disabled:**  
  When disabled, SSL certificates are verified internally.
