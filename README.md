# Login with Lens

[Lens Protocol](https://www.lens.xyz/docs/) is an open social network that allows users to own their content and connections. Developers can build on the network, leveraging its audience and infrastructure. Users can seamlessly switch between social apps without losing their profiles, content, or connections.

Allowing users to log into Lens with Privy is fully supported and very simple to integrate.

Here's how!

> Building a new app? Check out the [Lens API Examples](https://github.com/lens-protocol/lens-sdk/tree/develop/examples), which includes a basic Lens <> Privy integration.

### 0. Prerequisite

In order to integrate the Privy React SDK with the Lens React SDK, your project must be on:

- a minimum Privy React SDK version of `1.73.1`
- a minimum Lens React SDK version of `2.0.0`

### 1. Installation

The following assumes you have set up the [Lens SDK](https://github.com/lens-protocol/lens-sdk) with your app. If you haven't, start by following the instructions in the [Lens Quickstart](https://www.lens.xyz/docs/getting-started/react-web) guide to get your app set up.

Next, install the Privy React SDK and [wagmi](https://wagmi.sh/) bindings:

```sh
npm install @privy-io/react-auth@latest @privy-io/wagmi
```

When your app integrates Privy alongside Wagmi, you should:

- use Privy to connect external wallets and create embedded wallets
- use wagmi to take read or write actions from a connected wallet

More details on using Privy with Wagmi is available [here](https://docs.privy.io/guide/react/wallets/usage/wagmi).

### 2. Configure the Privy Provider

If following the Lens Quickstart, your existing Provider setup should look similar to this:

```tsx
<WagmiProvider config={wagmiConfig}>
  <QueryClientProvider client={queryClient}>
    <LensProvider config={lensConfig}>
        {/** Your App */}
    </LensProvider>
  </QueryClientProvider>
</WagmiProvider>
```

Lens Protocol runs on Polygon, so the Privy Client configuration must include `polygon` as a supported chain. Your config should look something like:

```tsx
const privyConfig: PrivyClientConfig = {
  defaultChain: polygon,
  supportedChains: [polygon],
  embeddedWallets: {
    createOnLogin: "users-without-wallets",
  },
};
```

**Note:** `PrivyProvider` requires that the `QueryClientProvider` wraps the `WagmiProvider`.

Next, simply wrap the existing Providers with the `PrivyProvider`. Your `Provider` should now look something like this:

```tsx
<PrivyProvider 
  appId={process.env.NEXT_PUBLIC_PRIVY_APP_ID!} 
  config={privyConfig}
>
  <QueryClientProvider client={queryClient}>
    <WagmiProvider config={wagmiConfig}>
      <LensProvider config={lensConfig}>
        {children}
      </LensProvider>
    </WagmiProvider>
  </QueryClientProvider>
</PrivyProvider>
}
```

Here's an example of the full `PrivyProvider` setup with the Lens SDK:

```tsx
import React from "react";
import { http } from "wagmi";
import { polygon, polygonAmoy } from "wagmi/chains";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { LensConfig, LensProvider, development } from "@lens-protocol/react-web";
import { bindings } from "@lens-protocol/wagmi";
import { WagmiProvider, createConfig } from "@privy-io/wagmi";
import { PrivyClientConfig, PrivyProvider } from "@privy-io/react-auth";

const wagmiConfig = createConfig({
  chains: [polygon, polygonAmoy],
  transports: {
    [polygon.id]: http(),
    [polygonAmoy.id]: http(),
  },
});

const privyConfig: PrivyClientConfig = {
  defaultChain: polygonAmoy, // or polygon
  supportedChains: [polygon, polygonAmoy],
  embeddedWallets: {
    createOnLogin: "users-without-wallets",
  },
};

const queryClient = new QueryClient();

const lensConfig: LensConfig = {
  environment: development, // or production
  bindings: bindings(wagmiConfig),
  debug: true,
};

export function Web3Provider({ children }: { children: React.ReactNode }) {
  return (
    <PrivyProvider appId={process.env.NEXT_PUBLIC_PRIVY_APP_ID!} config={privyConfig}>
      <QueryClientProvider client={queryClient}>
        <WagmiProvider config={wagmiConfig}>
          <LensProvider config={lensConfig}>{children}</LensProvider>
        </WagmiProvider>
      </QueryClientProvider>
    </PrivyProvider>
  );
}

```

### 3. Login with Lens

That's it! Your app is now fully setup to interact with the Lens SDK using Privy. To implement Lens login we must first retrieve the list of Profiles that the wallet currently owns or manages. This can be accomplished using the `useProfilesManaged` hook from the Lens SDK:

```tsx
import { useProfilesManaged, useLogin, Profile } from '@lens-protocol/react-web';

const { data: profiles } = useProfilesManaged({
  for: walletAddress,
  includeOwned: true
});
```

Next, use the `useLogin` hook to log in with one of the profiles returned:

```tsx
const { execute: login } = useLogin();
const result = await login({
  address: wallet,
  profileId: profiles[0].id,
});

if (result.isSuccess()) {
  // You can now interact with the Lens SDK hooks that require a logged in user
}
```

For a more complete example, refer to the Authentication instructions from [Lens SDK docs](https://www.lens.xyz/docs/primitives/authentication#profile-login).

### 4. Further

Check out the [Lens SDK examples](https://github.com/lens-protocol/lens-sdk/tree/develop/examples), including `lens-next-privy-app` which includes profile creation and login using the setup from this guide. You can now use the Lens SDK to:

- [log your users in with Lens](https://www.lens.xyz/docs/primitives/authentication#profile-login)
- [create a Lens Profile](https://www.lens.xyz/docs/best-practices/onboarding#crypto-onboarding)
- [update Lens Profile metadata](https://www.lens.xyz/docs/primitives/profile/metadata#update-profile-metadata)
- [create a post](https://www.lens.xyz/docs/primitives/publications/content-creation#creating-a-post)
- [make a post collectable](https://www.lens.xyz/docs/primitives/collect/collectables#collect-actions-simple-collect)
- [react to (like) a publication](https://www.lens.xyz/docs/primitives/publications/reactions#react-to-publication)

and so much more! Examples from the [Lens Docs](https://www.lens.xyz/docs/) can be used without any modification.