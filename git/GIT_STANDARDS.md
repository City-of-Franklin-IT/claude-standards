## Deployment
- Build produces a `dist/` directory via `tsc -b && vite build`
- Deploy uses `scp -r dist` to internal servers
- Standard npm scripts pattern:
```json
{
  "dev": "vite --port 6000",
  "build": "tsc -b && vite build",
  "deploy": "scp -r dist $USER@<server>:/opt/builds/<app-name>",
  "lint": "eslint .",
  "preview": "vite preview",
  "test": "vitest"
}
```

# Version Control Guide
  - When prompted to add untracked changes, commit, and push to remote repo, conduct npm audit for high and critical level vulnerabilities. If any exist, prompt to fix them before conducting any git functions
  - Any time a repo is pushed to GitHub - be sure to update the README.md to reflect any changes beforehand. Use /opt/claude-standards/README_STANDARDS.md as the guide
  - Always bring attention to API keys or other sensitive data that should not be commited to the remote repo

## Git ignore
- Be sure unwanted .js files emitted during compilation are committed and pushed to remote repo
- Use the following .gitignore template for any projects you encounter:

```
# Logs
logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
pnpm-debug.log*
lerna-debug.log*

# Dependencies
node_modules

# Build outputs
dist
dist-ssr
build
lib
*.local

# TypeScript compiled output (be more specific)
src/**/*.js
src/**/*.js.map
*.d.ts.map
*.tsbuildinfo
tsconfig.tsbuildinfo

# But keep important config files that are actually JavaScript
!vite.config.js
!tailwind.config.js
!postcss.config.js
!vitest.config.js
!eslint.config.js
!prettier.config.js

# Editor directories and files
.vscode/*
!.vscode/extensions.json
.idea
.DS_Store
*.suo
*.ntvs*
*.njsproj
*.sln
*.sw?

# Environment files
.env
.env.local
.env.*.local
```
