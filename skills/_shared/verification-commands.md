# Verification Commands

Always use the project's package manager. Detect it first:

```bash
if [ -f "bun.lockb" ]; then pm="bun"
elif [ -f "pnpm-lock.yaml" ]; then pm="pnpm"
elif [ -f "yarn.lock" ]; then pm="yarn"
else pm="npm"; fi
```

## By Framework

### Angular
```bash
ng build              # if available
{pm} run lint         # if available
ng test --watch=false # if available
ng build --configuration=production  # final check
```

### React (Nx)
```bash
{pm} run lint                                          # if available
{pm} test                                              # if available
{pm} run build                                         # if available
# Detailed test commands
{pm} vitest run --reporter=verbose src/features/{feature}  # specific feature
{pm} vitest run --coverage                             # coverage report
{pm} vitest watch                                      # watch mode (dev)
```

### TypeScript
```bash
{pm} run lint    # if available
{pm} test        # if available
{pm} run build   # if available
# Detailed test commands
{pm} vitest run --reporter=verbose src/{module}        # specific module
{pm} vitest run --coverage                             # coverage report
```

## Grep Checks

```bash
# Export defaults (should be 0, except App entrypoint/config)
grep -r "export default" src/ --include="*.ts" --include="*.tsx"

# any usage (should be justified)
grep -r ": any" src/ --include="*.ts" --include="*.tsx"

# function keyword (should be 0 for new code)
grep -r "^function \|^export function " src/ --include="*.ts"
```

## Angular-Specific Grep Checks

```bash
# Standalone components (should be 0)
grep -r "standalone: true" src/app/modules/

# Pages not extending IonBasePage (Ionic)
grep -r "class.*Page" src/app/modules/ --include="*.page.ts" | grep -v "extends IonBasePage"

# Missing super.ionViewWillEnter() (Ionic)
grep -rL "super.ionViewWillEnter" src/app/modules/ --include="*.page.ts"

# Direct controller usage (Ionic - should use service wrappers)
grep -r "modalCtrl\|toastCtrl\|loadingCtrl" src/app/modules/ --include="*.ts"

# IonicRouteStrategy configured
grep -r "IonicRouteStrategy" src/app/app.module.ts

# localStorage without env suffix (multi-env)
grep -r "localStorage\|sessionStorage" src/app/ | grep -v "_\${environment"
```

## React-Specific Grep Checks

```bash
# Default exports
grep -r "export default" libs/{feature}/

# any usage
grep -r ": any" libs/{feature}/ --include="*.ts" --include="*.tsx"
```
