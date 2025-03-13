# Project Guidelines and Commands

## Build Commands
- `npm run dev` or `pnpm dev` - Start development server with auto-reload
- `npm run build` or `pnpm build` - Build for production
- `npm run export` or `pnpm export` - Export slides to PDF

## Code Style Guidelines
- Vue components use Composition API with `<script setup lang="ts">` syntax
- Component structure: script → template → style
- Use TypeScript for type safety (`lang="ts"` in script tags)
- PascalCase for component filenames (e.g., `Counter.vue`, `ProjectTimeline.vue`)
- camelCase for variables, functions, methods, and props
- Utility-first CSS with custom classes where needed
- Define props with type validation and required status
- Use `computed()` for derived state
- Prefer defensive coding with optional chaining when appropriate

## Error Handling
- Use Math.max/min for value clamping
- Handle null/undefined values with optional chaining
- Use computed properties for derived data to prevent runtime errors

## Project Structure
- `/components` - Reusable Vue components
- `/layouts` - Slidev layout templates
- `/pages` - Additional slide content
- `/snippets` - Code snippets for slides