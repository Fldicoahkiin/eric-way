# Write React in Eric way

1. Use `type XXXProps` and `FC<XXXProps>`
2. If Eric's writing React, react compiler must be enabled, so agent do not need to write `useMemo` or `useCallback` unnecessarily.
3. Never use `React` namespace.
4. Use `ReactNode` type for render-able children, slot, and other renderable content.
5. Expose native elements props when creating custom components, such as `interface InputProps extends ComponentProps<"input"> {}` and `...` in component's root element.
6.
