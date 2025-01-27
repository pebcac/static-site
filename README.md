# OpenShift Static Site Deployment

This repository contains configuration files for deploying a static website on OpenShift. The setup uses NGINX to serve static content and includes all necessary OpenShift resources for a production-ready deployment.

## Prerequisites

- OpenShift CLI (`oc`) installed and configured
- Access to an OpenShift cluster with necessary permissions
- Git repository containing your static website content
- `envsubst` command (part of `gettext` package) for variable substitution

## Components

The deployment consists of several OpenShift resources:

- **BuildConfig**: Creates an image using NGINX to serve your static content
- **ImageStream**: Manages the container images for your application
- **DeploymentConfig**: Manages the deployment with 2 replicas for high availability
- **Service**: Provides internal networking for the application
- **Route**: Exposes the application externally with TLS termination

## Resource Specifications

- CPU Request: 100m
- CPU Limit: 200m
- Memory Request: 128Mi
- Memory Limit: 256Mi
- Replicas: 2

## Deployment Steps

1. Clone this repository:
   ```bash
   git clone <repository-url>
   cd <repository-directory>
   ```

2. Set required environment variables:
   ```bash
   # Your static site repository URL
   export REPOSITORY_URL="https://github.com/yourusername/your-static-site"
   
   # Your OpenShift project name
   export PROJECT_NAME=$(oc project -q)
   ```

3. Import the NGINX base image:
   ```bash
   oc import-image nginx --from=registry.redhat.io/ubi8/nginx-120 --confirm
   ```

4. Deploy to OpenShift:
   ```bash
   envsubst < static-site.yaml | oc apply -f -
   ```

5. Start the build:
   ```bash
   oc start-build static-site
   ```

## Build and Deployment Verification

1. Monitor the build progress:
   ```bash
   oc logs -f bc/static-site
   ```

2. Check deployment status:
   ```bash
   oc get pods
   oc get dc
   ```

3. Verify the route:
   ```bash
   oc get route static-site -o jsonpath='{.spec.host}{"\n"}'
   ```

## Configuration Details

### BuildConfig

The BuildConfig uses the Source strategy with NGINX as the base image:
- Sources content from your Git repository
- Uses OpenShift's NGINX builder image
- Outputs to an ImageStream tag

### ImageStream

The ImageStream manages your application images:
- Tags: latest
- Automatically triggers new deployments on image updates
- Internal registry path: image-registry.openshift-image-registry.svc:5000/${PROJECT_NAME}/static-site:latest

### DeploymentConfig

The deployment includes:
- Health checks (liveness and readiness probes)
- Resource limits and requests
- Rolling deployment strategy
- Automatic triggers for image and configuration changes

## Troubleshooting

### Common Issues and Solutions

1. Image Pull Errors
   ```
   Error: ErrImagePull or ImagePullBackOff
   ```
   Solutions:
   - Verify ImageStream exists: `oc get is`
   - Check build completed successfully: `oc logs -f bc/static-site`
   - Ensure project name is correct in image path
   - Verify NGINX base image is imported

2. Build Failures
   - Check build logs: `oc logs -f bc/static-site`
   - Verify Git repository accessibility
   - Ensure NGINX builder image is available: `oc get is nginx -n openshift`

3. Deployment Issues
   - Check deployment events: `oc describe dc/static-site`
   - Verify pod logs: `oc logs <pod-name>`
   - Check resource quotas: `oc get quota`

### Debug Commands

```bash
# Check build status
oc get builds
oc describe build/static-site-1

# Verify image stream
oc get imagestream static-site
oc describe imagestream static-site

# Check deployment
oc rollout status dc/static-site
oc describe dc/static-site

# Pod verification
oc get pods
oc describe pod <pod-name>
oc logs <pod-name>
```

## Customization

### NGINX Configuration

To customize the NGINX configuration:

1. Create a ConfigMap:
   ```bash
   oc create configmap nginx-config --from-file=nginx.conf
   ```

2. Mount it in the DeploymentConfig:
   ```yaml
   volumes:
   - name: nginx-config
     configMap:
       name: nginx-config
   volumeMounts:
   - name: nginx-config
     mountPath: /etc/nginx/nginx.conf
     subPath: nginx.conf
   ```

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.
