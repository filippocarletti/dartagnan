<template>
  <span>
    <button @click="showRenewModal()" type="button" class="btn btn-primary">
      <span class="fa fa-refresh"></span>
      Renew
    </button>
    <div class="modal fade" :id="'paymentModalRenew-'+obj.id" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
              <span class="pficon pficon-close"></span>
            </button>
            <h4 class="modal-title" id="myModalLabel">{{onUpgrade ? $t('payment.upgrade') : $t('payment.renew')}}</h4>
          </div>
          <div class="modal-body">
            <div class="card-pf-view">
              <div>
                <div class="card-pf-top-element">
                  <span :class="['fa', onUpgrade ? 'fa-certificate' : 'fa-refresh', 'card-pf-icon-circle', 'adjust-icon-size']"></span>
                </div>
                <h2 class="card-pf-title text-center">
                  {{onUpgrade ? $t('payment.upgrade_proceed') : $t('payment.renew_proceed')}}
                </h2>
                <div class="card-pf-items text-center">
                  <div class="card-pf-item details-pay-item">
                    <span class="card-pf-item-text">
                      <strong>{{$t('payment.plan')}}</strong>
                    </span>
                  </div>
                  <div class="card-pf-item details-pay-item">
                    <div class="dropdown">
                      <button class="btn btn-default dropdown-toggle" type="button" id="dropdownMenu1" data-toggle="dropdown">
                        {{currentPlan.name}}
                        <span class="caret"></span>
                      </button>
                      <ul class="dropdown-menu" role="menu" aria-labelledby="dropdownMenu1">
                        <li v-for="p in plans" v-if="p.code !== 'trial'" v-bind:key="p.code" role="presentation" :class="[p.code == currentPlan.code ? 'selected' : '']">
                          <a @click="changePlan(p)" role="menuitem" tabindex="-1" href="#">{{p.name}}</a>
                        </li>
                      </ul>
                    </div>
                  </div>
                </div>
                <div class="card-pf-items text-center">
                  <div class="card-pf-item details-pay-item">
                    <span class="card-pf-item-text">
                      <strong>{{$t('payment.details')}}</strong>
                    </span>
                  </div>
                  <div class="card-pf-item details-pay-item">
                    <span v-if="!onUpgradePriceCalc" >{{currentPlan.description}}</span>
                    <div v-if="onUpgradePriceCalc" class="spinner spinner-sm"></div>
                  </div>
                </div>
                <div class="card-pf-items text-center">
                  <div class="card-pf-item details-pay-item">
                    <span class="card-pf-item-text">
                      <strong>{{$t('payment.price')}}</strong>
                    </span>
                  </div>
                  <div class="card-pf-item details-pay-item">
                    <span v-if="!onUpgradePriceCalc" class="card-pf-item-text">{{currentPlan.price > 0 ? currentPlan.price : 0}} €</span>
                    <div v-if="onUpgradePriceCalc" class="spinner spinner-sm"></div>
                  </div>
                </div>
                <div class="card-pf-items text-center">
                  <div class="card-pf-item details-pay-item">
                    <span class="card-pf-item-text">
                      <strong>{{$t('payment.period')}}</strong>
                    </span>
                  </div>
                  <div class="card-pf-item details-pay-item">
                    <span v-if="!onUpgrade && !onUpgradePriceCalc" class="card-pf-item-text">{{obj.subscription.valid_until | formatDate(false)}} - {{calculateSubscription(obj.subscription.valid_until,
                      obj.subscription.subscription_plan) | formatDate(false)}}</span>
                    <span v-if="onUpgrade && !onUpgradePriceCalc" class="card-pf-item-text">{{new Date().toISOString() | formatDate(false)}} - {{calculateSubscription(new Date().toISOString(),
                      currentPlan) | formatDate(false)}}</span>
                    <div v-if="onUpgradePriceCalc" class="spinner spinner-sm"></div>
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="divider"></div>
          <div class="modal-footer text-center">
            <div v-if="payment.onProgress" class="spinner"></div>
            <div v-if="payment.done && !errors.state && !payment.onProgress" class="alert alert-success alert-dismissable">
              <span class="pficon pficon-ok"></span>
              <strong>{{$t('payment.confirmed')}}</strong>. {{$t('payment.payment_id_ref')}}
              <pre class="filters-container"><strong>{{payment.details.paymentID}}</strong></pre>
            </div>
            <button v-if="payment.done && !errors.state && !payment.onProgress" type="button" class="btn btn-primary done-payment-button"
              @click="hideRenewModal()">{{$t('servers.done')}}</button>
            <div v-if="!payment.done && !errors.state && !payment.onProgress" class="card-pf-item details-pay-item">
              <span class="card-pf-item-text">
                <strong>Pay with</strong>
              </span>
            </div>
            <div v-show="!payment.done && !errors.state && !payment.onProgress" :id="'paypal-button-container-'+obj.id"></div>
            <div v-if="payment.done && errors.state && !payment.onProgress" class="alert alert-danger alert-dismissable">
              <span class="pficon pficon-error-circle-o"></span>
              <strong>Hey there is a problem!</strong>. {{errors.message}}
            </div>
          </div>
        </div>
      </div>
    </div>
  </span>
