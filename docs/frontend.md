# How to write Frontend code in Eric way

1. Use tailwind rather than writing CSS. Eric prefers to use tailwind for styling, as it allows for faster development and easier maintenance of styles.
2. Use tailwind along side component library is ok, since we can use tailwind to reduce duplicate code and make the code more readable.
3. Use FSD (Feature-Sliced Design) architecture for organizing your code. This means that each feature should have its own folder, which contains all the necessary files for that feature (e.g., components, services, etc.).
4. Always use TypeScript, there's no reason to use JavaScript in Eric way. Only some configuration files can be written in JavaScript, but the main application code should always be written in TypeScript for better type safety and maintainability.

## Data

1. Treat every non-trivial data source as server state from the renderer's point of view, even when it is local.
2. Put query keys in one shared file named `queryKeys.ts` or `query-keys.ts`; do not inline query key arrays across components.
3. Prefer `queryOptions` and `mutationOptions` helpers for reusable feature queries instead of hiding everything in page components.
4. Mutations should invalidate the narrow query groups they affect, not refetch the whole app.
