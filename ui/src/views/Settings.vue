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
    <h2>{{$t('settings.title')}}</h2>
    
    <!-- error message -->
    <div v-if="errorMessage" class="alert alert-danger alert-dismissable">
      <button type="button" class="close" @click="closeErrorMessage()" aria-label="Close">
        <span class="pficon pficon-close"></span>
      </button>
      <span class="pficon pficon-error-circle-o"></span>
      {{ errorMessage }}.
    </div>

    <div v-if="!uiLoaded" class="spinner spinner-lg"></div>
    <div v-if="uiLoaded">
      <div v-if="configuration.installed && configuration.isrunning" class="margintop">
        <h3>{{$t('settings.configure_shared_folder')}}</h3>
        <h5>{{$t('settings.shared_folder_desc')}}.</h5>
        <div v-show="configuration.SharedFolderStatus">
          <div class="alert alert-success col-lg-7">
            <span class="fa fa-check"></span>
            {{$t('settings.default_shared_folder_already_configured')}}.
          </div>
        </div>
        <form class="form-horizontal" v-on:submit.prevent="confSharedFolder()">
          <div class="row">
            <div class="col-lg-12">
              <div :class="['form-group margintop', errors.SharedFolder.hasError ? 'has-error' : '']">
                <label class="col-sm-2 control-label">
                  {{$t('settings.shared_folder_name')}}
                  <doc-info
                    :placement="'top'"
                    :title="$t('settings.shared_folder')"
                    :chapter="'SharedFolder'"
                    :inline="true"
                  ></doc-info>
                </label>
                <div class="col-sm-5">
                  <input
                    v-model="configuration.SharedFolder"
                    type="text"
                    class="form-control"
                    required
                    :disabled="configuration.SharedFolderStatus"
                  >
                  <span v-if="errors.SharedFolder.hasError" class="help-block">{{$t('settings.shared_folder_not_exists_or_not_empty')}}</span>
                </div>
              </div>
              <div class="form-group margintop">
                <label class="col-sm-2 control-label">
                </label>
                <div class="col-sm-5">
                  <button 
                    class="btn btn-primary" 
                    type="submit"
                    :disabled="configuration.SharedFolderStatus || !configuration.SharedFolder"
                  >
                    {{$t('settings.connect_share')}}
                  </button>
                  <button 
                    class="btn btn-danger marginleft" 
                    type="button"
                    :disabled="!configuration.SharedFolderStatus"
                    v-on:click="disconnectSharedFolder()"
                  >
                    {{$t('settings.disconnect_share')}}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </form>
        
        <h3>{{$t('settings.configure_default_databases')}}</h3>
        <h5>{{$t('settings.databases_desc')}}.</h5>
        <div v-show="configuration.ArcprocDBStatus || configuration.CompanyDBStatus">
          <div class="alert alert-success col-lg-7">
            <span class="fa fa-check"></span>
            {{$t('settings.default_databases_already_configured')}}.
          </div>
        </div>
        <form class="form-horizontal" v-on:submit.prevent="confDatabases()">
          <div class="row">
            <div class="col-lg-12">
              <div :class="['form-group margintop', errors.ArcprocDB.hasError ? 'has-error' : '']">
                <label class="col-sm-2 control-label">
                  {{$t('settings.arcproc_db_name')}}
                  <doc-info
                    :placement="'top'"
                    :title="$t('settings.arcproc_db')"
                    :chapter="'ArcprocDB'"
                    :inline="true"
                  ></doc-info>
                </label>
                <div class="col-sm-5">
                  <input
                    v-model="configuration.ArcprocDB"
                    type="text"
                    class="form-control"
                    required
                    disabled
                  >
                  <span v-if="errors.ArcprocDB.hasError" class="help-block">{{$t('settings.arcproc_db_already_exists')}}</span>
                </div>
              </div>
              <div :class="['form-group margintop', errors.CompanyDB.hasError ? 'has-error' : '']">
                <label class="col-sm-2 control-label">
                  {{$t('settings.company_db_name')}}
                  <doc-info
                    :placement="'top'"
                    :title="$t('settings.company_db')"
                    :chapter="'CompanyDB'"
                    :inline="true"
                  ></doc-info>
                </label>
                <div class="col-sm-5">
                  <input
                    v-model="configuration.CompanyDB"
                    type="text"
                    class="form-control"
                    required
                    disabled
                  >
                </div>
                <span v-if="errors.CompanyDB.hasError" class="help-block">{{$t('settings.company_db_already_exists')}}</span>
              </div>
              <div class="form-group">
                <label class="col-sm-2 control-label">
                </label>
                <div class="col-sm-5">
                  <button 
                    class="btn btn-primary margintop" 
                    type="submit"
                    :disabled="configuration.ArcprocDBStatus || configuration.CompanyDBStatus"
                  >
                    {{$t('settings.import_databases')}}
                  </button>
                </div>
              </div>
            </div>
          </div>
        </form>
      </div>
      <div v-else-if="configuration.installed && !configuration.isrunning" class="margintop">
        <div class="alert alert-warning">
          <span class="pficon pficon-warning-triangle-o"></span>
          <strong>{{$t('settings.services_are_not_running')}}:</strong>
          {{$t('settings.check_services')}}.
        </div>
      </div>
      
      <div v-else-if="!configuration.installed">
        <div class="blank-slate-pf" id>
          <div class="blank-slate-pf-icon">
            <span class="pficon pficon pficon-add-circle-o"></span>
          </div>
          <h1>{{$t('package_required')}}</h1>
          <p>{{$t('package_required_desc')}}.</p>
          <pre>nethserver-mssql mssql-server nethserver-samba</pre>
          <div class="margintop">
            <center>{{$t('settings.install_from_software_center')}}.</center>
          </div>
        </div>
      </div>
      
    </div>
  </div>
