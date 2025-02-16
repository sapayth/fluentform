<template>
    <div class="ff_form_integrations">
        <el-row class="setting_header">
            <el-col :md="12"><h2>{{ $t('All Form Integrations') }}</h2></el-col>
            <!--Add Feed-->
            <el-col v-if="!isEmpty(available_integrations)" :md="12" class="action-buttons mb15 clearfix">
                <el-dropdown type="primary" class="pull-right" @command="add" :hide-on-click="false" >
                    <el-button size="small" type="primary">
                        {{ $t('Add New Integration') }}<i class="el-icon-arrow-down el-icon--right"></i>
                    </el-button>
                    <el-dropdown-menu slot="dropdown" style="max-height: 400px;overflow: auto">
                        <el-dropdown-item>
                            <el-input @click.prevent autofocus v-model="search" :placeholder="$t('Search Integration')"></el-input>
                        </el-dropdown-item>
                        <el-dropdown-item v-for="(integration,integration_name) in filteredList" :key="integration_name" :command="integration_name">{{integration.title}}</el-dropdown-item>
                    </el-dropdown-menu>
                </el-dropdown>
            </el-col>
        </el-row>

        <div v-if="has_pro && isEmpty(available_integrations) && !loading">
            <p style="font-size: 18px; text-align: center;">
                {{ $t(this.integrationsResource.instruction) }}
            </p>
            <p style="text-align: center;">
                <a 
                    class="el-button el-button--primary el-button--small el-dropdown-selfdefine" 
                    :href="all_module_config_url"
                >
                    {{ $t('Configure Modules') }}
                </a>
            </p>
        </div>

        <!-- GetResponse Feeds Table: 1 -->
        <el-table v-if="!isEmpty(available_integrations)" stripe v-loading="loading" :data="integrations" class="el-fluid">
            <template slot="empty">
                <div class="getting_started_message">
                    <p>{{ $t('You don\'t have any form feed integration yet. Create new feed and connect your data to your favorite CRM / Marketing tool') }}</p>
                </div>
            </template>
            <el-table-column label="Status" width="90">
                <template slot-scope="scope">
                    <el-switch active-color="#13ce66" @change="handleActive(scope.row)" v-model="scope.row.enabled"></el-switch>
                </template>
            </el-table-column>

            <el-table-column
                width="180"
                :label="$t('Integration')">
                <template slot-scope="scope">
                    <img v-if="scope.row.provider_logo" class="general_integration_logo" :src="scope.row.provider_logo" :alt="scope.row.provider" />
                    <span class="general_integration_name" v-else>{{scope.row.provider}}</span>
                </template>
            </el-table-column>

            <el-table-column
                :label="$t('Title')">
                <template slot-scope="scope">
                    {{scope.row.name}}
                </template>
            </el-table-column>

            <el-table-column width="160" :label="$t('Actions')" class-name="action-buttons">
                <template slot-scope="scope">
                    <el-button
                        @click="edit(scope.row)"
                        type="primary"
                        icon="el-icon-setting"
                        size="mini"></el-button>
                    <remove @on-confirm="remove(scope.row.id, scope)"></remove>
                </template>
            </el-table-column>
        </el-table>

        <br />
        <p v-show="has_pro && !integrations.length" style="text-align: right;">
            <a :href="all_module_config_url">Check Global Integration Settings</a>
            <a style="margin-left: 20px" target="_blank" rel="noopener" href="https://wpmanageninja.com/docs/fluent-form/integrations-available-in-wp-fluent-form/">{{ $t('View Documentations') }}</a>
        </p>

        <div v-if="!has_pro" class="upgrade_to_pro">
            <p>
                {{ $t(this.integrationsResource.instruction) }}
            </p>

            <div class="action-btns">
                <a 
                    class="el-button el-button--primary el-button--small el-dropdown-selfdefine" 
                    :href="upgrade_url"
                >
                    {{ $t('Upgrade to PRO') }}
                </a>

                <a 
                    class="" 
                    :href="integrationsResource.list_url"
                >
                    {{ $t('See All Integrations') }}
                </a>
            </div>
            
            <img :src="integrationsResource.asset_url" alt="integrations asset" />
        </div>
    </div>
