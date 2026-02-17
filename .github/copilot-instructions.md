# GitHub Copilot Instructions: ECNS Documentation

> **Self-contained instructions for GitHub Copilot.** Does not reference external files.

---

## Project Overview

Official ECNS (Ethereum Classic Name Service) documentation site. Built with Vocs framework, React 19, and Bun. Deployed to Cloudflare Pages.

**Repository:** https://github.com/ecnsdomains/docs
**Live Site:** docs.ecns.domains (planned)
**Framework:** Vocs 1.4.1+ (documentation framework)

---

## Tech Stack & LTS Versions

| Technology | Version | Notes |
|------------|---------|-------|
| **Runtime** | Bun 1.2.8+ | Primary (Node 22+ fallback) |
| **Framework** | Vocs 1.4.1+ | React-based docs framework |
| **UI** | React 19.2.1 | Server Components |
| **Language** | TypeScript 5.9.3 | Strict mode |
| **Styling** | Tailwind CSS 4.0.7 | CSS-first config |
| **Blockchain** | viem 2.41.2 | Ethereum interactions |
| **Wallet** | wagmi 2.19.5 | Wallet hooks |
| **Queries** | TanStack Query 5.64.1 | Async state |
| **Formatting** | Prettier 3.7.4 | Import sorting |
| **Deployment** | Cloudflare Pages | Wrangler CLI |

### LTS Requirements (CRITICAL)

- **Node.js 24.x** (LTS until Apr 2028) - NOT 22.x (security-only)
- **React 19.x** - Server Components, automatic JSX runtime
- **Next.js 16.x** if used elsewhere - NOT 15.x/14.x (EOL)
- **Bun 1.x** - Preferred package manager and runtime
- **TypeScript 5.x** - Strict mode required

**Never suggest:**
- Node 22.x, 20.x, 18.x (deprecated/EOL)
- React 18.x (upgrade to 19.x)
- Next.js 14.x/15.x (EOL)
- Deprecated blockchain libraries (ethers.js → use viem)

---

## Quick Commands

```bash
# Development
bun install                      # Install dependencies
bun run dev                      # Generate content + dev server
bun run generate                 # Regenerate external content

# Production
bun run build                    # Production build
bun run preview                  # Preview build
bun run deploy                   # Deploy to Cloudflare Pages

# Code Quality
bun run format                   # Format with Prettier
```

---

## Project Structure

```
src/
├── pages/              # MDX documentation
│   ├── index.mdx       # Homepage
│   ├── learn/          # ECNS protocol
│   ├── web/            # SDK guides
│   ├── contracts/      # Smart contracts
│   ├── resolvers/      # Resolver docs
│   ├── wrapper/        # Namewrapper
│   ├── dao/            # Governance
│   └── ensip/          # GENERATED - ECNSIPs
├── components/         # React components
│   ├── ui/             # UI primitives
│   └── *.tsx           # Domain components
└── data/
    └── generated/      # Build-time generated

scripts/                # Content generation
├── ensips.ts           # Fetch ECNSIPs
├── deployments.ts      # Fetch addresses
└── dao-proposals.ts    # Scan proposals

functions/api/          # Cloudflare Functions
├── og.tsx              # OG images
└── blah/               # Analytics proxy

vocs.config.tsx         # Vocs configuration
```

---

## Code Style

### Prettier Rules

- **No semicolons**
- **Single quotes**
- **2-space indentation**
- **Trailing commas (ES5)**
- **Import sorting:**
  1. Third-party (`react`, `viem`)
  2. `@/` aliases
  3. Relative (`./`, `../`)

### TypeScript

- **Strict mode** enabled
- **Explicit types** for function returns
- **`interface`** over `type` for objects
- **`unknown`** over `any`

### File Naming

- Components: `PascalCase.tsx`
- Pages: `kebab-case.mdx`
- Utils: `camelCase.ts`
- Data: `kebab-case.json`

---

## React 19 Patterns

### Server Components (Default)

Most pages are Server Components. Use `'use client'` only when needed.

### When to Use `'use client'`

Required for:
- React hooks (`useState`, `useEffect`, `useQuery`)
- Event handlers (`onClick`, `onChange`)
- Browser APIs (`window`, `localStorage`)
- Wallet interactions (`useAccount`, `useConnect`)

