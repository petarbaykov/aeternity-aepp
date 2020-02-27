<template>
  <div>
    <nav class="flex items-center justify-between flex-wrap bg-white p-6 shadow-lg">
      <div class="flex items-center flex-shrink-0  mr-6">
        <span class="font-semibold text-xl tracking-tight">Sample Aeternity Aepp</span>
      </div>
      <div class="w-full block flex-grow lg:flex lg:items-center lg:w-auto">
        <div class="text-sm lg:flex-grow">
        </div>
        <div v-if="addressResponse">
          <div class="flex  items-center justify-between" >
            <div class="mr-3   text-right">
                <ae-address :value="addressResponse.result" length="short" class="font-bold"/>
                <span class="text-grey-dark">{{ getBalance }} AE</span>
              </div>
            <ae-identicon v-if="addressResponse" :address="addressResponse.result" size="base" />
          </div>
        </div>
      </div>
    </nav>
    <div class="px-2">
      <div class="flex flex-wrap p-4 -mx-2 ">
        <div class="w-full sm:w-5/5 md:w-2/5 lg:w-2/5 xl:w-2/5 mb-4 px-2">
          <div class=" border rounded shadow-lg p-2 text-center">
            <h2 class="mt-4 mb-4 big-title">Wallet info</h2>

            <ae-list>
              <ae-list-item fill="neutral" v-if="balance">
                <div class="flex  items-center justify-between w-full" >
                  <div class="mr-3 text-grey-dark font-bold">Balance</div>
                  <div>{{ getBalance }} AE</div>
                </div>
              </ae-list-item>
              <ae-list-item fill="neutral" v-if="heightResponse">
                <div class="flex  items-center justify-between w-full" >
                  <div class="mr-3 text-grey-dark font-bold">Height</div>
                  <div>{{heightResponse | responseToString}} </div>
                </div>
              </ae-list-item>
              <template v-if="nodeInfoResponse">
                <div v-if="nodeInfoResponse.error" class="bg-green w-full flex flex-row font-mono border border-b">
                  <div class="p-2 w-1/4">
                    NodeInfo error
                  </div>
                  <div class="p-2 w-3/4 bg-grey-lightest break-words">
                    {{nodeInfoResponse.error}}
                  </div>
                </div>
                <ae-list-item fill="neutral" 
                  v-for="(value, name) in nodeInfoResponse.result"
                  v-if="['url', 'name', 'nodeNetworkId', 'version'].includes(name)"
                >
                  <div class="flex  items-center justify-between w-full" >
                    <div class="capitalize mr-3 text-grey-dark font-bold">{{name.replace('nodeNetworkId', 'NetworkId')}}</div>
                    <div> {{value}} </div>
                  </div>
                </ae-list-item>
              </template>
              <ae-list-item fill="neutral" v-if="compilerVersionResponse">
                <div class="flex  items-center justify-between w-full" >
                  <div class="mr-3 text-grey-dark font-bold">Compiler version</div>
                  <div>{{compilerVersionResponse | responseToString}} </div>
                </div>
              </ae-list-item>
            </ae-list>
            <ae-button v-if="addressResponse" face="round" fill="primary" class="mt-4" @click="disconnect">Disconnect</ae-button>
            <ae-button face="round" fill="primary" class="mt-4" @click="signMessage">Sign Message</ae-button>
          </div>
        </div>
        <div class="w-full sm:w-5/5 md:w-3/5 lg:w-3/5 xl:w-3/5 mb-4 px-2 ">
          <div class=" border rounded shadow-lg p-2 text-center">
            <h2 class="mt-4 mb-4 p-2 big-title">Spend tokens</h2>
            <div  v-if="spendResponse" class="bg-red-lightest border border-red-light text-red-darkest px-4 py-3 rounded relative text-left mb-4 mt-4" role="alert">
              <strong class="font-bold">Send result!</strong>
              <span class="block sm:inline break-words whitespace-pre-wrap">{{ spendResponse | responseToFormattedJSON }}</span>
            </div>
            <div class="bg-grey-lightest w-full flex flex-row font-mono">
              <div class="p-2 w-1/4">
                Recipient address
              </div>
              <div class="p-2 w-3/4 bg-white break-words">
                <input
                  class="bg-black text-white border-b border-black p-2 w-full"
                  v-model="spendTo"
                  placeholder="ak_..."
                />
              </div>
            </div>
            <div class="bg-grey-lightest w-full flex flex-row font-mono">
              <div class="p-2 w-1/4">
                Tokens amount
              </div>
              <div class="p-2 w-3/4 bg-white break-words">
                <input
                  class="bg-black text-white border-b border-black p-2 w-full"
                  v-model="spendAmount"
                />
              </div>
            </div>
            <div class="bg-grey-lightest w-full flex flex-row font-mono">
              <div class="p-2 w-1/4">
                Payload
              </div>
              <div class="p-2 w-3/4 bg-white break-words">
                <input
                  class="bg-black text-white border-b border-black p-2 w-full"
                  v-model="spendPayload"
                />
              </div>
            </div>

            <ae-button v-if="client" face="round" fill="primary" class="mt-4" @click="spend">Spend</ae-button>
          </div>
        </div>
      </div>
    </div>
    <div class="w-full p-4 flex flex-col">
      <div class="border mt-4 rounded overflow-hidden shadow-lg p-4 text-center" v-if="contractMode == 'compile'">
        <h2 class="mt-4 mb-4 p-2 big-title">Compile Contract</h2>
        <div class="bg-grey-lightest w-full flex flex-row font-mono">
          <div class="p-2 w-1/4">
            Contract Code
          </div>
          <div class="p-2 w-3/4 bg-white">
            <textarea class="bg-black text-white border-b border-black p-2 w-full h-64" v-model="contractCode" placeholder="contact code"/>
          </div>
        </div>

        <ae-button v-if="client" face="round" fill="primary" class="mt-4" @click="compile">Compile</ae-button>
      </div>

      <div v-if="compileBytecodeResponse && contractMode == 'compiled'" class="border mt-4 mb-8 rounded overflow-hidden shadow-lg p-4 text-center">
        <h2 class="mt-4 mb-4 p-2 big-title">Deploy Contract</h2>
        <div class="bg-grey-lightest w-full flex flex-row font-mono">
          <div class="p-2 w-1/4">
            Compiled Code
          </div>
          <div class="p-2 w-3/4 bg-white">
            <div class="bg-black text-white border-b border-black p-2 w-full h-64 break-words" >
              {{ compileBytecodeResponse | responseToString }}
            </div>
          </div>
        </div>
        <ae-button v-if="compileBytecodeResponse && compileBytecodeResponse.result" face="round" fill="secondary" class="mt-4" @click="contractMode = 'compile'">Compile</ae-button>
        <ae-button v-if="compileBytecodeResponse && compileBytecodeResponse.result" face="round" fill="primary" class="mt-4" @click="deploy">Deploy</ae-button>

        <ae-button v-if="deployResponse && deployResponse.result" face="round" fill="primary" class="mt-4" @click="call">Call</ae-button>
      </div>

      <div  class="border mt-4 mb-8 rounded overflow-hidden shadow-lg p-4 text-center">
        <h2 class="mt-4 mb-4 p-2 big-title">Send Tip</h2>

      </div>

      <div v-if="deployResponse && contractMode == 'deployed'" class="border mt-4 mb-8 rounded">
        <div class="bg-green w-full flex flex-row font-mono border border-b">
          <div class="p-2 w-1/4">
            Deployed Contract
          </div>
          <div
            class="p-2 w-3/4 bg-grey-lightest break-words whitespace-pre-wrap"
          >{{ deployResponse | responseToFormattedJSON }}</div>
        </div>
      </div>
      

      <div v-if="callResponse" class="border mt-4 mb-8 rounded">
        <div class="bg-green w-full flex flex-row font-mono border border-b">
          <div class="p-2 w-1/4">
            Call Result
          </div>
          <div
            class="p-2 w-3/4 bg-grey-lightest break-words whitespace-pre-wrap"
          >{{ callResponse | responseToFormattedJSON }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  //  is a webpack alias present in webpack.config.js
  import { RpcAepp } from '@aeternity/aepp-sdk/es'
  import Detector from '@aeternity/aepp-sdk/es/utils/aepp-wallet-communication/wallet-detector'
  import BrowserWindowMessageConnection from '@aeternity/aepp-sdk/es/utils/aepp-wallet-communication/connection/browser-window-message'
  import Node from '@aeternity/aepp-sdk/es/node'

  // Send wallet connection info to Aepp throug content script
  const networks = {
    'ae_mainnet': {
      NODE_URL: 'https://mainnet.aeternal.io',
      NODE_INTERNAL_URL: 'https://mainnet.aeternal.io',
      COMPILER_URL: 'https://compiler.aepps.com'
    },
    'ae_uat': {
      NODE_URL: 'https://testnet.aeternal.io',
      NODE_INTERNAL_URL: 'https://testnet.aeternal.io',
      COMPILER_URL: 'https://latest.compiler.aepps.com'
    }
  }

  const errorAsField = async fn => {
    try {
      return { result: await fn }
    } catch (error) {
      return { error }
    }
  }

  export default {
    data () {
      return {
        runningInFrame: window.parent !== window,
        client: null,
        addressResponse: null,
        heightResponse: null,
        compilerVersionResponse: null,
        nodeInfoResponse: null,
        spendTo: null,
        spendAmount: null,
        spendPayload: null,
        spendResponse: null,
        spendResult: null,
        spendError: null,
        balance: null,
        contractCode: `@compiler >= 4

contract Example =
  record state = { b: bytes(32) }
  entrypoint init() : state = { b = #aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa }
  entrypoint get_bytes() : bytes(32) = state.b
  stateful entrypoint set_bytes(x: bytes(32)) = put(state{ b = x })`,
        byteCode: null,
        compileBytecodeResponse: null,
        contractInitState: [],
        deployResponse: null,
        callResponse: null,
        walletName: null,
        onAccount: null,
        accounts: [],
        contractMode:'compile'
      }
    },
    filters: {
      responseToString: response => `${response.error ? 'Error: ' : ''}${response.result || response.error}`,
      responseToFormattedJSON: response => response.error
        ? `Error: ${response.error}`
        : JSON.stringify(response.result, null, 4),
    },
    computed: { 
      getBalance() {
        return this.convertToAE(this.balance)
      }
    },
    methods: {
      convertToAE  (balance)  {
          return +(balance / 10 ** 18).toFixed(7);
      },
      async signMessage() {
        const messageSig  = await this.client.signMessage('test');
        console.log("signed message => ", messageSig)
        const isValid = await this.client.verifyMessage('test', messageSig)
        console.log("verified", isValid)
      },
      async spend () {
        const onAccount = Object.keys(this.accounts.address.connected)[0]
        this.spendResponse = await errorAsField(this.client.spend(
          this.spendAmount,
          this.spendTo, {
            payload: this.spendPayload,
            // fee: 1
            // onAccount: onAccount
          }
        ));
      },
      async compile () {
        this.compileBytecodeResponse = await errorAsField(
          (await this.client.contractCompile(this.contractCode)).bytecode
        );
        if(this.compileBytecodeResponse) {
          this.contractMode = 'compiled'
        }
      },
      async deploy () {
        this.deployResponse = await errorAsField(this.client.contractDeploy(
          this.compileBytecodeResponse.result, this.contractCode, this.contractInitState
        ));
      },
      async call (code, method = 'get_bytes', returnType = 'bytes(32)', args = []) {
        let arr = "#aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa"
        // args = [arr]
        this.callResponse = await errorAsField((async () => {
          const result = await this.client.contractCall(
            this.contractCode, this.deployResponse.result.address, method,  args
          )
          return Object.assign(
            result,
            { decodedRes: await result.decode() }
          )
        })())
      },
      async disconnect() {
        await this.client.disconnectWallet()
        this.walletName = null
        this.pub = null
        this.balance = null
        this.addressResponse = null
        this.scanForWallets()
      },
      async getReverseWindow() {
        const iframe = document.createElement('iframe')
        // iframe.src = prompt('Enter wallet URL', 'http://localhost:9000')
        iframe.src = ''
        iframe.style.display = 'none'
        document.body.appendChild(iframe)
        return iframe.contentWindow
      },
      async connectToWallet (wallet) {
        await this.client.connectToWallet(await wallet.getConnection())
        this.accounts = await this.client.subscribeAddress('subscribe', 'connected')
        this.pub = await this.client.address()
        this.onAccount = this.pub
        this.balance = await this.client.getBalance(this.pub)
        this.walletName = this.client.rpcClient.info.name
        this.addressResponse = await errorAsField(this.client.address())
        this.heightResponse = await errorAsField(this.client.height())
        this.nodeInfoResponse = await errorAsField(this.client.getNodeInfo())
        this.compilerVersionResponse = await errorAsField(this.client.getCompilerVersion())
      },
      async scanForWallets () {
        try {
          const handleWallets = async function ({ wallets, newWallet }) {
            newWallet = newWallet || Object.values(wallets)[0]
            // if (confirm(`Do you want to connect to wallet ${newWallet.name}`)) {
              
            // }
            this.detector.stopScan()
            
            await this.connectToWallet(newWallet)
            // let addr = await this.client.askAddresses()
          }

          const scannerConnection = await BrowserWindowMessageConnection({
            connectionInfo: { id: 'spy' }
          })
          this.detector = await Detector({ connection: scannerConnection })
          this.detector.scan(handleWallets.bind(this))
        } catch(e) {
          console.log(e)
        }
        
      }
    },
    async created () {
      window !== window.parent || await this.getReverseWindow()
      const node = await Node({ url: networks.ae_mainnet.NODE_URL, internalUrl:  networks.ae_mainnet.NODE_INTERNAL_URL })
      this.client = await RpcAepp({
        name: 'AEPP',
        nodes: [
            { name: 'ae_mainnet', instance: node },
        ],
        compilerUrl: networks.ae_mainnet.COMPILER_URL,
        onNetworkChange (params) {
          console.log(params)
          if (this.getNetworkId() !== params.networkId) alert(`Connected network ${this.getNetworkId()} is not supported with wallet network ${params.networkId}`)
        },
        onAddressChange:  async (addresses) => {
          console.log(addresses)
          this.pub = await this.client.address()
          this.balance = await this.client.balance(this.pub).catch(e => '0')
          this.addressResponse = await errorAsField(this.client.address())
        },
        onDisconnect (a) {
          console.log("disconnect")
          console.log(a)
        }
      })
      this.height = await this.client.height()

      // Start looking for wallets
      await this.scanForWallets()
    }
  }
</script>


<style scoped>
.big-title {
  font-size: 2.4375rem;
  font-weight: 700;
  line-height: 4.0625rem;
  color: #203040;
  margin: 1.5rem 0;
}
.ae-identicon {
    box-shadow: 0 0 0 2px #ff0d6a;
    border: .125rem solid transparent;
    width: 2.625rem !important;
    height: 2.625rem !important;
}
.ae-address {
  font-weight: bold;
}
.ae-address li {
  list-style: none;
}
.ae-button.round {
  height: 45px;
  padding: 0 3rem;
}
</style>