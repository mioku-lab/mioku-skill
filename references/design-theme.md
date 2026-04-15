# Mioku Design System Reference

## Scope

Use this reference when creating WebUI pages, screenshots, help images, or any visual content that requires Mioku's design system.

## Design Philosophy

Mioku's design emphasizes:

- **Teal/cyan primary color**: Evokes technology, freshness, and calm
- **Glassmorphism**: backdrop-filter blur for depth and elegance
- **Subtle grid patterns**: Adds visual interest without distraction
- **Smooth transitions**: 150-320ms animations with easing curves
- **Light/dark mode consistency**: Same structure, adjusted hue/saturation

## CSS Variables (mioku-webui)

### Light Mode

```css
:root {
  --background: 190 78% 98%; /* Very light cyan */
  --foreground: 197 70% 12%; /* Dark teal-gray */
  --card: 0 0% 100%; /* Pure white */
  --card-foreground: 197 70% 14%;
  --border: 186 40% 82%; /* Light teal */
  --input: 186 40% 86%;
  --primary: 180 73% 43%; /* Teal */
  --primary-foreground: 0 0% 100%;
  --secondary: 185 69% 92%; /* Light cyan */
  --secondary-foreground: 197 70% 16%;
  --muted: 188 45% 92%; /* Muted cyan */
  --muted-foreground: 193 24% 38%;
}
```

### Dark Mode

```css
.dark {
  --background: 199 42% 8%; /* Very dark blue-gray */
  --foreground: 186 66% 94%; /* Light cyan */
  --card: 201 37% 12%; /* Dark card */
  --card-foreground: 186 66% 94%;
  --border: 191 35% 24%; /* Dark teal border */
  --input: 191 35% 22%;
  --primary: 180 68% 46%; /* Bright teal */
  --primary-foreground: 199 42% 8%;
  --secondary: 190 28% 18%; /* Dark secondary */
  --secondary-foreground: 183 64% 90%;
  --muted: 195 24% 16%; /* Dark muted */
  --muted-foreground: 186 25% 70%;
}
```

## Background Style

### Light Mode

Three-layer radial gradient overlay on hsl background:

```css
background:
  radial-gradient(circle at 0% 0%, rgba(49, 203, 191, 0.16), transparent 36%),
  radial-gradient(circle at 90% 20%, rgba(52, 171, 192, 0.2), transparent 44%),
  hsl(var(--background));
```

### Dark Mode

Same structure, darker base with adjusted accent positions.

## Markdown Screenshot Theme

Used in `plugins/chat/core/markdown-message.ts` for generating help images and code screenshots.

### Light Theme

| Property    | Value                                      |
|-------------|--------------------------------------------|
| pageBg      | linear-gradient(#eefcfb, #ecfbfe, #f6fbff) |
| shellBg     | rgba(255, 255, 255, 0.78)                  |
| shellBorder | rgba(132, 204, 196, 0.35)                  |
| shellShadow | 0 34px 80px rgba(30, 91, 95, 0.18)         |
| foreground  | #12343a                                    |
| heading     | #0f2d34                                    |
| link        | #0f766e                                    |
| quoteBorder | #2dd4bf                                    |
| codeBg      | #f6fffe                                    |
| dotA        | #fb7185 (red)                              |
| dotB        | #facc15 (yellow)                           |
| dotC        | #34d399 (green)                            |

### Dark Theme

| Property    | Value                                      |
|-------------|--------------------------------------------|
| pageBg      | linear-gradient(#07141c, #0b1c25, #102730) |
| shellBg     | rgba(9, 24, 32, 0.72)                      |
| shellBorder | rgba(56, 189, 248, 0.14)                   |
| shellShadow | 0 34px 80px rgba(0, 0, 0, 0.36)            |
| foreground  | #dcf7f5                                    |
| heading     | #f2fffe                                    |
| link        | #67e8f9                                    |
| quoteBorder | #2dd4bf                                    |
| codeBg      | #071722                                    |

### Shared Screenshot Elements

- **Glassmorphism**: `backdrop-filter: blur(18px)` on shell
- **Grid pattern**: 1px lines, 30px grid, rgba(13, 148, 136, 0.06) (light) / rgba(151, 214, 210, 0.04) (dark)
- **Radial accents**: 2-3 layered radial gradients for page depth
- **Rounded corners**: 28px (shell), 22px (code block), 18px (blockquote, table)
- **Page padding**: 22-26px based on viewport width

### Code Highlighting Colors

**Light mode**:

- text: #17434a
- comment: #6b8790
- keyword: #0f766e
- number: #2563eb
- string: #b45309
- title: #7c3aed
- type: #be185d
- attr: #0f766e
- builtin: #0284c7
- symbol: #c2410c

**Dark mode**:

- text: #d8f6ff
- comment: #6f9aa6
- keyword: #5eead4
- number: #7dd3fc
- string: #fdba74
- title: #c4b5fd
- type: #f9a8d4
- attr: #5eead4
- builtin: #67e8f9
- symbol: #fda4af

## Animations & Transitions

### Page Transitions

- **fadeIn**: 0.2s ease-out
- **scaleIn**: 0.2s ease-out (scale 0.95 → 1)
- **slideInRight**: 0.3s ease-out

### Component Transitions

- **Topbar chips**: cubic-bezier(0.22, 0.9, 0.3, 1), 320ms
- **Clock colon pulse**: 0.92 → 0.28 opacity
- **Form elements**: all 0.15s ease
- **Scrollbar**: gradient from primary 0.9 → 0.55 opacity

### Important Notes

- Always respect `prefers-reduced-motion: reduce`
- Use `will-change: transform, opacity` for animated elements

## Font Stack

- **Chinese**: "Noto Sans SC", "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", sans-serif
- **Code**: "JetBrains Mono", "SFMono-Regular", "Consolas", "Liberation Mono", monospace

## Best Practices

1. **Always support both modes**: Define light and dark theme objects; detect mode via time (hour >= 19 || hour < 7) or CSS media query
2. **Use teal/cyan as primary**: Primary color should be around hue 170-200
3. **Glassmorphism**: Apply `backdrop-filter: blur(18px)` on cards/containers
4. **Grid texture**: Add subtle 30px grid pattern for depth
5. **Rounded corners**: Use generous border-radius (18-28px) for modern feel
6. **Accent gradients**: Layer 2-3 radial gradients for visual interest
7. **Smooth transitions**: Use 150-320ms with ease or cubic-bezier curves
8. **Respect system preference**: Use auto theme detection with prefers-color-scheme

## Reference Files

- `mioku-webui/src/styles/globals.css` - CSS variables and animations
- `mioku-webui/src/lib/theme.ts` - Theme mode handling (light/dark/auto)
- `plugins/chat/core/markdown-message.ts` - Screenshot rendering with full theme objects
