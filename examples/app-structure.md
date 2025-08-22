# Application Repository Structure

## 1. Application Code (app-repo)

### package.json
```json
{
  "name": "my-node-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node server.js",
    "test": "jest"
  },
  "dependencies": {
    "express": "^4.18.0"
  }
}
```

### server.js
```javascript
const express = require('express');
const app = express();
const port = 3000;

app.get('/', (req, res) => {
  res.json({ 
    message: 'Hello from Node.js!', 
    version: process.env.APP_VERSION || '1.0.0' 
  });
});

app.listen(port, () => {
  console.log(`App running on port ${port}`);
});
```

### Dockerfile
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
EXPOSE 3000
CMD ["npm", "start"]
```

## 2. CI Pipeline (.github/workflows/ci.yml)

```yaml
name: CI Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - run: npm test

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Log in to Container Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and push Docker image
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
  
  update-gitops:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Update GitOps repo
        run: |
          # Clone config repo
          git clone https://github.com/khoitranlord/argo-cicd.git config
          cd config
          
          # Update image tag in deployment
          sed -i "s|image: .*|image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}|" k8s/deployment.yaml
          
          # Commit and push
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .
          git commit -m "Update image to ${{ github.sha }}"
          git push
        env:
          GITHUB_TOKEN: ${{ secrets.GITOPS_TOKEN }}
```

## 3. Kubernetes Manifests in GitOps Repo

These would be in your current argo-cicd repo, updated by CI.