<script setup lang="ts">
import { Ref, ref, computed } from 'vue'
import { EthereumProvider } from '@walletconnect/ethereum-provider'

const account: Ref<null | string> = ref(null)
const chain: Ref<null | string> = ref(null)

const activeChain = computed(() => {
  if (chain.value === '0x1') {
    return 'ETH'
  } else if (chain.value === '0xa4ec') {
    return 'CELO'
  } else {
    return null
  }
})

const provider = await EthereumProvider.init({
  projectId: 'foo',
  chains: [1],
  optionalChains: [42220],
  showQrModal: true,
})

provider.on('connect', (obj) => {
  console.log('connected', obj)
})

provider.on('chainChanged', (chainId) => {
  console.log(chainId)
  chain.value = chainId
})

const handleConnect = async () => {
  try {
    await provider.connect()
    account.value = provider.accounts[0]
  } catch (error) {
    console.log(error)
  }
}

const handleDisconnect = async () => {
  try {
    await provider.disconnect()
    account.value = null
    chain.value = null
  } catch (error) {
    console.log(error)
  }
}

const handleNetworkChangeCelo = async () => {
  try {
    await provider.request({
      method: 'wallet_switchEthereumChain',
      params: [
        {
          chainId: '0xa4ec',
        },
      ],
    })
  } catch (error) {
    console.log(error)
  }
}

const handleNetworkChangeEth = async () => {
  try {
    await provider.request({
      method: 'wallet_switchEthereumChain',
      params: [
        {
          chainId: '0x1',
        },
      ],
    })
  } catch (error) {
    console.log(error)
  }
}

const handleSign = async () => {
  const msg = 'Hello message!';
  try {
    const signed = await provider.request({
      method: 'personal_sign',
      params: [msg, account.value],
    })
    console.log(signed)
  } catch (error) {
    console.log(error)
  }
}
</script>

<template>
  <div>
    <h1>Walletconnect + Metamask</h1>
    <button v-if="!account" type="button" @click="handleConnect">Conectar</button>
    <button v-else type="button" @click="handleDisconnect">Desconectar</button>
    
    <div v-if="account">
      <p> Active network: {{ activeChain }}</p>
      <p>{{ account }}</p>
    </div>
    
    <div v-if="account">
      <button v-if="activeChain === 'CELO'" type="button" @click="handleNetworkChangeEth">Change to ETH</button>
      <button v-if="activeChain === 'ETH'" type="button" @click="handleNetworkChangeCelo">Change to CELO</button>
    </div>
    
    <div v-if="account" style="margin-top: 1rem">
      <button type="button" @click="handleSign">Sign Message</button>
    </div>
  </div>
</template>
