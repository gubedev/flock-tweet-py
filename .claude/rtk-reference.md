# RTK Command Reference

**Golden Rule**: Always prefix commands with `rtk`. Safe to use everywhere — passes through if no filter exists.

```bash
# Even in chains:
rtk git add . && rtk git commit -m "msg" && rtk git push
```

## Commands by Category

### Test (90-99% savings)

```bash
rtk pytest              # Python test failures only
rtk vitest run          # Vitest failures only (99.5%)
rtk playwright test     # Playwright failures only (94%)
```

### Build & Compile (80-90% savings)

```bash
rtk tsc                 # TypeScript errors only (83%)
rtk next build          # Next.js build summary (87%)
```

### Git (59-80% savings)

```bash
rtk git status / log / diff / show / add / commit / push / pull / branch / fetch
```

### GitHub (26-87% savings)

```bash
rtk gh pr view <num> / pr checks / pr list
```

### JS/TS Tooling

```bash
rtk npm run <script>
rtk npx <cmd>
```

### Docker

```bash
rtk docker ps / images / logs
rtk docker-compose up / down / exec
```

### Meta

```bash
rtk gain                # Token savings stats
rtk gain --history
rtk discover            # Find missed RTK usage
rtk proxy <cmd>         # Run without filtering (debug)
```
