<template>
  <div class="payment-create main-content">
    <form action="" @submit.prevent="submitPaymentData">
      <div class="page-header">
        <h3 class="page-title">{{ isEdit ? $t('payments.edit_payment') : $t('payments.new_payment') }}</h3>
        <ol class="breadcrumb">
          <li class="breadcrumb-item"><router-link slot="item-title" to="/admin/dashboard">{{ $t('general.home') }}</router-link></li>
          <li class="breadcrumb-item"><router-link slot="item-title" to="/admin/payments">{{ $tc('payments.payment', 2) }}</router-link></li>
          <li class="breadcrumb-item">{{ isEdit ? $t('payments.edit_payment') : $t('payments.new_payment') }}</li>
        </ol>
        <div class="page-actions header-button-container">
          <base-button
            :loading="isLoading"
            :disabled="isLoading"
            icon="save"
            color="theme"
            type="submit">
            {{ isEdit ? $t('payments.update_payment') : $t('payments.save_payment') }}
          </base-button>
        </div>
      </div>
      <div class="payment-card card">
        <div class="card-body">
          <div class="row">
            <div class="col-sm-6">
              <div class="form-group">
                <label class="form-label">{{ $t('payments.date') }}</label><span class="text-danger"> *</span>
                <base-date-picker
                  v-model="formData.payment_date"
                  :invalid="$v.formData.payment_date.$error"
                  :calendar-button="true"
                  calendar-button-icon="calendar"
                  @change="$v.formData.payment_date.$touch()"
                />
                <div v-if="$v.formData.payment_date.$error">
                  <span v-if="!$v.formData.payment_date.required" class="text-danger">{{ $t('validation.required') }}</span>
                </div>
              </div>
            </div>
            <div class="col-sm-6">
              <div class="form-group">
                <label class="form-label">{{ $t('payments.payment_number') }}</label><span class="text-danger"> *</span>
                <base-input
                  :invalid="$v.formData.payment_number.$error"
                  v-model.trim="formData.payment_number"
                  read-only
                  type="text"
                  name="email"
                  @input="$v.formData.payment_number.$touch()"
                />
                <div v-if="$v.formData.payment_number.$error">
                  <span v-if="!$v.formData.payment_number.required" class="text-danger">{{ $tc('validation.required') }}</span>
                </div>
              </div>
            </div>
            <div class="col-sm-6">
              <label class="form-label">{{ $t('payments.customer') }}</label><span class="text-danger"> *</span>
              <base-select
                ref="baseSelect"
                v-model="customer"
                :invalid="$v.customer.$error"
                :options="customerList"
                :searchable="true"
                :show-labels="false"
                :allow-empty="false"
                :disabled="isEdit"
                :placeholder="$t('customers.select_a_customer')"
                label="name"
                track-by="id"
              />
              <div v-if="$v.customer.$error">
                <span v-if="!$v.customer.required" class="text-danger">{{ $tc('validation.required') }}</span>
              </div>
            </div>
            <div class="col-sm-6">
              <div class="form-group">
                <label class="form-label">{{ $t('payments.invoice') }}</label>
                <base-select
                  v-model="invoice"
                  :options="invoiceList"
                  :searchable="true"
                  :show-labels="false"
                  :allow-empty="false"
                  :disabled="isEdit"
                  :placeholder="$t('invoices.select_invoice')"
                  label="invoice_number"
                  track-by="invoice_number"
                />
              </div>
            </div>
            <div class="col-sm-6">
              <div class="form-group">
                <label class="form-label">{{ $t('payments.amount') }}</label><span class="text-danger"> *</span>
                <div class="base-input">
                  <money
                    :class="{'invalid' : $v.formData.amount.$error}"
                    v-model="amount"
                    v-bind="customerCurrency"
                    class="input-field"
                  />
                </div>
                <div v-if="$v.formData.amount.$error">
                  <span v-if="!$v.formData.amount.required" class="text-danger">{{ $t('validation.required') }}</span>
                  <span v-if="!$v.formData.amount.between && $v.formData.amount.numeric && amount <= 0" class="text-danger">{{ $t('validation.payment_greater_than_zero') }}</span>
                  <span v-if="!$v.formData.amount.between && amount > 0" class="text-danger">{{ $t('validation.payment_greater_than_due_amount') }}</span>
                </div>
              </div>
            </div>
            <div class="col-sm-6">
              <div class="form-group">
                <label class="form-label">{{ $t('payments.payment_mode') }}</label>
                <base-select
                  v-model="formData.payment_mode"
                  :options="getPaymentMode"
                  :searchable="true"
                  :show-labels="false"
                  :placeholder="$t('payments.select_payment_mode')"
                />
              </div>
            </div>
            <div class="col-sm-12 ">
              <div class="form-group">
                <label class="form-label">{{ $t('payments.note') }}</label>
                <base-text-area
                  v-model="formData.notes"
                  @input="$v.formData.notes.$touch()"
                />
                <div v-if="$v.formData.notes.$error">
                  <span v-if="!$v.formData.notes.maxLength" class="text-danger">{{ $t('validation.notes_maxlength') }}</span>
                </div>
              </div>
            </div>
            <div class="col-sm-12">
              <div class="form-group collapse-button-container">
                <base-button
                  :loading="isLoading"
                  icon="save"
                  color="theme"
                  type="submit"
                  class="collapse-button"
                >
                  {{ $t('payments.save_payment') }}
                </base-button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </form>
  </div>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import MultiSelect from 'vue-multiselect'
import { validationMixin } from 'vuelidate'
import moment from 'moment'
const { required, between, maxLength } = require('vuelidate/lib/validators')