</template>
<script>
  import StorageService from './../../services/storage';

  import paypal from 'paypal-checkout'

  export default {
    name: 'RenewButton',
    props: ['obj', 'update'],
    mixins: [StorageService],
    mounted() {
      var context = this
      paypal.Button.render({
        env: 'sandbox',
        style: {
          layout: 'vertical', // horizontal | vertical
          size: 'medium', // medium | large | responsive
          shape: 'rect', // pill | rect
          color: 'gold' // gold | blue | silver | black
        },
        funding: {
          allowed: [paypal.FUNDING.CARD],
          disallowed: [paypal.FUNDING.CREDIT]
        },
        client: {
          sandbox: 'Aeya87Tf68qIoNsevVnMCv-JsaU8QqjWR40LuU7bVEHR093d-1TV6XGZxXjBeNMpANFC04aZuC1Nwosc',
          production: '<insert production client id>'
        },
        payment: function (data, actions) {
          return actions.payment.create({
            payment: {
              transactions: [{
                amount: {
                  total: context.currentPlan.price > 0 ? context.currentPlan.price : 0.01,
                  currency: 'EUR'
                },
                "item_list": {
                  "items": [{
                    "name": context.currentPlan.code,
                    "description": context.currentPlan.name,
                    "sku": context.obj.uuid,
                    "price": context.currentPlan.price > 0 ? context.currentPlan.price : 0.01,
                    "currency": "EUR",
                    "quantity": "1",
                  }, ],
                },
              }]
            },
            experience: {
              input_fields: {
                no_shipping: 1
              }
            }
          });
        },

        onAuthorize: function (data, actions) {
          return actions.payment.get().then(function (paymentDetails) {
            console.log(paymentDetails)
            // Execute the payment
            return actions.payment.execute().then(function () {
              console.log('Payment Complete!', data);
              context.payment.onProgress = true
              if (context.onUpgrade) {
                context.upgradeCheck(data)
              } else {
                context.renewCheck(data)
              }
            });
          });
        }

      }, '#paypal-button-container-' + this.obj.id);
    },
    data() {
      // get plans
      this.plansList()

      return {
        payment: {
          done: false,
          details: {},
          onProgress: false
        },
        errors: {
          message: '',
          state: false
        },
        plans: [],
        currentPlan: this.obj.subscription.subscription_plan,
        onUpgradePriceCalc: false,
        onUpgrade: false
      }
    },
    methods: {
      showRenewModal() {
        this.payment.done = false
        this.payment.onProgress = false
        this.payment.details = {}
        this.errors.message = ''
        this.errors.state = false
        this.currentPlan = this.obj.subscription.subscription_plan
        this.onUpgradePriceCalc = false
        this.onUpgrade = false
        $('#paymentModalRenew-' + this.obj.id).modal('toggle')
      },
      hideRenewModal() {
        $('#paymentModalRenew-' + this.obj.id).modal('hide')
        this.update()
      },
      plansList() {
        this.$http.get('http://' + this.$root.$options.api_host + '/api/ui/plans', {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          this.plans = _.orderBy(success.body, 'price', 'asc')
        }, function (error) {
          console.error(error)
        });
      },
      changePlan(plan) {
        if (plan.code !== this.obj.subscription.subscription_plan.code) {
          // handle upgrade
          var context = this
          this.calculateUpgradePrice(plan.code, function (data) {
            context.onUpgrade = true
            context.currentPlan = plan
            context.currentPlan.price = data.price
          })
        } else {
          this.onUpgrade = false
          this.currentPlan = plan
        }
      },
      calculateUpgradePrice(plan, callback) {
        this.onUpgradePriceCalc = true
        this.$http.get('http://' + this.$root.$options.api_host + '/api/ui/systems/' + this.obj.id +
          '/upgrade_price?plan=' + plan, {
            headers: {
              'Authorization': 'Bearer ' + this.get('access_token', false) || ''
            }
          }).then(function (success) {
          this.onUpgradePriceCalc = false
          callback(success.body)
        }, function (error) {
          console.error(error)
          this.onUpgradePriceCalc = false
        });
      },
      calculateSubscription(date, subscription) {
        var moment = require("patternfly/node_modules/moment/moment.js")
        return moment(date, "YYYY-MM-DDTHH:mm:ss.sssZ").add(subscription.period, 'days');
      },
      renewCheck(payment) {
        this.$http.post('http://' + this.$root.$options.api_host + '/api/ui/systems/' + this.obj.id + '/renewal', {
          payment_id: payment.paymentID
        }, {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          this.payment.onProgress = false
          this.payment.details = payment
          this.payment.done = true
        }, function (error) {
          console.error(error)
          this.payment.onProgress = false
          this.payment.details = {}
          this.payment.done = true
          this.errors.state = true
          this.errors.message = error.body.message
        });
      },
      upgradeCheck(payment) {
        this.$http.post('http://' + this.$root.$options.api_host + '/api/ui/systems/' + this.obj.id + '/upgrade', {
          payment_id: payment.paymentID,
          subscription_plan_id: this.currentPlan.id,
        }, {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          this.payment.onProgress = false
          this.payment.details = payment
          this.payment.done = true
        }, function (error) {
          console.error(error)
          this.payment.onProgress = false
          this.payment.details = {}
          this.payment.done = true
          this.errors.state = true
          this.errors.message = error.body.message
        });
      },
    }
  }

</script>
<style>


</style>
