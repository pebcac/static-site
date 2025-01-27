# OpenShift Static Site Deployment

This repository contains configuration files for deploying a static website on OpenShift. The setup uses NGINX to serve static content and includes all necessary OpenShift resources for a production-ready deployment.

## Prerequisites

- OpenShift CLI (`oc`) installed and configured
- Access to an OpenShift cluster with necessary permissions
- Git repository containing your static website content
- `envsubst` command (part of `gettext` package) if using variable substitution

## Components

The deployment consists of several OpenShift resources:

- **BuildConfig**: Creates an image using NGINX to serve your static content
- **DeploymentConfig**: Manages the deployment with 2 replicas for high availability
- **Service**: Provides internal networking for the application
- **Route**: Exposes the application externally with TLS termination

## Resource Specifications

- CPU Request: 100m
- CPU Limit: 200m
- Memory Request: 128Mi
- Memory Limit: 256Mi
- Replicas: 2

## Quick Start

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. Set your static site repository URL:
   ```bash
   export REPOSITORY_URL="https://github.com/yourusername/your-static-site"
   ```

3. Deploy to OpenShift:
   ```bash
   # Using variable substitution
   envsubst < static-site.yaml | oc apply -f -
   
   # Or directly if you've modified the YAML file
   oc apply -f static-site.yaml
   ```

## Deployment Verification

1. Check the build status:
   ```bash
   oc get builds
   oc logs -f bc/static-site
   ```

2. Verify the deployment:
   ```bash
   oc get pods
   oc get routes
   ```

3. Access your site using the Route URL:
   ```bash
   oc get route static-site -o jsonpath='{.spec.host}{"\n"}'
   ```

## Configuration

### BuildConfig

The BuildConfig uses the Source strategy with NGINX as the base image. You can customize the build by:

- Modifying the `contextDir` if your static content is in a subdirectory
- Changing the Git reference (`ref`) to use a different branch or tag
- Adding build environment variables if needed

### DeploymentConfig

The deployment is configured with:

- Health checks (liveness and readiness probes)
- Resource limits and requests
- Rolling deployment strategy

### Route Configuration

The Route is configured with edge TLS termination and redirects insecure traffic to HTTPS.

## Customization

### NGINX Configuration

To customize the NGINX configuration:

1. Create a ConfigMap with your custom nginx.conf:
   ```bash
   oc create configmap nginx-config --from-file=nginx.conf
   ```

2. Mount it in the DeploymentConfig:
   ```yaml
   volumes:
   - name: nginx-config
     configMap:
       name: nginx-config
   ```

### Environment Variables

To add environment variables to your deployment:

1. Edit the DeploymentConfig in `static-site.yaml`
2. Add environment variables under the container specification:
   ```yaml
   containers:
   - name: static-site
     env:
     - name: MY_VARIABLE
       value: "my-value"
   ```

## Troubleshooting

### Common Issues

1. Build Failures
   - Verify Git repository accessibility
   - Check build logs: `oc logs -f bc/static-site`

2. Deployment Issues
   - Check pod status: `oc get pods`
   - View pod logs: `oc logs <pod-name>`
   - Describe pod for events: `oc describe pod <pod-name>`

3. Route Access Issues
   - Verify route creation: `oc get routes`
   - Check TLS configuration
   - Verify DNS resolution

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.