</template>

<script>
    import remove from '../../confirmRemove.vue';
    import isEmpty from 'lodash/isEmpty';
    export default {
        name: 'generalSettings',
        props: ['form_id', 'inputs', 'has_pro', 'editorShortcodes'],
        components: {
            remove
        },
        data() {
            return {
                search: '',
                loading: true,
                integrations: [],
                errors: new Errors,
                available_integrations: {},
                all_module_config_url: '',
                instruction: "Fluent Forms Pro has tons of integrations to take your forms to the next level. From payment gateways to quiz building, SMS notifications to email marketing - you'll get integrations for various purposes. Even if you don't find your favorite tools, you can integrate them easily with Zapier.",
                integrationsAsset: window.FluentFormApp.integrations_asset_url,
                upgrade_url: window.FluentFormApp.upgrade_url,
                integrations_url: window.FluentFormApp.integrations_url,
                integrationsResource: window.FluentFormApp.integrationsResource,
            }
        },
        methods: {
            add(integration_name) {
                let integration = this.available_integrations[integration_name];

                if(!integration.is_active) {
                    // Handle Inactive state
                    this.$confirm(integration.configure_message, integration.configure_title, {
                        confirmButtonText: integration.configure_button_text,
                        cancelButtonText: 'Cancel',
                        type: 'warning'
                    }).then(() => {
                        window.location.href = integration.global_configure_url;
                        return;
                    }).catch(() => {

                    });
                    return;
                }

                this.$router.push({
                    name: 'edit_integration',
                    params: {
                        integration_id: 0,
                        integration_name: integration_name
                    }
                });
                return;

                console.log(integration);
                this.selectedIndex = this.integrations.length;
                this.selected_id = 0;
                this.editing_item = false;
                this.show_edit = true;
            },
            edit(integration) {
                this.$router.push({
                    name: 'edit_integration',
                    params: {
                        integration_id: integration.id,
                        integration_name: integration.provider
                    }
                });
            },
            handleActive(row) {
                let data = {
                    form_id: this.form_id,
                    status: row.enabled,
                    notification_id: row.id,
                    action: 'fluentform_post_update_form_integration_status'
                };
                FluentFormsGlobal.$post(data)
                    .then(response => {
                        console.log(response);
                        this.$notify.success({
                            offset: 30,
                            title: 'Success!',
                            message: response.data.message
                        });
                    })
                    .fail(error => {
                        this.$notify.error({
                            offset: 30,
                            title: 'Success!',
                            message: error.responseJSON.data.message
                        });
                    });
            },
            remove(id, scope) {
               let $index  = scope.$index;
                let data = {
                    action: 'fluentform-delete-general_integration_feed',
                    integration_id: id,
                    form_id: this.form_id
                };

                FluentFormsGlobal.$post(data)
                    .then(response => {
                        this.$notify.success({
                            offset: 30,
                            title: 'Success!',
                            message: response.data.message
                        });
                        this.integrations.splice($index, 1);
                    })
                    .fail(e => console.log(e));
            },
            getFeeds() {
                this.loading = true;
                FluentFormsGlobal.$get({
                    action: 'fluentform_get_all-general-integration-feeds',
                    form_id: this.form_id
                })
                    .then(response => {
                        this.integrations = response.data.feeds;
                        this.available_integrations = response.data.available_integrations;
                        this.all_module_config_url = response.data.all_module_config_url;
                    })
                    .fail(error => {
                        console.log(error);
                    })
                    .always(r => this.loading = false);
            },
            isEmpty
        },
        computed: {
            filteredList() {
                let filteredList = {};
                Object.keys(this.available_integrations).map(key => {
                    if (key.toLowerCase().includes(this.search.toLowerCase())) {
                        filteredList[key] = this.available_integrations[key];
                    }
                });
                return filteredList;
            }

        },
        beforeMount() {
            this.getFeeds();
        },
        beforeCreate() {
            jQuery('head title').text('Form Integrations - Fluent Forms');
        }
    }
</script>


