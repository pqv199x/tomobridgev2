<template>
    <div class="container">
        <AddressInfo/>
        <b-container
            class="container-medium">
            <div class="open-product text-center">
                <h1 class="title-tmp-large">
                    CONVERTING FORM
                </h1>
                <p class="txt-dec-tem-2 font-weight-bold">
                    TomoChain wrapped {{ fromWrapSelected.symbol }}
                    <span class="txt-dec">to</span>
                    {{ fromWrapSelected.symbol.toLowerCase() !== 'eth' ? `ERC-20 ${fromWrapSelected.symbol}` : 'native ETH' }}
                </p>
            </div>

            <b-row class="wrapbox__row mb-lg-4 mt-4">
                <b-col cols="7">
                    <b-form-group
                        class="mb-4 font-weight-bold"
                        label="Asset"
                        label-for="amount">
                        <multiselect
                            id="fromwrap-select"
                            v-model="fromWrapSelected"
                            :options="fromData"
                            :custom-label="customLabel"
                            :show-labels="false"
                            :allow-empty="false"
                            :close-on-select="true"
                            track-by="name"
                            @select="selectToken">
                            <template
                                slot="singleLabel"
                                slot-scope="props">
                                <div class="d-flex justify-content-between align-items-center">
                                    <div class="d-flex align-items-center">
                                        <img
                                            v-if="props.option.image"
                                            :src="props.option.image"
                                            class="multiselect__img">
                                        <span class="multiselect__name">{{ props.option.name }}</span>
                                        <i
                                            v-if="(verifiedList.indexOf((props.option.wrapperAddress || '').toLowerCase()) > 0)
                                                || props.option.name === 'BTC' || props.option.name === 'ETH'"
                                            class="tb-check-circle-o multiselect_greentick ml-2"/>
                                    </div>
                                    <div
                                        v-if="tokenBalanceToFixed > 0"
                                        class="mr-4">{{ tokenBalanceToFixed }}</div>
                                </div>
                            </template>
                            <template
                                slot="option"
                                slot-scope="props">
                                <div class="d-flex align-items-center">
                                    <img
                                        v-if="props.option.image"
                                        :src="props.option.image"
                                        class="multiselect__img">
                                    <span class="multiselect__name">{{ props.option.name }}</span>
                                    <i
                                        v-if="(verifiedList.indexOf((props.option.wrapperAddress || '').toLowerCase()) > 0)
                                            || props.option.name === 'BTC' || props.option.name === 'ETH'"
                                        class="tb-check-circle-o multiselect_greentick ml-2"/>
                                </div>
                            </template>
                        </multiselect>
                    </b-form-group>
                </b-col>
                <b-col cols="5">
                    <b-form-group
                        class="mb-4 font-weight-bold"
                        label="Amount"
                        label-for="amount">
                        <b-form-input
                            v-model="amount"
                            placeholder="Withdrawal amount"
                            type="text"/>
                        <b-button
                            class="token-max"
                            variant="success"
                            @click="maxToken">
                            Max
                        </b-button>
                    </b-form-group>
                </b-col>
                <b-col
                    v-if="fromWrapSelected.symbol === 'ETH'"
                    cols="12"
                    class="blink_me mb-2 text-center">
                    We’re migrating old token ETH to new token, go migrate
                    <a
                        href="https://migrate.tomochain.com"
                        target="_blank"
                        class="text-blue">here</a>
                </b-col>
                <b-col cols="12">
                    <b-form-group
                        class="mb-4 font-weight-bold"
                        label="Receiving Address"
                        label-for="recAddress">
                        <b-form-input
                            v-model="recAddress"
                            type="text"
                            class="pr-8"
                            placeholder="Please use only Ethereum network address"/>
                        <b-button
                            class="add-address"
                            variant="success"
                            @click="useAddress">
                            Address
                        </b-button>
                    </b-form-group>
                </b-col>
            </b-row>

            <div class="box-more-infor">
                <div
                    v-if="fromWrapSelected.symbol !== 'BTC' && fromWrapSelected.symbol !== 'ETH'">
                    <div class="txt-cdgt ">
                        View Token address on
                        <a
                            :href="etherScanUrl"
                            target="_blank">Etherscan</a>
                    </div>
                </div>
                <div class="content-fee">
                    <div class="pr-2">Estimate transaction fee:</div>
                    <div class="infor-fee mt-2">
                        <!-- <div
                            v-if="!isApproved"
                            class="px-3">
                            Approve: 1 TOMO
                        </div> -->
                        <div class="px-3">Swap: {{ fee }} ETH</div>
                        <div class="px-3">
                            Total: {{ fee }} ETH
                        </div>
                    </div>
                </div>
                <div class="txt-confirm">
                    <p class="font-weight-bold mb-2">Please confirm the following:</p>
                    <b-form-checkbox
                        v-model="agreeEx"
                        class="mr-1 m1 light-h">
                        The receiving address is NOT created on a centralized exchange (e.g. Binance)
                    </b-form-checkbox>
                    <b-form-checkbox
                        v-model="agreePk"
                        class="mr-1 m1 light-h">
                        I have the Private Key/Mnemonics for the receiving address entered above
                    </b-form-checkbox>
                    <b-form-checkbox
                        v-model="agreeAll"
                        class="mr-1 m1 light-h">
                        I have doubled checked that the receiving address is correct
                    </b-form-checkbox>
                    <div
                        v-if="allChecked"
                        class="text-error">
                        <b-icon
                            class="mr-1 m1 light-h"
                            icon="exclamation-circle"
                            font-scale="1"/>
                        Required fields
                    </div>
                </div>
            </div>
            <div class="d-flex mt-4">
                <b-button
                    class="st-back w-100 mr-2"
                    @click="back">
                    Back
                </b-button>
                <b-button
                    :disabled="!agreeAll || !agreeEx || !agreeEx || !amount || !recAddress"
                    class="st-next w-100 ml-2"
                    @click="unwrapToken">
                    Next
                </b-button>
            </div>
        </b-container>
        <div
            :class="(loading ? 'tomo-loading' : '')"/>
    </div>
