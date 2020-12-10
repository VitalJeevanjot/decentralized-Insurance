<template>
  <q-page style="background-color: #21222C">
    <div class="row justify-center q-pt-md">
      <q-tabs
        v-model="tab"
        class="text-grey-2"
        indicator-color="teal-9"
      >
        <q-tab name="policy_maker" label="Buy" />
        <q-tab name="agent" label="Middleman" />
        <q-tab name="clients" label="Users" />
      </q-tabs>

    </div>
    <div class="row justify-center text-center q-pt-md">
      <q-card dark flat class="my-card" style="min-width: 300px; max-width: 90vw; background-color: #292B35;">
        <q-card-section>

          <q-tab-panels v-model="tab" animated style="background-color: #292B35; color: #55565D">
              <q-tab-panel name="policy_maker" >
                <div class="row justify-center text-grey-2 text-h5">Policy Maker</div>
                <div class="row justify-center">
                  Get policy details
                </div>
              </q-tab-panel>

              <q-tab-panel name="agent">
                <div class="text-h6">Alarms</div>
                Lorem ipsum dolor sit amet consectetur adipisicing elit.
              </q-tab-panel>

              <q-tab-panel name="clients">
                <div class="text-h6">Movies</div>
                Lorem ipsum dolor sit amet consectetur adipisicing elit.
              </q-tab-panel>
            </q-tab-panels>
        </q-card-section>
        <q-card-section>
          <q-input label-color="grey-2" input-style="color:#F5F5F5" color="teal-9" filled v-model="policy_id" label="User address">
            <template v-if="policy_id" v-slot:append>
              <q-icon name="cancel" @click.stop="policy_id = null" class="text-grey-2 cursor-pointer" />
            </template>
          </q-input>
        </q-card-section>
        <q-card-section>
          <q-btn color="teal-6" label="Query" @click="queryPolicy"/>
        </q-card-section>
    </q-card>
    </div>
  </q-page>
</template>

<script>
export default {
  name: 'PageIndex',
  data () {
    return {
      client: null,
      testnet_url: 'https://testnet.aeternity.io',
      mainnet_url: 'https://mainnet.aeternity.io',
      compiler_url: 'https://compiler.aepps.com',
      rpc_client: null,
      network_id: null,
      contract: null,
      tab: 'policy_maker',
      policy_id: null
    }
  },
  methods: {
    async queryPolicy () {
      if (this.policy_id == null) {
        this.$q.notify('Please add policy id first')
        return
      }
      try {
        const details = await this.contract.methods.get_policy_details_of.get(this.policy_id)
        console.log(details.decodedResult)
      } catch (e) {
        console.log('hi')
      }
    },
    async initProvider () {
      try {
        const networkId = (await this.client.getNodeInfo()).nodeNetworkId
        this.network_id = networkId
        console.log(this.network_id)
        this.contract = await this.client.getContractInstance(this.$contract_code, { contractAddress: this.$contract_address })
        return true
      } catch (e) {
        console.error(e)
        return false
      }
    },
    async scanForWallets () {
      const scannerConnection = await this.$browser_message({
        connectionInfo: { id: 'spy' }
      })
      const detector = await this.$detector({ connection: scannerConnection })
      const handleWallets = async ({ wallets, newWallet }) => {
        await detector.stopScan()
        const connected = await this.rpc_client.connectToWallet(await wallets[Object.keys(wallets)[0]].getConnection())
        await this.rpc_client.selectNode(connected.networkId) // connected.networkId needs to be defined as node in RpcAepp
        await this.rpc_client.subscribeAddress('subscribe', 'current')
        this.client = this.rpc_client
        await this.initProvider()
      }
      detector.scan(handleWallets)
    }
  },
  async mounted () {
    console.log('-----------------')
    this.rpc_client = await this.$rpc_aepp({
      name: 'AEPP',
      nodes: [
        { name: 'ae_mainnet', instance: await this.$node({ url: this.mainnet_url }) },
        { name: 'ae_uat', instance: await this.$node({ url: this.testnet_url }) }
      ],
      compilerUrl: this.compiler_url,
      onNetworkChange (params) {
        console.log(params)
        // aeternity.initProvider();
      },
      onAddressChange (addresses) {
        // if (!addresses.current[aeternity.address]) {
        //   console.log('address changed')
        // }
      }
    })
    this.scanForWallets()
  }
}
</script>
