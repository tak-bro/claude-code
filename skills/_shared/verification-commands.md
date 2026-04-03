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

### Node.js (Backend)
```bash
{pm} run lint                          # Lint
{pm} test                              # Unit tests
{pm} run build                         # TypeScript compilation
{pm} run test:integration              # Integration tests (if configured)
```

### Android (Kotlin)
```bash
./gradlew lint{Variant}                    # Lint check
./gradlew test{Variant}UnitTest            # Unit tests
./gradlew assembleDebug                    # Build check
./gradlew detekt                           # Static analysis (if configured)
./gradlew ktlintCheck                      # Code style (if configured)
```

### iOS (Swift)
```bash
# Xcode project
xcodebuild -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator build
xcodebuild test -workspace {Name}.xcworkspace -scheme {Scheme} -sdk iphonesimulator -destination 'platform=iOS Simulator,name=iPhone 16'

# Swift Package
swift build
swift test

# Linting
swiftlint lint --strict                    # if configured
swiftformat --lint .                        # if configured
```

## React-Specific Grep Checks

```bash
# Default exports
grep -r "export default" libs/{feature}/

# any usage
grep -r ": any" libs/{feature}/ --include="*.ts" --include="*.tsx"
```

## Node.js Backend Grep Checks

```bash
# DB queries in route handlers (should be in repository)
grep -rn "prisma\.\|\.findOne\|\.findMany\|\.aggregate\|\.insertOne" --include="*.route.ts" --include="*.controller.ts"

# Scattered process.env access (should be centralized in config)
grep -rn "process\.env\." --include="*.ts" | grep -v "config\|\.test\.\|\.spec\.\|node_modules"

# Hardcoded secrets patterns
grep -rn "password.*=.*['\"]" --include="*.ts" | grep -v "\.test\.\|\.spec\.\|schema\|type\|interface"

# Missing error propagation (next not called)
grep -rn "catch.*err" --include="*.route.ts" | grep -v "next(err)"
```

## Android-Specific Grep Checks

```bash
# MutableStateFlow exposed publicly (should be private)
grep -r "val.*MutableStateFlow" --include="*.kt" | grep -v "private"

# GlobalScope usage (should use viewModelScope)
grep -r "GlobalScope" --include="*.kt"

# Force unwrap equivalent (should use safe calls)
grep -r "\.first()" --include="*.kt" | grep -v "firstOrNull"

# Missing @Inject (ViewModel without injection)
grep -r "class.*ViewModel" --include="*.kt" | grep -v "@Inject\|@HiltViewModel"
```

## iOS-Specific Grep Checks

```bash
# Force unwrapping (should be 0 in new code)
grep -rn '!' --include="*.swift" | grep -v '//' | grep -v 'IBOutlet\|IBAction\|#if\|import\|!=\|Protocol'

# Completion handler in new code (should use async/await)
grep -rn 'completion:.*@escaping' --include="*.swift"

# Missing weak self in closures
grep -rn '{ self\.' --include="*.swift" | grep -v '\[weak self\]\|\[unowned self\]'
```