**Example:**

```tsx
'use client'

import { useState } from 'react'
import { useAccount, useReadContract } from 'wagmi'
import { normalize } from 'viem/ens'

export function NameResolver() {
  const [name, setName] = useState('')
  const { address } = useAccount()

  const { data, isLoading } = useReadContract({
    address: '0x...', // ECNS Registry
    abi: [...],
    functionName: 'resolve',
    args: name ? [normalize(name)] : undefined,
    enabled: !!name
  })

  return (
    <div>
      <input
        value={name}
        onChange={(e) => setName(name)}
        placeholder="vitalik.etc"
      />
      {isLoading && <p>Loading...</p>}
      {data && <p>Address: {data}</p>}
    </div>
  )
}
```

---

## Tailwind 4 Styling

### Conditional Classes

Use `clsx` + `tailwind-merge`:

```tsx
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

const cn = (...classes: (string | undefined)[]) => twMerge(clsx(classes))

<div className={cn('p-4', active && 'bg-blue-500', disabled && 'opacity-50')} />
```

### CSS-First Config

Tailwind 4 uses CSS variables, not `tailwind.config.js`:

```css
@import "tailwindcss";

@theme {
  --color-primary: #3b82f6;
  --font-sans: Inter, system-ui, sans-serif;
}
```

---

## MDX Authoring

### Page Template

```mdx
---
title: Page Title
description: SEO description
---

# Heading

Introduction paragraph.

## Section

Code example with syntax highlighting:

```typescript
import { normalize } from 'viem/ens'

const name = normalize('example.etc')
```

<ContractDeployments />

:::info
Callout box for important info
:::
```

### Code Blocks

Features:
- Syntax highlighting
- Line highlighting: ` ```ts {3-5}`
- Filenames: ` ```ts [file.ts]`
- Diffs: `// [!code ++]` or `// [!code --]`

### Components in MDX

Auto-imported from `src/components/`:

```mdx
<ContractDeployments />
<Libraries category="javascript" />
<Avatar name="vitalik.etc" />
```

---

## ECNS-Specific Rules

### Terminology

| ENS | ECNS |
|-----|------|
| `.eth` | `.etc` |
| Ethereum | Ethereum Classic |
| Chain 1 | Chain 61 (mainnet) |
| Goerli | Mordor (chain 63) |
| ensdomains.* | ecnsdomains.* |

### Network Config

```typescript
import { classic, mordor } from 'viem/chains'

// Mainnet
const client = createPublicClient({
  chain: classic, // Chain ID 61
  transport: http('https://etc.rivet.cloud')
})

// Testnet
const testnet = createPublicClient({
  chain: mordor, // Chain ID 63
  transport: http('https://rpc.mordor.etccooperative.org')
})
```

### Contract Names

- `ECNSRegistry` (not `ENSRegistry`)
- `ETCRegistrarController` (not `ETHRegistrarController`)
- Root node: `namehash('etc')` (not `namehash('eth')`)

---

## Code Examples

### TypeScript (viem)

```typescript
import { createPublicClient, http, normalize } from 'viem'
import { classic } from 'viem/chains'

const client = createPublicClient({
  chain: classic,
  transport: http()
})

// Resolve name to address
const address = await client.getEnsAddress({
  name: normalize('vitalik.etc')
})

// Get primary name
const name = await client.getEnsName({
  address: '0x...'
})
```

### Solidity

```solidity
pragma solidity ^0.8.17;

import "@ecnsdomains/ens-contracts/contracts/registry/ECNS.sol";

contract MyResolver {
  ECNS public ecns;

  constructor(ECNS _ecns) {
    ecns = _ecns;
  }

  function resolve(bytes32 node) public view returns (address) {
    return ecns.resolver(node);
  }
}
```

### React Component

