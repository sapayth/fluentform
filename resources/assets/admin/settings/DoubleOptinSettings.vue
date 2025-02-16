<template>
    <div v-loading="loading">
        <!--Save settings-->
        <el-row class="setting_header">
            <el-col :md="18">
                <h2>
                    {{ $t('Global Double Optin Settings') }}
                    <el-tooltip class="item" placement="bottom-start" effect="light">
                        <div slot="content">
                            <h3>{{ $t('Double Optin Settings') }}</h3>
                            <p>{{
                                    $t('Configure email confirmation / double optin for form form submissions.')
                                }}
                                <br/>
                                {{
                                    $t('This is a global settings. After configure this, You can enable in form settings for a specific form.')
                                }}
                            </p>
                        </div>
                        <i class="el-icon-info el-text-info"></i>
                    </el-tooltip>
                </h2>
            </el-col>
            <el-col :md="6" class="action-buttons clearfix mb15">
                <el-button v-loading="saving" size="medium" class="pull-right" type="success" icon="el-icon-success" @click="save"
                >{{ $t('Save Settings') }}
                </el-button>
            </el-col>
        </el-row>

        <el-form label-width="205px"
                 label-position="left" v-if="settings" :data="settings">
            <el-checkbox true-label="yes" false-label="no" v-model="settings.enabled">
                {{ $t('Enable Double Optin Module') }}
            </el-checkbox>
            <template v-if="settings.enabled == 'yes'">
                <el-form-item class="ff_top_50">
                    <template slot="label">
                        {{ $t('Global Email Subject') }}
                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <p>
                                    {{ $t('Email Subject for double optin email.') }}
                                </p>
                            </div>

                            <i class="el-icon-info el-text-info"/>
                        </el-tooltip>
                    </template>
                    <el-input size="small" :placeholder="$t('Email Subject')"
                              v-model="settings.email_subject"/>
                </el-form-item>
                <el-form-item>
                    <template slot="label">
                        {{ $t('Global Optin Email Body') }}
                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <h3>{{ $t('Optin Email Body') }}</h3>
                                <p>
                                    {{ $t('Enter the content you would like the user to ') }}<br>
                                    {{ $t('send via email for confirmation.') }}
                                </p>
                            </div>

                            <i class="el-icon-info el-text-info"/>
                        </el-tooltip>
                    </template>
                    <el-input v-if="settings.asPlainText == 'yes'" v-model="settings.email_body" type="textarea" :rows="12"></el-input>
                    <wp-editor v-else :height="250"
                               v-model="settings.email_body"/>

                    <el-checkbox style="margin-bottom: 10px;" true-label="yes" false-label="no" v-model="settings.asPlainText">
                        {{ $t('Send Email as RAW HTML Format') }}
                    </el-checkbox>

                    <p>{{ $t('Use #confirmation_url# smartcode for double optin confirmation URL') }}</p>

                </el-form-item>

                <!--from name-->
                <el-form-item>
                    <template slot="label">
                        {{ $t('From Name') }}

                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <h3>{{ $t('From Name') }}</h3>

                                <p>
                                    {{ $t('Enter the name you would like the notification email sent from') }}
                                </p>
                            </div>

                            <i class="el-icon-info el-text-info"></i>
                        </el-tooltip>
                    </template>

                    <el-input size="small" placeholder="From Name"
                              v-model="settings.fromName"/>
                    <p v-if="settings.fromName">
                        {{ $t('It will only be visible in the email if "From Email" value is available') }}
                    </p>
                </el-form-item>

                <!--from email-->
                <el-form-item>
                    <template slot="label">
                        {{ $t('From Email') }}

                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <h3>{{ $t('From Email Address') }}</h3>
                                <p>
                                    {{ $t('Enter the email address you would like the ') }}<br>
                                    {{ $t('notification email sent from.') }}
                                </p>
                            </div>
                            <i class="el-icon-info el-text-info"></i>
                        </el-tooltip>
                    </template>

                    <el-input size="small" :placeholder="$t('From Email')"
                              v-model="settings.fromEmail"/>
                    <p v-if="settings.fromEmail">{{
                            $t('It\'s not recommended to change from email. Please use your domain\'s email / SMTP main email. Otherwise email may failed to send.')
                        }}</p>
                </el-form-item>

                <!--reply to-->
                <el-form-item>
                    <template slot="label">
                        {{ $t('Reply To') }}

                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <h3>{{ $t('Reply To') }}</h3>

                                <p>
                                    {{ $t('Enter the email address you would like to be ')}}<br>
                                    {{ $t('used as the reply to address for the notification email.') }}
                                </p>
                            </div>

                            <i class="el-icon-info el-text-info"></i>
                        </el-tooltip>
                    </template>

                    <el-input size="small" :placeholder="$t('Reply To Email')"
                              v-model="settings.replyTo"/>
                </el-form-item>

                <el-checkbox true-label="yes" false-label="no" v-model="settings.auto_delete_status">
                    {{ $t('Automatically delete unconfirmed entries if not confirmed in certain days') }}
                </el-checkbox>
                <el-form-item v-if="settings.auto_delete_status == 'yes'" class="ff_top_25">
                    <template slot="label">
                        {{ $t('Waiting Days') }}
                        <el-tooltip class="item" placement="bottom-start" effect="light">
                            <div slot="content">
                                <p>
                                    {{ $t('How many days, it will wait before deleting the unconfirmed entries') }}
                                </p>
                            </div>
                            <i class="el-icon-info el-text-info"/>
                        </el-tooltip>
                    </template>
                    <el-input-number v-model="settings.auto_delete_day_span" :min="1"></el-input-number>
                </el-form-item>
            </template>
        </el-form>
        <div v-else-if="!hasPro">
            <h2>{{ $t('This is a pro feature.Please upgrade to pro to enable this feature') }}</h2>
        </div>
        <div v-else-if="need_update">
            <h2>{{ $t('Please update Fluent Forms Pro Addon to latest version') }}</h2>
        </div>
    </div>
</template>

<script type="text/babel">
    import wpEditor from '../../common/_wp_editor';

    export default {
        name: 'DoubleOptinSettings',
        components: {
            wpEditor
        },
        data() {
            return {
                settings: false,
                hasPro: !!window.FluentFormApp.has_pro,
                loading: true,
                saving: false,
                need_update: false
            }
        },
        methods: {
            save() {
                this.saving = true;
                FluentFormsGlobal.$post({
                    action: 'fluentform_save_global_double_optin',
                    settings: this.settings
                })
                    .then((response) => {
                        this.$notify.success(response.data.message);
                    })
                    .fail((errors) => {
                        console.log(errors);
                    })
                    .always(() => {
                        this.saving = false;
                    });
            },
            fetch() {
                this.loading = true;
                FluentFormsGlobal.$get({
                    action: 'fluentform_get_global_double_optin'
                })
                    .then(response => {
                        this.settings = response.data.settings;
                    })
                    .fail((errors) => {
                        if (errors.status == 400) {
                            this.need_update = true;
                        }
                    })
                    .always(() => {
                        this.loading = false;
                    });
            }
        },
        mounted() {
            if (this.hasPro) {
                this.fetch();
            } else {
                this.loading = false;
            }
        }
    }
</script>