</template>

<script>
export default {
  name: "Settings",
  mounted() {
    this.getConfig()
  },
  data() {
    return {
      uiLoaded: false,
      errorMessage: null,
      configuration: {
        installed: false,
        isrunning: false,
        SharedFolderStatus: false,
        ArcprocDBStatus: false,
        CompanyDBStatus: false,
        SharedFolder: null,
        ArcprocDB: "Arcproc",
        CompanyDB: "Prova"
      },
      errors: this.initErrors()
    }
  },
  methods: {
    showErrorMessage(errorMessage, error) {
      console.error(errorMessage, error) /* eslint-disable-line no-console */
      this.errorMessage = errorMessage;
    },
    closeErrorMessage() {
      this.errorMessage = null;
    },
    getConfig() {
      var context = this;
      context.uiLoaded = false;
      nethserver.exec(
        ["nethserver-business/settings/read"],
        { action: "configuration" },
        null,
        function(success) {
          try {
            context.configuration = JSON.parse(success).configuration;
          } catch (e) {
            console.error(e);
          }
          context.uiLoaded = true;
        },
        function(error) {
          context.showErrorMessage(context.$i18n.t("settings.error_reading_business_configuration"), error);
        }
      );
    },
    confDatabases() {
      var context = this;
      var settingsObj = {
        action: "conf-db",
        "databases": {
          ArcprocDB: context.configuration.ArcprocDB,
          CompanyDB: context.configuration.CompanyDB
        }
      };
      context.loaders = true;
      context.errors = context.initErrors();
      nethserver.exec(
        ["nethserver-business/settings/validate"],
        settingsObj,
        null,
        function(success) {
          context.loaders = false;
          
          nethserver.notifications.success = context.$i18n.t(
            "settings.databases_configuration_ok"
          );
          
          nethserver.notifications.error = context.$i18n.t(
            "settings.databases_configuration_error"
          );
          
          // update values
          nethserver.exec(
            ["nethserver-business/settings/update"],
            settingsObj,
            function(stream) {
              console.info("nethserver-business-conf-db", stream);
            },
            function(success) {
              context.getConfig();
            },
            function(error, data) {
              console.error(error, data);
            },
            true //sudo
          );
        },
        function(error, data) {
          var errorData = {};
          context.loaders = false;
          context.errors = context.initErrors();
          try {
            errorData = JSON.parse(data);
            for (var e in errorData.attributes) {
              var attr = errorData.attributes[e];
              context.errors[attr.parameter].hasError = true;
              context.errors[attr.parameter].message = attr.error;
            }
          } catch (e) {
            console.error(e);
          }
        },
        true // sudo
      );
    },
    confSharedFolder() {
      var context = this;
      var settingsObj = {
        action: "conf-folder",
        "folder": {
          SharedFolder: context.configuration.SharedFolder.replace(".","")
        }
      };
      context.loaders = true;
      context.errors = context.initErrors();
      nethserver.exec(
        ["nethserver-business/settings/validate"],
        settingsObj,
        null,
        function(success) {
          context.loaders = false;
          
          nethserver.notifications.success = context.$i18n.t(
            "settings.folder_configuration_ok"
          );
          
          nethserver.notifications.error = context.$i18n.t(
            "settings.folder_configuration_error"
          );
          
          // update values
          nethserver.exec(
            ["nethserver-business/settings/update"],
            settingsObj,
            function(stream) {
              console.info("nethserver-business-update", stream);
            },
            function(success) {
              context.getConfig();
            },
            function(error, data) {
              console.error(error, data);
            },
            true //sudo
          );
        },
        function(error, data) {
          var errorData = {};
          context.loaders = false;
          context.errors = context.initErrors();
          try {
            errorData = JSON.parse(data);
            for (var e in errorData.attributes) {
              var attr = errorData.attributes[e];
              context.errors[attr.parameter].hasError = true;
              context.errors[attr.parameter].message = attr.error;
            }
          } catch (e) {
            console.error(e);
          }
        },
        true // sudo
      );
    },
    disconnectSharedFolder() {
      var context = this;
      context.configuration.SharedFolder = '';
      context.confSharedFolder();
    },
    initErrors() {
      return {
        SharedFolder: {
          hasError: false,
          message: ""
        },
        ArcprocDB: {
          hasError: false,
          message: ""
        },
        CompanyDB: {
          hasError: false,
          message: ""
        }
      }
    }
  }
};
</script>

<style scoped>
.margintop {
  margin-top: 15px;
}
.marginleft {
  margin-left: 10px;
}
</style>