export default {
  components: { MultiSelect },
  mixins: [validationMixin],
  data () {
    return {
      formData: {
        user_id: null,
        payment_number: null,
        payment_date: null,
        amount: 0,
        payment_mode: null,
        invoice_id: null,
        notes: null
      },
      money: {
        decimal: '.',
        thousands: ',',
        prefix: '$ ',
        precision: 2,
        masked: false
      },
      customer: null,
      invoice: null,
      customerList: [],
      invoiceList: [],
      isLoading: false,
      maxPayableAmount: Number.MAX_SAFE_INTEGER
    }
  },
  validations () {
    return {
      customer: {
        required
      },
      formData: {
        payment_number: {
          required
        },
        payment_date: {
          required
        },
        amount: {
          required,
          between: between(1, this.maxPayableAmount + 1)
        },
        notes: {
          maxLength: maxLength(255)
        }
      }
    }
  },
  computed: {
    ...mapGetters('currency', [
      'defaultCurrencyForInput'
    ]),
    getPaymentMode () {
      return ['Cash', 'Check', 'Credit Card', 'Bank Transfer']
    },
    amount: {
      get: function () {
        return this.formData.amount / 100
      },
      set: function (newValue) {
        this.formData.amount = newValue * 100
      }
    },
    isEdit () {
      if (this.$route.name === 'payments.edit') {
        return true
      }
      return false
    },
    customerCurrency () {
      if (this.customer && this.customer.currency) {
        return {
          decimal: this.customer.currency.decimal_separator,
          thousands: this.customer.currency.thousand_separator,
          prefix: this.customer.currency.symbol + ' ',
          precision: this.customer.currency.precision,
          masked: false
        }
      } else {
        return this.defaultCurrencyForInput
      }
    }
  },
  watch: {
    customer (newValue) {
      this.formData.user_id = newValue.id
      this.invoice = null
      this.formData.amount = 0
      this.invoiceList = []
      if (!this.isEdit) {
        this.fetchCustomerInvoices(newValue.id)
      }
    },
    invoice (newValue) {
      if (newValue) {
        this.formData.invoice_id = newValue.id
        if (!this.isEdit) {
          this.setPaymentAmountByInvoiceData(newValue.id)
        }
      }
    }
  },
  async mounted () {
    // if (!this.$route.params.id) {
    //   this.$refs.baseSelect.$refs.search.focus()
    // }
    this.$nextTick(() => {
      this.loadData()
      if (this.$route.params.id && !this.isEdit) {
        this.setInvoicePaymentData()
      }
    })
  },
  methods: {
    ...mapActions('invoice', [
      'fetchInvoice'
    ]),
    ...mapActions('payment', [
      'fetchCreatePayment',
      'addPayment',
      'updatePayment',
      'fetchPayment'
    ]),
    async loadData () {
      if (this.isEdit) {
        let response = await this.fetchPayment(this.$route.params.id)
        this.customerList = response.data.customers
        this.formData = { ...response.data.payment }
        this.customer = response.data.payment.user
        this.formData.payment_date = moment(response.data.payment.payment_date, 'YYYY-MM-DD').toString()
        this.formData.amount = parseFloat(response.data.payment.amount)
        this.maxPayableAmount = response.data.payment.amount
        if (response.data.payment.invoice !== null) {
          this.maxPayableAmount = parseInt(response.data.payment.amount) + parseInt(response.data.payment.invoice.due_amount)
          this.invoice = response.data.payment.invoice
        }
        // this.fetchCustomerInvoices(this.customer.id)
      } else {
        let response = await this.fetchCreatePayment()
        this.customerList = response.data.customers
        this.formData.payment_number = response.data.nextPaymentNumber
        this.formData.payment_date = moment(new Date()).toString()
      }
      return true
    },
    async setInvoicePaymentData () {
      let data = await this.fetchInvoice(this.$route.params.id)
      this.customer = data.data.invoice.user
      this.invoice = data.data.invoice
    },
    async setPaymentAmountByInvoiceData (id) {
      let data = await this.fetchInvoice(id)
      this.formData.amount = data.data.invoice.due_amount
      this.maxPayableAmount = data.data.invoice.due_amount
    },
    async fetchCustomerInvoices (userID) {
      let response = await axios.get(`/api/invoices/unpaid/${userID}`)
      if (response.data) {
        this.invoiceList = response.data.invoices
      }
    },
    async submitPaymentData () {
      this.$v.customer.$touch()
      this.$v.formData.$touch()
      if (this.$v.$invalid) {
        return true
      }
      if (this.isEdit) {
        let data = {
          editData: {
            ...this.formData,
            payment_date: moment(this.formData.payment_date).format('DD/MM/YYYY')
          },
          id: this.$route.params.id
        }
        let response = await this.updatePayment(data)
        if (response.data.success) {
          window.toastr['success'](this.$t('payments.updated_message'))
          this.$router.push('/admin/payments')
          return true
        }
        if (response.data.error === 'invalid_amount') {
          window.toastr['error'](this.$t('invalid_amount_message'))
          return false
        }
        window.toastr['error'](response.data.error)
      } else {
        let data = {
          ...this.formData,
          payment_date: moment(this.formData.payment_date).format('DD/MM/YYYY')
        }
        this.isLoading = true
        let response = await this.addPayment(data)
        if (response.data.success) {
          window.toastr['success'](this.$t('payments.created_message'))
          this.$router.push('/admin/payments')
          this.isLoading = true
          return true
        }
        if (response.data.error === 'invalid_amount') {
          window.toastr['error'](this.$t('invalid_amount_message'))
          return false
        }
        window.toastr['error'](response.data.error)
      }
    }
  }
}
</script>
