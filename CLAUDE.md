# Project Guidelines and Commands

## Build Commands
- `pnpm dev` or `npm run dev` - Start development server with auto-reload
- `pnpm build` or `npm run build` - Build for production
- `pnpm export` or `npm run export` - Export slides to PDF

## Code Style Guidelines
- Vue components use Composition API with `<script setup lang="ts">` syntax
- Component structure: script → template → style (with scoped styles)
- PascalCase for component filenames and component registration
- camelCase for variables, functions, methods, and props
- Import order: Vue core → external libraries → internal components/utilities
- Define props with type validation (`defineProps<{...}>()`) 
- Use `computed()` for derived state and `ref()/reactive()` for reactive data
- Prefer `const` over `let` unless reassignment is necessary
- Use consistent indentation (2 spaces) and trailing commas

## Error Handling
- Use optional chaining (`?.`) and nullish coalescing (`??`) for null/undefined
- Implement appropriate fallbacks for missing data
- Use try/catch blocks for async operations
- Validate user input with defensive programming
- Leverage TypeScript for compile-time error prevention

## Project Structure
- `/components` - Reusable Vue components
- `/layouts` - Slidev layout templates
- `/pages` - Slide content files
- `/public` - Static assets (images, fonts)
- `/snippets` - Code snippets for slides