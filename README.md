### Dockerfile Breakdown

#### 1. **Set Node.js Version**

```dockerfile
ARG NODE_VERSION=19.6
```

- Defines a build-time variable `NODE_VERSION` with the value `19.6`.

#### 2. **Base Image**

```dockerfile
FROM node:${NODE_VERSION}-alpine AS baseImage
WORKDIR /usr/src/app
COPY package*.json .
```

- Uses the Node.js Alpine image based on the specified `NODE_VERSION`.
- Sets the working directory to `/usr/src/app`.
- Copies `package.json` and `package-lock.json` (if present) to the working directory.

#### 3. **Dependencies Stage**

```dockerfile
FROM baseImage AS dependencies
RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm install
```

- Uses the `baseImage` stage as the base.
- Utilizes a cache mount for the npm cache to speed up subsequent builds.
- Sets the npm cache directory and installs dependencies.

#### 4. **Development Stage**

```dockerfile
FROM dependencies AS development
COPY . .
CMD ["npm", "run", "dev"]
```

- Uses the `dependencies` stage as the base.
- Copies the entire application source code to the working directory.
- Sets the default command to run the development server.

#### 5. **Production Stage**

```dockerfile
FROM development as production
ENV NODE_ENV production
RUN --mount=type=cache,target=/usr/src/app/.npm \
    npm set cache /usr/src/app/.npm && \
    npm ci --only=production
USER node
COPY --chown=node:node ./src/ .
COPY --chown=node:node ./healthcheck/ .
```

- Uses the `development` stage as the base.
- Sets the `NODE_ENV` environment variable to `production`.
- Utilizes a cache mount for the npm cache and installs only production dependencies.
- Switches to the `node` user for security.
- Copies the source code and healthcheck scripts with appropriate ownership.

#### 6. **Health Check**

```dockerfile
HEALTHCHECK CMD node healthcheck.js
```

- Defines a health check command to ensure the container is running properly.

#### 7. **Expose Port**

```dockerfile
EXPOSE 3000
```

- Exposes port `3000` for the application.

#### 8. **Start Application**

```dockerfile
CMD [ "node", "index.js" ]
```

- Sets the default command to start the application.

### Summary

This Dockerfile is structured to create a multi-stage build for a Node.js application. It optimizes the build process by caching dependencies and separating development and production stages. The use of health checks and user switching enhances the security and reliability of the container.

If you have any questions or need further assistance, feel free to ask!

### Try My Docker Container To See It Actually Work

#### Pull Image From My Docker Hub Repository

- docker pull hazem196/api-node
