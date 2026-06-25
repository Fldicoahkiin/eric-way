# How to write Frontend code in Eric way

1. Use tailwind rather than writing CSS. Eric prefers to use tailwind for styling, as it allows for faster development and easier maintenance of styles.
2. Use tailwind along side component library is ok, since we can use tailwind to reduce duplicate code and make the code more readable.
3. Use FSD (Feature-Sliced Design) architecture for organizing your code. This means that each feature should have its own folder, which contains all the necessary files for that feature (e.g., components, services, etc.).
4. Components only have one responsibility, and should be small and reusable. If a component is too large or has multiple responsibilities, it should be split into smaller components.
5. Component's responsibility have to be clear, such as "layout", "styling", "composition", "functionality", "data fetching", etc. This will make it easier to understand the purpose of each component and how it fits into the overall architecture of the application.
6. Always use tanstack query for data fetching, as it provides a simple and efficient way to manage server state in your application.
7. Always use TypeScript, there's no reason to use JavaScript in Eric way. Only some configuration files can be written in JavaScript, but the main application code should always be written in TypeScript for better type safety and maintainability.
8.
