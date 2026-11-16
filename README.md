# ASP.NET Ghostscript Docker Images

This repository contains Docker images based on ASP.NET 11.0 on Ubuntu 26.04 (Resolute) with Ghostscript tools installed.

> **Note:** The images were previously published as `aspnet-noble-ghostscript` and `aspnet-noble-ghostscript-xps` (ASP.NET 10.0 on Ubuntu 24.04 Noble). Those images are no longer rebuilt; .NET 11 is not published on Noble, so switch to the `aspnet-ghostscript` / `aspnet-ghostscript-xps` images when moving to .NET 11.

## Docker Images

The repository provides two Docker images:

### aspnet-ghostscript (Root)
- Base: `mcr.microsoft.com/dotnet/aspnet:11.0-resolute`
- Ghostscript for PDF processing
- zbar-tools for barcode/QR code processing
- Pre-configured GhostScript settings

### aspnet-ghostscript-xps (XPS)
- Base: `mcr.microsoft.com/dotnet/aspnet:11.0-resolute`
- Ghostscript 10.08.0
- gpcl6 for PCL processing
- gxps for XPS processing
- Full GhostPDL suite

## GitHub Actions CI/CD

### Automated Builds

The repository includes a GitHub Actions workflow that:
- Runs nightly at 2 AM UTC
- Triggers on pushes to the main branch
- Builds (without pushing) on pull requests targeting the main branch
- Builds both Docker images in parallel using matrix strategy
- Pushes to Azure Container Registry
- Supports manual triggering via `workflow_dispatch`

### Required GitHub Secrets

To use the GitHub Actions workflow, you need to configure the following secrets in your repository:

1. **ACR_LOGIN_SERVER** - Your Azure Container Registry login server URL
   - Example: `myregistry.azurecr.io`

2. **ACR_USERNAME** - Your Azure Container Registry username
   - Example: `myregistry` (usually the same as registry name)

3. **ACR_PASSWORD** - Your Azure Container Registry password
   - You can get this from Azure Portal > Container Registry > Access Keys

### Setting up GitHub Secrets

1. Go to your GitHub repository
2. Navigate to Settings > Secrets and variables > Actions
3. Click "New repository secret"
4. Add each of the three secrets listed above

### Workflow Features

- **Scheduled builds**: Runs automatically every night at 2 AM UTC
- **Push triggers**: Runs on every push to the main branch
- **Pull request validation**: Builds both images on pull requests without logging in or pushing
- **Matrix strategy**: Builds both images in parallel for faster execution
- **Manual triggers**: Can be triggered manually from the Actions tab
- **Simple tagging**: Uses only the `latest` tag for all builds
- **Docker CLI**: Uses standard `docker build` and `docker push` commands

### Image Tags

The workflow creates a single tag for each image:
- `aspnet-ghostscript:latest` - Standard Ghostscript image
- `aspnet-ghostscript-xps:latest` - Full GhostPDL suite image

## Usage

Once the images are built and pushed to your Azure Container Registry, you can pull them using:

```bash
# Standard Ghostscript image
docker pull <your-registry>.azurecr.io/aspnet-ghostscript:latest

# Full GhostPDL suite image (includes XPS and PCL support)
docker pull <your-registry>.azurecr.io/aspnet-ghostscript-xps:latest
```

## Environment Variables

The standard image (`aspnet-ghostscript`) includes pre-configured environment variables for GhostScript:

- `GhostScriptSettings__Executable="/usr/bin/gs"`
- `GhostScriptSettings__Parameter="-sDEVICE=pdfwrite -o \"{1}\" -dCompatibilityLevel=\"1.4\" -dPDFSETTINGS=\"/screen\" -dNOPAUSE -dQUIET -dBATCH \"{0}\""`
- `GhostScriptSettings__WorkDir="/tmp"`

The XPS image (`aspnet-ghostscript-xps`) includes the full GhostPDL suite with:
- Ghostscript 10.08.0 at `/usr/local/bin/gs`
- gpcl6 for PCL processing at `/usr/local/bin/gpcl6`
- gxps for XPS processing at `/usr/local/bin/gxps`