```tsx
'use client'

import { useState } from 'react'
import { useReadContract } from 'wagmi'
import { normalize } from 'viem/ens'
import { clsx } from 'clsx'
import { twMerge } from 'tailwind-merge'

const cn = (...classes: (string | undefined)[]) => twMerge(clsx(classes))

export function NameLookup() {
  const [input, setInput] = useState('')

  const { data, isLoading, error } = useReadContract({
    address: '0x...', // ECNS Registry
    abi: [
      {
        name: 'resolve',
        type: 'function',
        stateMutability: 'view',
        inputs: [{ name: 'name', type: 'bytes32' }],
        outputs: [{ name: '', type: 'address' }]
      }
    ],
    functionName: 'resolve',
    args: input ? [normalize(input)] : undefined,
    enabled: !!input
  })

  return (
    <div className="space-y-4">
      <input
        type="text"
        value={input}
        onChange={(e) => setInput(e.target.value)}
        placeholder="vitalik.etc"
        className={cn(
          'w-full px-4 py-2 border rounded-lg',
          error && 'border-red-500'
        )}
      />

      {isLoading && <p className="text-gray-500">Resolving...</p>}

      {error && <p className="text-red-500">Error: {error.message}</p>}

      {data && (
        <div className="p-4 bg-gray-50 rounded-lg">
          <p className="font-mono text-sm">{data}</p>
        </div>
      )}
    </div>
  )
}
```

---

## Vocs Framework

### Configuration

```tsx
// vocs.config.tsx
import { defineConfig } from 'vocs'

export default defineConfig({
  title: 'ECNS Documentation',
  rootDir: 'src',
  sidebar: [
    {
      text: 'Getting Started',
      items: [
        { text: 'Introduction', link: '/intro' },
        { text: 'Quick Start', link: '/quickstart' }
      ]
    }
  ],
  theme: {
    variables: {
      color: {
        background: { light: '#fff', dark: '#000' },
        textAccent: { light: '#3b82f6', dark: '#60a5fa' }
      }
    }
  }
})
```

### Content Generation

Scripts in `scripts/*.ts` fetch external content at build time:

- **ensips.ts** - Fetches ECNSIPs from GitHub
- **deployments.ts** - Fetches contract addresses
- **dao-proposals.ts** - Scans local proposals

Generated files:
- `src/pages/ensip/*.mdx`
- `src/data/generated/*.json`

**Delete to re-fetch** - files are cached locally.

---

## Protected Files

Do not modify without explicit request:

- `vocs.config.tsx` - Core config
- `package.json` - Dependencies
- `scripts/*.ts` - Generation logic
- `functions/api/*.tsx` - Cloudflare Functions
- `src/data/generated/*.json` - Auto-generated
- `src/pages/ensip/*.mdx` - Auto-generated

---

## Validation Workflow

Before committing:

```bash
# 1. Regenerate content
bun run generate

# 2. Format code
bun run format

# 3. Build
bun run build

# 4. Preview
bun run preview  # http://localhost:4173
```

---

## Key Rules

### Always Do

1. Use Bun commands (not npm/yarn)
2. Run `bun run generate` before committing
3. Format with `bun run format`
4. Test with `bun run build`
5. Use `'use client'` only when needed
6. Preserve ENS attribution in docs

### Never Do

1. Commit generated files manually
2. Use npm/yarn (package.json specifies Bun)
3. Skip build testing
4. Hardcode contract addresses
5. Break ECNS→ENS historical credit
6. Use deprecated dependencies

---

## Common Tasks

### Add New Page

1. Create `src/pages/topic/page.mdx`
2. Add frontmatter (title, description)
3. Add to sidebar in `vocs.config.tsx`
4. Test with `bun run dev`

### Create Component

1. Create `src/components/MyComponent.tsx`
2. Add `'use client'` if interactive
3. Use in MDX: `<MyComponent />`
4. Add TypeScript types

### Update Addresses

1. Edit JSONs in `ecnsdomains/ens-contracts` repo
2. Run `bun run generate`
3. Component `<ContractDeployments />` auto-updates

---

## Resources

- **Vocs:** https://vocs.dev
- **React 19:** https://react.dev
- **Tailwind 4:** https://tailwindcss.com
- **viem:** https://viem.sh
- **wagmi:** https://wagmi.sh
- **Bun:** https://bun.sh
- **Cloudflare Pages:** https://pages.cloudflare.com

---

Write clear, example-driven documentation. Every code snippet should be copy-paste ready. Every command should be tested. Every link should be verified.
