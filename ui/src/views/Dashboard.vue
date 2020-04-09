<!--
#
# Copyright (C) 2020 Nethesis S.r.l.
# http://www.nethesis.it - nethserver@nethesis.it
#
# This script is part of NethServer.
#
# NethServer is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License,
# or any later version.
#
# NethServer is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with NethServer.  If not, see COPYING.
#
-->

<template>
  <div>
    <h2>{{$t('dashboard.title')}}</h2>
    <!-- error message -->
    <div v-if="errorMessage" class="alert alert-danger alert-dismissable">
      <button type="button" class="close" @click="closeErrorMessage()" aria-label="Close">
        <span class="pficon pficon-close"></span>
      </button>
      <span class="pficon pficon-error-circle-o"></span>
      {{ errorMessage }}.
    </div>

    <div v-show="!uiLoaded" class="spinner spinner-lg"></div>
    <div v-show="uiLoaded">
      <div class="row rowstats">
        <div class="content">
          <div class="stats-container col-xs-12 col-sm-6 col-md-4 col-lg-4">
            <span class="card-pf-utilization-card-details-count stats-count">
              <span
                :class="status.MssqlServer ? 'fa fa-check text-success' : 'fa fa-times text-danger'"
                data-toggle="tooltip"
                data-placement="top"
                :title="status.MssqlServer ? $t('dashboard.running') : $t('dashboard.not_running')"
              ></span>
            </span>
            <span class="card-pf-utilization-card-details-description stats-description">
              <span
                class="card-pf-utilization-card-details-line-2 stats-text"
              >{{$t('dashboard.mssql_status')}}</span>
            </span>
          </div>
        </div>
      </div>
      <div class="row rowstats">
        <div class="content">
          <div class="stats-container col-xs-12 col-sm-6 col-md-4 col-lg-4">
            <span class="card-pf-utilization-card-details-count stats-count">
              <span
                :class="status.SharedFolder ? 'fa fa-check text-success' : 'fa fa-times text-danger'"
                data-toggle="tooltip"
                data-placement="top"
                :title="status.SharedFolder ? $t('dashboard.configured') : $t('dashboard.not_configured')"
              ></span>
            </span>
            <span class="card-pf-utilization-card-details-description stats-description">
              <span
                class="card-pf-utilization-card-details-line-2 stats-text"
              >{{$t('dashboard.shared_folder_status')}}</span>
            </span>
          </div>
        </div>
      </div>
      <div class="row rowstats">
        <div class="content">
          <div class="stats-container col-xs-12 col-sm-6 col-md-4 col-lg-4">
            <span class="card-pf-utilization-card-details-count stats-count">
              <span
                :class="status.ArcprocDB ? 'fa fa-check text-success' : 'fa fa-times text-danger'"
                data-toggle="tooltip"
                data-placement="top"
                :title="status.ArcprocDB ? $t('dashboard.configured') : $t('dashboard.not_configured')"
              ></span>
            </span>
            <span class="card-pf-utilization-card-details-description stats-description">
              <span
                class="card-pf-utilization-card-details-line-2 stats-text"
              >{{$t('dashboard.arcproc_status')}}</span>
            </span>
          </div>
        </div>
      </div>
      <div class="row rowstats">
        <div class="content">
          <div class="stats-container col-xs-12 col-sm-6 col-md-4 col-lg-4">
            <span class="card-pf-utilization-card-details-count stats-count">{{status.AggNumber}}</span>
            <span class="card-pf-utilization-card-details-description stats-description">
              <span
                class="card-pf-utilization-card-details-line-2 stats-text"
              >{{$t('dashboard.agg_number')}}</span>
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Dashboard",
  components: {
  },
  props: {
  },
  mounted() {
    this.getDashboardData()
  },
  data() {
    return {
      uiLoaded: false,
      errorMessage: null,
      status: {
        MssqlStatus: false,
        SharedFolder: false,
        ArcprocDB: false,
        AggNumber: "N/A"
      }
    };
  },
  methods: {
    showErrorMessage(errorMessage, error) {
      console.error(errorMessage, error) /* eslint-disable-line no-console */
      this.errorMessage = errorMessage
    },
    closeErrorMessage() {
      this.errorMessage = null
    },
    getDashboardData() {
      var context = this;
      context.uiLoaded = false;
      nethserver.exec(
        ["nethserver-business/dashboard/read"],
        { action: "status" },
        null,
        function(success) {
          try {
            context.status = JSON.parse(success).status;
            
            setTimeout(function() {
              $('[data-toggle="tooltip"]').tooltip();
            }, 250);
          } catch (e) {
            console.error(e);
          }
          context.uiLoaded = true;
        },
        function(error) {
          context.showErrorMessage(context.$i18n.t("dashboard.error_reading_business_status"), error);
        }
      );
    }
  }
};
</script>

<style>
.stats-description-small {
  float: left;
  overflow: hidden;
  text-overflow: ellipsis;
  font-size: 16px;
  font-weight: 300;
  line-height: 2;
}
.stats-text-small {
  line-height: 0.5;
}
.stats-count {
  font-size: 26px;
  font-weight: 300;
  margin-right: 10px;
  float: left;
  line-height: 1;
}
.rowstats {
  padding-left: 20px;
  padding-right: 20px;
}
.stats-text {
  margin-top: 10px !important;
  display: block;
}
.stats-description {
  float: left;
  line-height: 1;
}
.stats-container {
  padding: 20px !important;
  border-width: initial !important;
  border-style: none !important;
  border-color: initial !important;
  -o-border-image: initial !important;
  border-image: initial !important;
}
.stats-text {
  margin-top: 10px !important;
  display: block;
}
.stats-description {
  float: left;
  line-height: 1;
}
.stats-count {
  font-size: 26px;
  font-weight: 300;
  margin-right: 10px;
  float: left;
  line-height: 1;
}
.item {
  padding-top: 3px;
}
.space {
  margin-bottom: 25px;
}
.margintop {
  margin-top: 10px;
}
</style>