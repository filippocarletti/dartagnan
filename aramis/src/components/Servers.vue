<template>
  <div>
    <h2>Servers</h2>
    <div v-if="isLoading" class="spinner spinner-lg"></div>
    <button v-if="!isLoading && servers.length > 0" @click="addServer()" class="btn btn-primary btn-lg create-server">
      {{ $t('servers.create_server') }} </button>

    <!-- filters -->
    <div v-if="!isLoading" class="row toolbar-pf filters-container">
      <div class="col-sm-12">
        <form class="toolbar-pf-actions">
          <div class="form-group toolbar-pf-filter">
            <label class="sr-only" for="filter">Name</label>
            <div class="input-group">
              <input v-model="filters.search" type="text" class="form-control" id="filter" :placeholder="$t('servers.filter_by')">
              <div class="input-group-btn">
                <button class="btn btn-default" type="button">
                  <span class="fa fa-search"></span>
                </button>
              </div>
            </div>
            <!-- /input-group -->
          </div>
          <div class="form-group">
            <label>{{$t('servers.status')}}</label>
            <div class="dropdown btn-group renew-button">
              <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
                {{$t('servers.'+filters.currentStatus)}}
                <span class="caret"></span>
              </button>
              <ul class="dropdown-menu">
                <li @click="setFilterStatus(s)" v-for="(s, i) in filters.statuses" v-bind:key="i" :class="s == filters.currentStatus ? 'selected' : ''">
                  <a class="filter-option">{{$t('servers.'+s)}}</a>
                </li>
              </ul>
            </div>
          </div>
          <div class="form-group">
            <label>{{$t('servers.license_type')}}</label>
            <div class="dropdown btn-group renew-button">
              <button type="button" class="btn btn-default dropdown-toggle" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">{{filters.currentLicense && filters.currentLicense.name || '-'}}
                <span class="caret"></span>
              </button>
              <ul class="dropdown-menu">
                <li @click="setFilterLicense(l)" v-for="(l, i) in filters.licenses" v-bind:key="i" :class="l && l.code == filters.currentLicense.code ? 'selected' : ''">
                  <a class="filter-option">{{l && l.code == 'all' ? $t('servers.all'): l && l.name || '-'}}</a>
                </li>
              </ul>
            </div>
          </div>
          <div class="form-group">
            <button @click="setFilterOrderSort()" class="btn btn-link" type="button">
              <span :class="['fa', 'fa-sort-alpha-'+filters.currentSort]"></span>
            </button>
          </div>
        </form>
        <div class="row toolbar-pf-results">
          <div class="col-sm-12">
            <h5>{{filteredServers().length}} {{$t('servers.results')}}</h5>
            <span v-if="filters.search.length > 0 || filters.currentStatus !== 'all' || filters.currentLicense.code !== 'all'">
              <p>{{$t('servers.active_filters')}}:</p>
              <ul class="list-inline">
                <li v-if="filters.search.length > 0">
                  <span class="label label-info">
                    {{$t('servers.search')}}: {{filters.search}}
                    <a class="filter-option" @click="clearFilters('search')">
                      <span class="pficon pficon-close"></span>
                    </a>
                  </span>
                </li>
                <li v-if="filters.currentStatus !== 'all'">
                  <span class="label label-info">
                    {{$t('servers.status')}}: {{filters.currentStatus}}
                    <a class="filter-option" @click="clearFilters('status')">
                      <span class="pficon pficon-close"></span>
                    </a>
                  </span>
                </li>
                <li v-if="filters.currentLicense.code !== 'all'">
                  <span class="label label-info">
                    {{$t('servers.license_type')}}: {{filters.currentLicense && filters.currentLicense.name || '-'}}
                    <a class="filter-option" @click="clearFilters('license')">
                      <span class="pficon pficon-close"></span>
                    </a>
                  </span>
                </li>
              </ul>
              <p>
                <a class="filter-option" @click="clearFilters('all')">{{$t('servers.clear_filters')}}</a>
              </p>
            </span>
          </div>
        </div>
      </div>
    </div>
    <!-- end filters -->

    <div v-if="!isLoading && servers.length == 0" class="blank-slate-pf">
      <div class="blank-slate-pf-icon">
        <span class="pficon pficon pficon-server"></span>
      </div>
      <h1>
        {{ $t('servers.no_servers_title')}}
      </h1>
      <p>
        {{ $t('servers.no_servers_subtitle')}}
      </p>
      <div class="blank-slate-pf-main-action">
        <button @click="addServer()" class="btn btn-primary btn-lg"> {{ $t('servers.create_server')}} </button>
      </div>
    </div>

    <div v-if="!isLoading && servers.length > 0" class="row row-cards-pf servers-container">
      <div v-for="s in filteredServers()" v-bind:key="s.id" class="col-xs-12 col-sm-12 col-md-6 col-lg-3">
        <div :class="[isExpired(s.subscription.valid_until) ? 'disabled-top' : '', 'card-pf card-pf-view card-pf-view-select card-pf-view-single-select card-pf-accented']">
          <div class="card-pf-body">
            <div class="card-pf-top-element">
              <span :class="[isExpired(s.subscription.valid_until) ? 'disabled-circle' : '', 'pficon pficon-server card-pf-icon-circle adjust-icon-size']"></span>
            </div>
            <h2 class="card-pf-title text-center">
              {{s.hostname || '-'}}
            </h2>
            <div class="card-pf-items text-center">
              <div class="card-pf-item">
                <span class="pficon pficon-screen"></span>
                <span class="card-pf-item-text">{{s.public_ip || '-'}}</span>
              </div>
              <div class="card-pf-item">
                <span v-if="s.status == 'active'" class="pficon pficon-ok"></span>
                <span v-if="s.status == 'no_active'" class="pficon pficon-error-circle-o"></span>
                <span v-if="s.status == 'no_comm'" class="pficon pficon-help"></span>
              </div>
            </div>
            <p class="card-pf-info text-center">
              <strong>{{$t('servers.created')}}</strong>{{s.created | formatDate}}
            </p>
            <div class="divider"></div>
            <div class="card-pf-items text-center">
              <div class="card-pf-item">
                <span class="fa fa-star"></span>
                <span v-if="!isExpired(s.subscription.valid_until)" class="card-pf-item-text">{{s && s.subscription && s.subscription.subscription_plan && s.subscription.subscription_plan.name || '-'}}</span>
                <span v-if="isExpired(s.subscription.valid_until)" class="card-pf-item-text">{{$t('servers.expired')}}</span>
              </div>
              <div class="card-pf-item">
                <span v-if="!isExpired(s.subscription.valid_until)" class="pficon pficon-ok"></span>
                <span v-if="isExpired(s.subscription.valid_until)" class="pficon pficon-warning-triangle-o"></span>
              </div>
            </div>
            <p class="card-pf-info text-center">
              <strong>{{$t('servers.until')}}</strong>
              <span v-if="!isExpired(s.subscription.valid_until)">{{s.subscription.valid_until | formatDate(false)}}</span>
              <span class="gray" v-if="isExpired(s.subscription.valid_until)">
                <strong>{{$t('servers.expired')}}</strong>
              </span>
            </p>
          </div>
          <div class="card-pf-footer">
            <div class="card-pf-time-frame-filter">
              <renew-button :obj="s" :update="listServers"></renew-button>
            </div>
            <p>
              <button @click="showDeleteModal(s)" type="button" class="btn btn-danger">
                {{$t('servers.delete')}}
              </button>
            </p>
          </div>
        </div>
      </div>
    </div>

    <div class="modal fade" id="newServerModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
              <span class="pficon pficon-close"></span>
            </button>
            <h4 class="modal-title" id="myModalLabel">{{$t('servers.new_server')}}</h4>
          </div>
          <div class="modal-body">
            <div class="card-pf-view">
              <div>
                <div class="card-pf-top-element">
                  <span class="pficon pficon-server card-pf-icon-circle"></span>
                </div>
                <h2 class="card-pf-title text-center">
                  {{$t('servers.server_created')}}
                </h2>
                <div class="card-pf-items text-center">
                  {{$t('servers.server_id')}}
                  <div class="card-pf-item">
                    <span class="card-pf-item-text">{{newServer.secret}}</span>
                  </div>
                  <div class="servers-container">
                    <button v-clipboard="newServer.uuid" @success="handleCopy(true)" @error="handleCopy(false)" type="button" class="clipboard-btn">
                      <span class="fa fa-copy"></span>
                    </button>
                    <span v-if="copySucceeded" class="pficon pficon-ok renew-button"></span>
                  </div>
                </div>
                <div class="card-pf-items text-center">
                  <div class="alert alert-info alert-dismissable">
                    <span class="pficon pficon-info"></span>
                    {{$t('servers.paste_it')}}
                  </div>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-primary" data-dismiss="modal">{{$t('servers.done')}}</button>
          </div>
        </div>
      </div>
    </div>
    <div class="modal fade" id="deleteServerModal" tabindex="-1" role="dialog" aria-labelledby="myModalLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <button type="button" class="close" data-dismiss="modal" aria-hidden="true">
              <span class="pficon pficon-close"></span>
            </button>
            <h4 class="modal-title" id="myModalLabel">{{$t('servers.delete')}}</h4>
          </div>
          <div class="modal-body">
            <form class="form-horizontal">
              <div class="alert alert-warning alert-dismissable">
                <span class="pficon pficon-warning-triangle-o"></span>
                <strong>{{$t('servers.are_you_sure')}}</strong>. {{$t('servers.no_reversible')}}
              </div>
            </form>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-default" data-dismiss="modal">{{$t('servers.cancel')}}</button>
            <button @click="deleteServer()" type="button" class="btn btn-danger">{{$t('servers.delete')}}</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import LoginService from './../services/login';
  import StorageService from './../services/storage';
  import UtilService from './../services/util';
  import {
    setTimeout
  } from 'timers';
  import _ from 'lodash'

  import RenewButton from './directives/RenewButton.vue';

  export default {
    name: 'servers',
    mixins: [LoginService, StorageService, UtilService],
    props: ['action'],
    components: {
      renewButton: RenewButton,
    },
    data() {
      // get licenses list
      this.licenseList()

      //get servers list
      this.listServers()

      // handle action
      if (this.action == 'newServer') {
        this.addServer()
        this.delete('query_params')
      }

      return {
        copySucceeded: false,
        isLoading: true,
        filters: {
          search: '',
          currentStatus: this.get('servers_filter_status') || 'all',
          currentLicense: {
            name: 'All',
            code: 'all'
          },
          currentSort: this.get('servers_filter_sort') || 'desc',
          licenses: [{
            name: 'All',
            code: 'all'
          }],
          statuses: ['all', 'active', 'no_active', 'no_comm'],
        },
        newServer: {},
        toDelete: {},
        servers: []
      }
    },
    methods: {
      handleCopy(status) {
        this.copySucceeded = status
      },
      isExpired(date) {
        return new Date().toISOString() > date
      },
      addServer() {
        this.$http.post('http://' + this.$root.$options.api_host + '/api/ui/systems', {}, {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          this.newServer = success.body
          setTimeout(function () {
            $('#newServerModal').modal('toggle')
          }, 0)
          this.listServers()
        }, function (error) {
          console.error(error)
        });
      },
      showDeleteModal(server) {
        this.toDelete = server
        $('#deleteServerModal').modal('toggle')
      },
      deleteServer() {
        this.$http.delete('http://' + this.$root.$options.api_host + '/api/ui/systems/' + this.toDelete.id, {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          setTimeout(function () {
            $('#deleteServerModal').modal('toggle')
          }, 0)
          this.listServers()
        }, function (error) {
          console.error(error)
        });
      },
      licenseList() {
        this.$http.get('http://' + this.$root.$options.api_host + '/api/ui/plans', {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          this.filters.licenses = this.filters.licenses.concat(success.body)
          this.filters.currentLicense = this.get('servers_filter_license') || this.filters.licenses[0]
        }, function (error) {
          console.error(error)
        });
      },
      listServers() {
        this.isLoading = true
        this.$http.get('http://' + this.$root.$options.api_host + '/api/ui/systems', {
          headers: {
            'Authorization': 'Bearer ' + this.get('access_token', false) || ''
          }
        }).then(function (success) {
          this.servers = success.body
          this.isLoading = false
        }, function (error) {
          console.error(error)
          this.isLoading = false
        });
      },
      clearFilters(filter) {
        if (filter == 'all') {
          this.filters.search = ''
          this.filters.currentStatus = 'all'
          this.set('servers_filter_status', 'all')
          this.filters.currentLicense = {
            name: 'All',
            code: 'all'
          }
          this.set('servers_filter_license', {
            name: 'All',
            code: 'all'
          })
        }

        if (filter == 'search') {
          this.filters.search = ''
        }

        if (filter == 'status') {
          this.filters.currentStatus = 'all'
          this.set('servers_filter_status', 'all')
        }

        if (filter == 'license') {
          this.filters.currentLicense = {
            name: 'All',
            code: 'all'
          }
          this.set('servers_filter_license', {
            name: 'All',
            code: 'all'
          })
        }

      },
      setFilterStatus(status) {
        this.filters.currentStatus = status
        this.set('servers_filter_status', status)
        this.filteredServers()
      },
      setFilterLicense(license) {
        this.filters.currentLicense = license
        this.set('servers_filter_license', license)
        this.filteredServers()
      },
      setFilterOrderSort() {
        this.filters.currentSort = this.filters.currentSort == 'desc' ? 'asc' : 'desc'
        this.set('servers_filter_sort', this.filters.currentSort)
        this.filteredServers()
      },
      filteredServers() {
        var filter = {}
        if (this.filters.currentStatus != 'all') {
          filter['status'] = this.filters.currentStatus
        }
        if (this.filters.currentLicense.code != 'all') {
          filter['subscription'] = {
            subscription_plan: {
              code: this.filters.currentLicense.code
            }
          }
        }
        var filtered = _.filter(this.servers, filter);
        if (this.filters.search.length > 0) {
          filtered = _.filter(filtered, function (item) {
            return _.some(item && item.hostname, function (tag) {
              console.log(tag)
              return _.startsWith(tag, this.filters.search);
            });
          });
        }
        return _.orderBy(filtered, ['created'], this.filters.currentSort)
      }
    }
  }

</script>

<style scoped>
  .create-server {
    float: right;
    margin-top: -52px;
    margin-right: 35px;
  }

</style>
