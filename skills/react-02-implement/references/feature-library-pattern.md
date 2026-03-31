# Feature Library Pattern (React + Nx)

## API Function

```typescript
export const fetchProducts = async (): Promise<Product[]> => {
  const { data } = await webCore.buildSignedRequest({ method: 'GET', baseURL: '/products' }).execute();
  return data;
};
```

## Query Keys Factory

```typescript
export const productsKeys = createQueryKeys('products');
```

## TanStack Query Hook

```typescript
export const useProducts = () => useQuery({
  queryKey: productsKeys.all,
  queryFn: fetchProducts,
});
```

## Mutation Hook

```typescript
export const useCreateProduct = () => {
  const queryClient = useQueryClient();
  return useMutation({
    mutationFn: (data: CreateProductDto) => createProduct(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: productsKeys.lists() });
    },
  });
};
```

## Component

```typescript
export const ProductList = () => {
  const { data, isLoading } = useProducts();
  if (isLoading) return <div>Loading...</div>;
  return <div className={cn('flex gap-4')}>{data?.map(...)}</div>;
};
```

## Zustand Store (Global Client State)

```typescript
interface FeatureState {
  filter: FilterType;
  setFilter: (filter: FilterType) => void;
}

export const useFeatureStore = create<FeatureState>((set) => ({
  filter: 'all',
  setFilter: (filter) => set({ filter }),
}));
```
