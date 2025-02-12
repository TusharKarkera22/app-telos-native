<script>
import { mapActions } from 'vuex';
import { Notify } from 'quasar';

export default {
    name: 'DevelopersPage',
    data() {
        return {
            form: {
                send_to: null,
                account_name: null,
                owner_key: null,
                active_key: null,
            },
            transactionId: null,
            submitting: false,
        };
    },
    computed: {
      isCreateAccountButtonDisabled() {
        return (
            !this.form.account_name ||
            !this.form.owner_key ||
            !this.form.active_key ||
            this.submitting
        );
      },
      isAnyInputInvalid() {
        return (
          !this.form.account_name ||
          (!/^EOS[0-9A-Za-z]{50}$|^PUB_K1_[0-9A-Za-z]{50}$/i.test(
            this.form.owner_key
          )) ||
          (!/^EOS[0-9A-Za-z]{50}$|^PUB_K1_[0-9A-Za-z]{50}$/i.test(
            this.form.active_key
          ))
        );
      },
    },
    methods: {
        ...mapActions('testnet', ['faucet', 'evmFaucet', 'account']),
        async onFaucet() {
            this.submitting = true;
            const result = await this.faucet(this.form.send_to);
            if (result) {
                Notify.create({
                    message: result,
                    position: 'top',
                    color: 'primary',
                    textColor: 'white',
                    actions: [{ label: 'Dismiss', color: 'white' }],
                });
                this.transactionId = result.transactionId;
            }
            this.submitting = false;
        },
        async onEvmFaucet() {
            this.submitting = true;
            const result = await this.evmFaucet(this.form.send_to_evm);
            if (result) {
                Notify.create({
                    message: result,
                    position: 'top',
                    color: 'primary',
                    textColor: 'white',
                    actions: [{ label: 'Dismiss', color: 'white' }],
                });
                this.transactionId = result.transactionId;
            }
            this.submitting = false;
        },
        async onAccount() {
            this.submitting = true;
            const result = await this.account(this.form);
            if (result) {
                Notify.create({
                    message: result,
                    position: 'top',
                    color: 'primary',
                    textColor: 'white',
                    actions: [{ label: 'Dismiss', color: 'white' }],
                });
                this.transactionId = result.transactionId;
            }
            this.submitting = false;
        },
    },
};
</script>

<template lang="pug">
q-page.flex.flex-center
  q-card
    q-card-section
      div.center.text-h4 Testnet faucet
    q-card-section
      q-input.q-mb-lg(
        ref="account_name"
        v-model="form.account_name"
        color="accent"
        label="Account name"
        outlined
        maxlength="12"
      )
      q-input.q-mb-lg(
        ref="owner_key"
        v-model="form.owner_key"
        color="accent"
        :rules="[ val => (/^EOS[0-9A-Za-z]{50}$|^PUB_K1_[0-9A-Za-z]{50}$/i.test(val)) || 'Please provide a valid Owner key']"
        label="Owner key"
        outlined
      )
      q-input.q-mb-lg(
        ref="active_key"
        v-model="form.active_key"
        color="accent"
        :rules="[ val => (/^EOS[0-9A-Za-z]{50}$|^PUB_K1_[0-9A-Za-z]{50}$/i.test(val)) || 'Please provide a valid Active key']"

        label="Active key"
        outlined
      )
      q-btn.full-width(
        color="primary"
        label="Create testnet account"
        size="lg"
        unelevated
        :loading="submitting"
        :disable="isAnyInputInvalid"
        @click="onAccount"
      )

    q-card-section
      q-input.q-mb-lg(
        ref="send_to"
        v-model="form.send_to"
        color="accent"
        label="Send to Telos account"
        outlined
        maxlength="12"
      )
      q-btn.full-width(
        color="primary"
        label="Send testnet TLOS"
        size="lg"
        unelevated
        :loading="submitting"
        @click="onFaucet"
      )

    q-card-section
      q-input.q-mb-lg(
        ref="send_to_evm"
        v-model="form.send_to_evm"
        color="accent"
        label="Send to EVM address"
        :rules="[ val => /^0x[a-fA-F0-9]{40}$/.test(val) || 'Please provide a valid EVM address with 0x prefix']"
        outlined
      )
      q-btn.full-width(
        color="primary"
        label="Send testnet EVM TLOS"
        size="lg"
        unelevated
        :loading="submitting"
        @click="onEvmFaucet"
      )

    q-card-section(v-if="transactionId")
      a(
        :href="`https://telos-test.bloks.io/transaction/${transactionId}`"
        target="_blank"
      ) Trx explorer
</template>
