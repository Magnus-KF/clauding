# Welcome to [Slidev](https://github.com/slidevjs/slidev)!

To start the slide show:

- `pnpm install`
- `pnpm dev`
- visit <http://localhost:3030>

Edit the [slides.md](./slides.md) to see the changes.

## Useful Commands

### Development

```bash
# Start dev server without opening browser automatically
pnpm dev --no-open

# Use a specific port
pnpm dev --port=3000

# Enable Monaco editor for code blocks
pnpm dev -- --monaco

# Enable remote control
pnpm dev -- --remote

# Set log level (debug, info, warn, error)
pnpm dev -- --log=info
```

### Server Management

```bash
# Kill running Slidev server
pkill -f "slidev"
```

### Export & Production

```bash
# Build for production
pnpm build

# Export slides to PDF
pnpm export

# Export slides to PNG
pnpm export -- --format png

# Include clicks in export
pnpm export -- --with-clicks
```

Learn more about Slidev at the [documentation](https://sli.dev/).

## Deployment
[here..](https://luminous-alpaca-eba620.netlify.app/1)
[netlify dash](https://app.netlify.com/sites/luminous-alpaca-eba620/overview)