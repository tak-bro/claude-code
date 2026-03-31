# TanStack Query Pattern (Angular)

For server state management alongside ComponentStore:

## QueryService Pattern

```typescript
// services/queries/feature-query.service.ts
@Injectable({ providedIn: 'root' })
export class FeatureQueryService {
    featureKeys = this.queryService.createQueryKeys('feature');

    constructor(
        private readonly queryService: TanstackQueryService,
        private readonly queryClient: QueryClient,
        private readonly featureApiService: FeatureApiService
    ) {}

    useFeatures = () => {
        return injectQuery(() => ({
            queryKey: this.featureKeys.lists(),
            queryFn: () => this.featureApiService.fetchFeatures$(),
            staleTime: 1000 * 60 * 5,
        }));
    };

    useInfiniteFeatures = (params: Signal<Params>) => {
        return injectInfiniteQuery(() => ({
            queryKey: this.featureKeys.list(params()),
            queryFn: ({ pageParam }) =>
                this.featureApiService.fetchFeatures$({ ...params(), page: pageParam || 0 }),
            initialPageParam: 0,
            getNextPageParam: (lastPage, allPages) => {
                const loadedCount = allPages.reduce((acc, page) => acc + page.list.length, 0);
                return loadedCount < lastPage.total ? allPages.length : undefined;
            },
        }));
    };
}
```

## Query Keys Factory Pattern

```typescript
// Before ([BAD] - Hard-coded keys)
queryKey: ['features']

// After ([GOOD] - Factory pattern)
featureKeys = this.queryService.createQueryKeys('feature');
queryKey: this.featureKeys.lists()
```
