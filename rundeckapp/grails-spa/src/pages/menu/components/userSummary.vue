<template>
  <div class="container-fluid">
    <div class="card">
      <div class="card-content">
        <div class="row">
          <div class="col-xs-12">
            <div class="form-group pull-left" style="padding-top:10px">
              <span class="prompt">{{ $t("message.pageUsersSummary")}}</span>
            </div>
            <div class="form-group pull-right" style="display:inline-block;">
              <div class="checkbox" v-if="showLoginStatus">
                <input type="checkbox" name="loggedOnly" id="loggedOnly" v-model="loggedOnly" @change="loadUsersList(0)">
                <label for="loggedOnly">
                  {{ $t("message.pageUserLoggedOnly")}}
                </label>
              </div>
              <div class="checkbox">
                <input type="checkbox" name="includeExec" id="includeExec" v-model="includeExec" @change="loadUsersList(0)">
                <label for="includeExec">
                  {{ $t("message.paramIncludeExecTitle")}}
                </label>
              </div>
            </div>
          </div>
        </div>
        <div class="base-filters">
          <div class="row">
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <input
                  type="text"
                  class="form-control"
                  :placeholder="$t( 'message.pageFilterLogin')"
                  v-model="loginFilter"
                >
              </div>
            </div>
            <div class="col-xs-12 col-sm-4" v-if="sessionIdEnabled">
              <div class="form-group">
                <input
                  type="text"
                  class="form-control"
                  :placeholder="$t( 'message.pageFilterSessionID')"
                  v-model="sessionIdFilter"
                >
              </div>
            </div>
            <div class="col-xs-12 col-sm-4">
              <div class="form-group">
                <input
                  type="text"
                  class="form-control"
                  :placeholder="$t( 'message.pageFilterHostName')"
                  v-model="hostNameFilter"
                >
              </div>
            </div>
            <div class="col-xs-12">
              <div class="form-group pull-right">
                <btn @click="loadUsersList(0)" type="primary">{{ $t("message.pageFilterBtnSearch")}}</btn>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="row">
      <div class="col-sm-12">
        <div class="card">
          <div class="card-content">
            <div class="pageBody" id="userProfilePage">
              <div class="row">
                <div class="col-sm-12">
                  <section class="section-space-bottom">
                    <span >
                      {{ $t("message.pageUsersTotalFounds")}}
                      <span v-if="!loading && this.pagination.total>=0" class="text-info">{{this.pagination.total}}</span>
                      <b class="fas fa-circle-notch fa-spin text-info" v-if="loading"></b>
                    </span>
                  </section>
                  <table class="table table-condensed  table-striped">
                    <tbody>
                    <tr>
                      <th class="table-header">
                        {{ $t("message.pageUsersLoginLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.domainUserEmailLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.domainUserFirstNameLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.domainUserLastNameLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.pageUsersCreatedLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.pageUsersUpdatedLabel")}}
                      </th>
                      <th v-if="includeExec" class="table-header">
                        {{ $t("message.pageUsersLastjobLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.pageUsersTokensLabel")}}
                        <span class="has_tooltip text-primary" data-placement="bottom"
                              :data-original-title="$t('message.pageUsersTokensHelp')">
                            <i class="glyphicon glyphicon-question-sign"></i>
                          </span>
                      </th>
                      <th class="table-header" v-if="sessionIdEnabled">
                        {{ $t("message.pageUsersSessionIDLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.pageUsersHostNameLabel")}}
                      </th>
                      <th class="table-header">
                        {{ $t("message.pageUsersLastLoginInTimeLabel")}}
                      </th>
                      <th class="table-header" v-if="showLoginStatus">
                        {{ $t("message.pageUsersLoggedStatus")}}
                      </th>
                    </tr>

                    <tr v-for="user in users">
                      <td>
                        <login-status :status="user.loggedStatus" :label="false" :showLoginStatus="showLoginStatus"/>
                        {{user.login}}
                      </td>
                      <td v-if="user.email">{{user.email}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      </td>
                      <td v-if="user.firstName">{{user.firstName}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      </td>
                      <td v-if="user.lastName">{{user.lastName}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      </td>
                      <td v-if="user.created">{{user.created | moment("MM/DD/YYYY hh:mm a")}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      </td>
                      <td v-if="user.updated" :class="user.updated===user.created?'text-muted ':''">
                        {{user.updated | moment("MM/DD/YYYY hh:mm a")}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      </td>
                      <td v-if="includeExec && user.lastJob">{{user.lastJob | moment("MM/DD/YYYY hh:mm a")}}
                      </td>
                      <td v-else-if="includeExec"><span class="text-muted small text-uppercase">{{ $t("message.pageUserNone")}}</span>
                      </td>
                      <td v-if="user.tokens > 0">{{user.tokens}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNone")}}</span>
                      </td>
                      <td v-if="user.lastSessionId && sessionIdEnabled">{{user.lastSessionId}}
                      </td>
                      <td v-else-if="sessionIdEnabled"><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      <td v-if="user.lastHostName">{{user.lastHostName}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      <td v-if="user.loggedInTime">
                      {{user.loggedInTime | moment("MM/DD/YYYY hh:mm a")}}
                      {{user.loggedInTime | moment('from','now')}}
                      </td>
                      <td v-else><span class="text-muted small text-uppercase">{{ $t("message.pageUserNotSet")}}</span>
                      </td>
                      <td v-if="showLoginStatus">
                        <login-status :status="user.loggedStatus" :label="true" :showLoginStatus="showLoginStatus"/>
                      </td>
                    </tr>
                    </tbody>
                  </table>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <offset-pagination
      :pagination="pagination"
      @change="changePageOffset($event)"
      :disabled="loading"
      :showPrefix="false"
    >
    </offset-pagination>
  </div>
</template>

<script>

  import axios from 'axios'
  import OffsetPagination from '@rundeck/ui-trellis/src/components/utils/OffsetPagination.vue'
  import LoginStatus from './LoginStatus'

  export default {
    name: 'UserSummary',
    components:{
      OffsetPagination,
      LoginStatus
    },
    props: [
      'menu',
      'userSummary'
    ],
    data () {
      return {
        showLoginStatus: false,
        sessionIdEnabled: false,
        users: [],
        loggedOnly: false,
        includeExec: false,
        sessionIdFilter: "",
        hostNameFilter: "",
        loginFilter: "",
        loading: false,
        rdBase: null,
        pagination:{
          offset:0,
          max:100,
          total:-1
        }
      }
    },
    methods: {
      changePageOffset(offset) {
        if (this.loading) {
          return;
        }
        this.loadUsersList(offset)
      },
      setSummaryPageConfig(){
          this.loading = true
          this.loggedOnly = false
          axios({
              method: 'get',
              headers: {'x-rundeck-ajax': true},
              url: `${this.rdBase}/user/getSummaryPageConfig`,
              withCredentials: true
          }).then((response) => {
              this.loggedOnly = response.data.loggedOnly
              this.showLoginStatus = response.data.showLoginStatus
              this.loadUsersList(0)
          })
      },

      async loadUsersList(offset) {
        this.loading = true
        this.pagination.offset = offset
        axios({
          method: 'get',
          headers: {'x-rundeck-ajax': true},
          url: `${this.rdBase}/user/loadUsersList`,
          params: {
            loggedOnly: `${this.loggedOnly}`,
            includeExec: `${this.includeExec}`,
            offset: this.pagination.offset,
            sessionFilter: this.sessionIdFilter,
            hostNameFilter: this.hostNameFilter,
            loginFilter: this.loginFilter
          },
          withCredentials: true
        }).then((response) => {
          this.pagination.max = response.data.maxRows
          this.pagination.total = response.data.totalRecords
          this.users = response.data.users
          this.sessionIdEnabled = response.data.sessionIdEnabled
          this.showLoginStatus = response.data.showLoginStatus
          this.loading = false
        })
      }
    },
    beforeMount() {
      if (window._rundeck && window._rundeck.rdBase) {
        this.rdBase = window._rundeck.rdBase;
        this.setSummaryPageConfig()
      }
    }
  }
</script>

<style scoped lang="scss">
  ul.users {
    display: inline;
    margin: 0;
    padding: 0;
  }

  ul.users li {
    display: inline;
    list-style: none;
    margin: 0;
    padding: 0;
  }

  ul.users li:after {
    content: ", ";
    color: #aaa;
  }

  ul.users li:last-child:after {
    content: "";
  }
</style>
