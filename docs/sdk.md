---
nav_order: 5
title: CIP SDK
layout: default
permalink: cip-sdk
---

# Canto Identity Protocol SDK Guide

The purpose of this guide is to get you up and running with the CIP-SDK in your application in just a few minutes, perfect for retrieving user canto identity protocol data. For the sake of simplicity, we will assume that you are building a web application with React. The SDK will also work in node as well. 

# Prerequisite
We will be using [ethers.js](https://docs.ethers.org/v5/) [wagmi](https://wagmi.sh/) and [Rainbowkit](https://www.rainbowkit.com) in Next.js to interact with the CIP SDK and Canto.

# Install
```
npm install wagmi viem @rainbow-me/rainbowkit ethers
```

# Example
To configure wagmi, create a file called providers.ts:

{% highlight javascript%}
import { getDefaultWallets } from '@rainbow-me/rainbowkit';
import { configureChains, createConfig, WagmiConfig } from 'wagmi';

export const { chains, publicClient, webSocketPublicClient } = configureChains(
  [canto],
  [publicProvider()]
);

const { connectors } = getDefaultWallets({
  appName: 'RainbowKit App',
  projectId: '54869hvf6h34a0fe6e04561318b5ec99', <-- your projectId
  chains,
});

export const wagmiConfig = createConfig({
  autoConnect: true,
  connectors,
  publicClient,
  webSocketPublicClient,
});
{% endhighlight %}

Then in your main _app.tsx file, wrap your app in the wagmi and Rainbowkit providers:

{% highlight jsx%}
import { RainbowKitProvider } from '@rainbow-me/rainbowkit'
import '@rainbow-me/rainbowkit/styles.css'
import type { AppProps } from 'next/app'
import { useEffect, useState } from 'react'
import { WagmiConfig } from 'wagmi'

import { chains, wagmiConfig } from './providers'

export default function App({ Component, pageProps }: AppProps) {
  const [isMounted, setIsMounted] = useState(false)
  useEffect(() => setIsMounted(true), [])

  return (
    <WagmiConfig config={wagmiConfig}>
      <RainbowKitProvider chains={chains}>
        {isMounted && <Component {...pageProps} />}
      </RainbowKitProvider>
    </WagmiConfig>
  )
}
{% endhighlight %}

Now we have an app that is ready to interact with the CIP sdk. 


# Retrieving Namespace, Bio and PFP.

Easiest way to retrieve a user namepsace, brio and pfp data is to use the getNamespaceByAddress(), getBioByAddress(), getPfpByAddress() methods. 

{% highlight jsx%}
import React, { useState } from 'react';
import { ethers } from 'ethers';
import { CIP } from 'cip-sdk';
import { chains } from './_app';

// pfp data shape
type ProfilePictureInfo = {
  src: string;
  alt: string;
};

//namespace data shape
type NameSpace = {
  displayName: string;
  baseName: string;
};

// select the right chain from wagmi
const cantoChain = chains[0];
const rpcUrl = cantoChain.rpcUrls.default.http[0];

// initialize provider and CIP instance
const provider = new ethers.providers.JsonRpcProvider(rpcUrl);
const cipInstance = new CIP(provider);

//helper function to handle nft picture data. 
const transformUri = (uri: string) => {
  if (!uri) return '';
  else if (uri.includes('ipfs://')) {
    return uri.replace('ipfs://', 'https://dweb.link/ipfs/');
  } else if (uri.includes('arweave://')) {
    return uri.replace('arweave://', 'https://arweave.net/');
  }
  return uri;
};

function Temp() {
  const [isLoading, setIsLoading] = useState(false);
  const [namespace, setNamespace] = useState<NameSpace>({
    displayName: '',
    baseName: '',
  });
  const [bio, setBio] = useState('');
  const [pfp, setPfp] = useState<ProfilePictureInfo>({
    src: '',
    alt: '',
  });

  const handleCall = async () => {
    try {
      setIsLoading(true);
      const addr = '0x7a52439b167eab8382677fdba54c2e2b616bd169'; // <------CIP user address
      
      const namespace = await cipInstance.getNamespaceByAddress(addr); // <---- CIP sdk methods
      const bio = await cipInstance.getBioByAddress(addr); // <---- CIP sdk methods
      const pfp = await cipInstance.getPfpByAddress(addr); // <---- CIP sdk methods
  
      setNamespace({
        displayName: namespace.displayName,
        baseName: namespace.baseName,
      });
  
      setBio(bio);
  
      setPfp({
        src: transformUri(pfp.src),
        alt: pfp.alt,
      });
  
    } catch (error) {
      console.log(error);
    } finally {
      setIsLoading(false);
    }
  };
  

  return (
    <div className='mt-20 flex justify-center align-middle'>
      <div className=''>
        {/* ID CARD */}
        <div
          id='shared-id'
          className='m-auto flex max-w-xl flex-col bg-idCard py-3 px-3'
        >
          <div className='mr-4 flex-shrink-0'></div>
          <div className='flex h-[345px] w-[345px] flex-col items-center justify-between bg-white p-3'>
            <img
              src={pfp.src}
              height={90}
              width={135}
              alt={pfp.alt}
              className='rounded-full'
            />
            <h1 className='mb-1 mt-4 break-words text-2xl'>{namespace.displayName}</h1>
            <h2 className='text-md mb-2 text-gray-400'>{namespace.baseName}</h2>
            <p className='flex-grow text-sm'>{bio}</p>
          </div>
          <div className='mt-1.5 flex justify-between'>
            <div className='mt-1 text-xl font-light text-white'>#14</div>
            <div className='text-md font-light text-white'>
            </div>
            <div
              onClick={handleCall}
              className='items-center bg-slate-600 text-white'
            >
              <button className='rounded bg-slate-500 py-2 px-4 font-bold text-white hover:bg-slate-700'>
                {isLoading ? 'loading' : 'fetch data'}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  );
}

export default Temp;
{% endhighlight %}

# NPM information
For additional information on the cip-sdk, visit the [package website](https://www.npmjs.com/package/cip-sdk).