</template>

<script>
import urljoin from 'url-join'
import BigNumber from 'bignumber.js'
import Multiselect from 'vue-multiselect'
import WAValidator from 'wallet-address-validator'
import AddressInfo from './../Address.vue'
export default {
    name: 'App',
    components: {
        Multiselect,
        AddressInfo
    },
    data () {
        return {
            verifiedList: [],
            amount: '',
            isApproved: false,
            agreeEx: false,
            agreePk: false,
            agreeAll: false,
            allChecked: false,
            fromWrapSelected: {},
            recAddress: '',
            fromData: [
                { name: 'Ethereum', symbol: 'ETH', image: '' }
            ],
            config: this.$store.state.config,
            etherScanUrl: '',
            tokenBalanceToFixed: 0,
            tokenBalance: '',
            isAddress: true,
            tomoFeeMode: false,
            fee: 0,
            contract: '',
            contractAddress: '',
            isNativeToken: false,
            tomoIds: [88, 89, 99],
            loading: false
        }
    },
    computed: {
        address: {
            get () {
                return this.$store.getters.address
            },
            set () {}
        },
        network: {
            get () {
                return this.$store.getters.network
            },
            set () {}
        },
        mobileCheck: () => {
            const isAndroid = navigator.userAgent.match(/Android/i)
            const isIOS = navigator.userAgent.match(/iPhone|iPad|iPod/i)
            return (isAndroid || isIOS)
        }
    },
    async updated () {
    },
    destroyed () { },
    created: async function () {
        this.$store.state.redirectTo = ''
        if (!this.$store.state.address) {
            this.$router.push({
                path: '/select'
            })
        } else {
            this.fromData = this.config.swapCoin || []
            this.fromWrapSelected = this.fromData[0]
            this.loading = true

            this.etherScanUrl = urljoin(
                this.fromWrapSelected.explorerUrl,
                'token',
                this.fromWrapSelected.tokenAddress
            )

            const { contract, contractAddress } = await this.getContract()
            this.contract = contract
            this.contractAddress = contractAddress
            this.getTokenBalance(this.fromWrapSelected)
            this.getWithdrawFee()
            // this.contract.methods.TOMO_FEE_MODE.call()
            //     .then(data => {
            //         this.tomoFeeMode = data
            //         // this.estimateGasClaim(this.fromWrapSelected)
            //         this.getWithdrawFee()
            //     }).catch(error => {
            //         this.loading = false
            //         this.$toasted.show(error, { type: 'error' })
            //     })
            this.loading = false
        }
    },
    methods: {
        customLabel ({ name }) {
            return `${name}`
        },
        back () {
            this.$router.go(-1)
        },
        useAddress () {
            this.recAddress = this.address
        },
        maxToken () {
            const token = this.fromWrapSelected
            this.amount = new BigNumber(this.tokenBalance || 0).div(10 ** token.decimals).toString()
        },
        async selectToken (token) {
            this.fromWrapSelected = token
            this.etherScanUrl = urljoin(
                token.explorerUrl,
                'token',
                token.tokenAddress
            )
            await this.getContract(token)
            this.getTokenBalance(token)
            // this.tomoFeeMode = await this.contract.methods.TOMO_FEE_MODE.call()
            // await this.estimateGasClaim(token)
            await this.getWithdrawFee(token)
        },
        getContract (token = this.fromWrapSelected) {
            this.contract = new this.web3.eth.Contract(
                // this.WrapperAbi.abi,
                this.TomoBridgeTokenAbi.abi,
                token.wrapperAddress.toLowerCase()
            )
            return { contract: this.contract, contractAddress: token.wrapperAddress }
        },
        async getTokenBalance (token) {
            const contract = new this.web3.eth.Contract(
                this.TomoBridgeTokenAbi.abi,
                token.wrapperAddress
            )
            const balanceBN = await contract.methods.balanceOf(this.address).call()
            this.tokenBalance = balanceBN
            this.tokenBalanceToFixed = new BigNumber(balanceBN).div(10 ** token.decimals).toFixed(5)
        },
        async getWithdrawFee (token = this.fromWrapSelected) {
            // let feeBN
            const config = this.config
            const gasPrice = await this.web3Eth.eth.getGasPrice()
            if (token.symbol.toLowerCase() !== 'eth') {
                this.fee = new BigNumber(config.etherChain.tokenClaimGas).multipliedBy(gasPrice).div(10 ** 18).toString(10)
            } else {
                this.fee = new BigNumber(config.etherChain.ethClaimGas).multipliedBy(gasPrice).div(10 ** 18).toString(10)
            }
            // if (this.tomoFeeMode) {
            //     feeBN = await this.contract.methods.WITHDRAW_FEE_TOMO().call()
            //     this.fee = new BigNumber(feeBN).div(10 ** 18).toString(10)
            // } else {
            //     feeBN = await this.contract.methods.WITHDRAW_FEE().call()
            //     this.fee = new BigNumber(feeBN).div(10 ** token.decimals).toString(10)
            // }
        },
        isValidAddresss () {
            const address = this.recAddress
            const config = this.config
            // Check network
            const network = config.blockchain.networkId === 88 ? 'prod' : 'testnet'
            switch (this.fromWrapSelected.network.toLowerCase()) {
            case 'bitcoin':
                return WAValidator.validate(address, 'BTC', network)
            case 'ethereum':
                return this.web3.utils.isAddress(address)
            default:
                return false
            }
        },
        checkMinimumWithdrawAmount () {
            // const coin = this.fromWrapSelected
            // if (new BigNumber(this.amount || 0).isLessThan(new BigNumber(coin.minimumWithdrawal))) {
            //     return false
            // }
            if (new BigNumber(this.amount || 0).isLessThanOrEqualTo(0)) {
                return false
            }
            return true
        },
        unwrapToken () {
            if (this.tomoIds.indexOf(this.network.chainId) > -1) {
                this.isAddress = this.isValidAddresss()
                if (this.isAddress) {
                    if (!this.agreeAll || !this.agreeEx || !this.agreePk) {
                        this.$toasted.show('Confirmation required', { type: 'error' })
                        // this.allChecked = true
                    } else if (!this.checkMinimumWithdrawAmount()) {
                        this.$toasted.show(`Withdraw amount must be greater than 0`)
                    } else {
                        this.$router.push({
                            name: 'UnwrapExecution',
                            params: {
                                recAddress: this.recAddress,
                                amount: this.amount,
                                fromWrapSelected: this.fromWrapSelected
                            }
                        })
                    }
                } else {
                    this.$toasted.show('Invalid recipient address', { type: 'error' })
                }
            } else { this.$toasted.show('Set your Metamask network to TomoChain', { type: 'error' }) }
        }
    }
}
</script>
