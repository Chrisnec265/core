# react-wallet-connect

Connect a Stellar wallet with Freighter, fetch balances, and send a payment — using `sorokit-core` in a React app.

```
Connect wallet → Fetch balances → Build payment → Sign → Submit
```

## Patterns covered

- Wallet connection and disconnection with `FreighterAdapter`
- Loading account balances with `client.account.getBalances`
- Building and signing a payment with `client.transaction.buildPayment`
- Handling `SorokitResult<T>` in component state (no try/catch)
- Loading and error states per operation

## Files

| File | Purpose |
|---|---|
| `src/useSorokit.ts` | Custom hook: client setup, wallet state, account ops, payment |
| `src/App.tsx` | UI: connect button, balance list, payment form, status messages |

## Setup

```bash
# From your own CRA / Vite React project:
npm install sorokit-core @creit.tech/stellar-wallets-kit
```

You need a Stellar Wallets Kit instance. Create it once at app startup:

```ts
import {
  StellarWalletsKit,
  WalletNetwork,
  FREIGHTER_ID,
  FreighterModule,
} from "@creit.tech/stellar-wallets-kit";

const kit = new StellarWalletsKit({
  network: WalletNetwork.TESTNET,
  selectedWalletId: FREIGHTER_ID,
  modules: [new FreighterModule()],
});
```

Then pass it to `useSorokit`:

```ts
const { walletState, connect, balances, sendPayment } = useSorokit(kit, "testnet");
```

## Running

Because this example lives inside the sorokit-core repo, its imports point to
`../../../src/...`. In your own app they become `import { ... } from "sorokit-core"`.

Type-check the example against the SDK source:

```bash
npm run typecheck:examples
```

## First-run walkthrough

`src/WalkthroughOverlay.tsx` adds a dismissible guided overlay that appears on a
user's first visit to the demo application. It guides through four steps: welcome,
wallet connection, network selection, and contract examples.

**Features:**
- Dismissible via the ✕ button or the `Escape` key (session-only — reappears on the next load unless completed)
- Keyboard navigation: `←` / `→` or `Enter` to move between steps, `Escape` to dismiss
- Persistent completion: clicking **Done ✓** writes a flag to `localStorage` so the overlay never shows again
- Falls back gracefully when `localStorage` is unavailable (SSR, private browsing, quota exceeded)

**Usage:**

```tsx
import { WalkthroughOverlay } from "./src/WalkthroughOverlay";

// Mount once at the app root — it self-hides when completed or dismissed.
function App() {
  return (
    <>
      <WalkthroughOverlay />
      {/* rest of your app */}
    </>
  );
}
```

**Using the manager directly (framework-agnostic):**

```ts
import { createWalkthroughManager } from "sorokit-core";

const manager = createWalkthroughManager({
  id: "my-app",
  steps: [
    { id: "welcome", title: "Welcome", description: "Let's get started." },
    { id: "wallet",  title: "Connect Wallet", description: "Connect Freighter." },
  ],
});

const unsubscribe = manager.subscribe((state) => console.log(state));
document.addEventListener("keydown", manager.handleKeyDown);

manager.next();     // advance
manager.complete(); // persist and hide
```
